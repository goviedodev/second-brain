---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 3. CI/CD e Integración Continua

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 3.1 Definiciones que hay que separar bien 🎯

| Término | Qué significa |
|---|---|
| **CI — Integración Continua** | Cada push se integra a la rama principal y dispara build + tests automáticos. Objetivo: detectar conflictos y regresiones en minutos, no en semanas. |
| **CD — Entrega Continua (Delivery)** | Todo commit que pasa el pipeline queda **listo para desplegar**; el despliegue a producción es un botón manual. |
| **CD — Despliegue Continuo (Deployment)** | Ese artefacto se despliega **automáticamente** a producción si pasa todas las puertas de calidad. |

> Respuesta corta de entrevista: *"CI es integrar y validar temprano; Delivery es tener siempre un artefacto desplegable; Deployment es que ese artefacto llegue solo a producción."*

## 3.2 Anatomía de un pipeline

```
commit → build → tests unitarios → análisis estático (SonarQube)
       → tests de integración → build de imagen Docker
       → push al registry (Artifact Registry / ECR)
       → deploy a DEV → tests E2E → deploy a QA (aprobación) → deploy a PROD
```

**Puertas de calidad (quality gates)** habituales: cobertura mínima (80%), 0 vulnerabilidades críticas, 0 code smells bloqueantes, build reproducible.

## 3.3 Ejemplo: GitHub Actions para Spring Boot

```yaml
name: ci
on:
  push: { branches: [main] }
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '21', distribution: 'temurin', cache: maven }
      - name: Tests
        run: mvn -B verify
      - name: Análisis estático
        run: mvn sonar:sonar -Dsonar.projectKey=mi-api
      - name: Build imagen
        run: docker build -t europe-docker.pkg.dev/$PROJECT/repo/api:${{ github.sha }} .
      - name: Push
        run: docker push europe-docker.pkg.dev/$PROJECT/repo/api:${{ github.sha }}
```

## 3.4 Ejemplo: Cloud Build (GCP)

```yaml
steps:
  - name: maven:3.9-eclipse-temurin-21
    entrypoint: mvn
    args: ['verify']
  - name: gcr.io/cloud-builders/docker
    args: ['build','-t','gcr.io/$PROJECT_ID/api:$SHORT_SHA','.']
  - name: gcr.io/cloud-builders/gcloud
    args: ['run','deploy','api','--image','gcr.io/$PROJECT_ID/api:$SHORT_SHA','--region','us-central1']
images: ['gcr.io/$PROJECT_ID/api:$SHORT_SHA']
```

## 3.5 Estrategias de ramas 🎯

| Estrategia | Descripción | Cuándo |
|---|---|---|
| **GitFlow** | `develop`, `release/*`, `hotfix/*`, `main` | Releases con fecha, varios entornos, software versionado |
| **Trunk Based** | Ramas cortas (<1 día) contra `main`, feature flags | CI/CD real, despliegues diarios |
| **GitHub Flow** | Rama por feature → PR → `main` → deploy | Equipos web, entrega continua |

Trunk Based es lo que suele buscarse cuando la empresa dice "queremos CI de verdad": ramas largas = integración tardía = conflictos.

## 3.6 Estrategias de despliegue 🎯

- **Rolling update:** reemplaza pods de a poco (default en Kubernetes).
- **Blue/Green:** dos entornos idénticos, se cambia el tráfico de golpe. Rollback instantáneo, cuesta el doble de infra.
- **Canary:** 5% del tráfico a la versión nueva, se observa, se sube gradualmente.
- **Feature flags:** el código va a producción apagado; se enciende por configuración. Desacopla *deploy* de *release*.

## 3.7 Conceptos complementarios
- **Artefacto inmutable:** se construye una vez y se promueve el *mismo* binario/imagen entre entornos. Nunca recompilar por ambiente.
- **IaC (Terraform):** la infraestructura también se versiona.
- **Secretos:** nunca en el repo. Secret Manager / GitHub Secrets / Kubernetes Secrets.
- **DORA metrics:** frecuencia de despliegue, lead time, MTTR, % de fallos de cambio. Si preguntan "¿cómo mides que el CI/CD funciona?", esta es la respuesta.
