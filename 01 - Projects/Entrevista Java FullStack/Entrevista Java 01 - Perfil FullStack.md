---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 1. Perfil FullStack

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## Qué es
Un perfil FullStack domina el ciclo completo: **frontend** (Angular/React), **backend** (Java/Spring Boot), **persistencia** (SQL/NoSQL), **infraestructura** (Docker, Kubernetes, cloud) y **entrega** (CI/CD). No significa ser experto en todo: significa poder llevar una funcionalidad de punta a punta sin bloquearse.

## Cómo lo explico en entrevista 🎯
> "Trabajo el flujo completo: modelo el contrato REST, implemento el backend en Spring Boot con su capa de servicio y repositorio, expongo el endpoint asegurado con JWT, consumo desde Angular con un servicio y un interceptor, y me hago cargo del pipeline que lo despliega en contenedores. Mi centro de gravedad es el backend Java, pero el frontend no me frena."

**Regla de oro:** declara tu *centro de gravedad*. Decir "soy experto en todo" resta credibilidad; decir "backend fuerte + frontend sólido" suma.

## Mapa mental del stack

| Capa | Tecnología | Qué debo saber responder |
|---|---|---|
| UI | Angular, TypeScript, RxJS | Componentes, estado, formularios, interceptores |
| API | Spring Boot, REST | Controllers, DTOs, validación, manejo de errores |
| Seguridad | Spring Security, JWT, OAuth2 | Filtros, flujos de token, roles |
| Datos | PostgreSQL / Firestore | JPA, transacciones, modelado NoSQL |
| Mensajería | Pub/Sub, Kafka | Eventos, idempotencia, orden |
| Infra | Docker, Kubernetes, GCP | Imagen, deployment, escalamiento |
| Entrega | CI/CD, Git | Pipeline, ramas, estrategia de release |

## Preguntas típicas
- *"¿Qué parte del stack prefieres?"* → Responde con el centro de gravedad + evidencia.
- *"Cuéntame una funcionalidad que hiciste end-to-end."* → Usa formato STAR (ver §16).
- *"¿Cómo decides si una lógica va en el front o en el back?"* → Regla: **validación de UX en el front, validación de verdad en el back**. El front nunca es fuente de confianza.
