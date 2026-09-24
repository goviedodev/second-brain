---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 12. Docker

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 12.1 Conceptos 🎯
- **Imagen:** plantilla inmutable en capas. **Contenedor:** instancia en ejecución de una imagen.
- **Contenedor vs VM:** el contenedor comparte el kernel del host y aísla procesos (namespaces + cgroups); la VM emula hardware completo con su propio SO. Contenedor: arranque en segundos, MBs. VM: minutos, GBs.
- **Capas y caché:** cada instrucción del Dockerfile crea una capa; si una cambia, se invalida el caché de todas las siguientes → **ordena de lo que menos cambia a lo que más cambia**.
- **Volumen:** persistencia fuera del ciclo de vida del contenedor.

## 12.2 Dockerfile multi-stage para Spring Boot 🎯

```dockerfile
# ---- build ----
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn -B dependency:go-offline          # capa cacheada: solo cambia si cambia el pom
COPY src ./src
RUN mvn -B clean package -DskipTests

# ---- runtime ----
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN addgroup -S app && adduser -S app -G app   # no ejecutar como root
COPY --from=build /app/target/*.jar app.jar
USER app
EXPOSE 8080
HEALTHCHECK --interval=30s CMD wget -qO- http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java","-XX:MaxRAMPercentage=75","-jar","app.jar"]
```

**Por qué multi-stage:** la imagen final no lleva Maven ni el código fuente → de ~700 MB a ~200 MB, menos superficie de ataque.

## 12.3 Buenas prácticas
- Imagen base ligera (`-alpine`, `-jre` en vez de `-jdk`, o *distroless*).
- `.dockerignore` (excluir `target/`, `node_modules/`, `.git`).
- **Nunca** secretos en el Dockerfile ni en `ENV`: van por variables de entorno o Secret Manager en tiempo de ejecución.
- Tag explícito por commit (`api:1.4.2` o `api:$SHA`), no `latest` en producción.
- Un proceso por contenedor.
- Escanear imágenes (Trivy, Artifact Registry scanning).

## 12.4 Comandos que debes saber decir de memoria
```bash
docker build -t api:1.0 .
docker run -d -p 8080:8080 -e SPRING_PROFILES_ACTIVE=prod --name api api:1.0
docker ps / docker logs -f api / docker exec -it api sh
docker stop api && docker rm api
docker image prune -a
```

## 12.5 docker-compose para desarrollo local
```yaml
services:
  api:
    build: .
    ports: ["8080:8080"]
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/app
    depends_on:
      db: { condition: service_healthy }
  db:
    image: postgres:16-alpine
    environment: { POSTGRES_PASSWORD: dev, POSTGRES_DB: app }
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
```
