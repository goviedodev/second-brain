---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 19. Cheat sheets

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 19.1 Respuestas de 30 segundos

| Pregunta | Respuesta comprimida |
|---|---|
| ¿Qué es CI/CD? | Integrar y validar cada commit automáticamente; tener siempre un artefacto desplegable; y opcionalmente desplegarlo solo si pasa las puertas de calidad. |
| ¿Qué es un JWT? | Un token firmado con tres partes (header, payload, firma) que transporta claims verificables sin consultar al servidor. Firmado, no cifrado. |
| ¿OAuth2 vs OIDC? | OAuth2 autoriza acceso a recursos; OIDC agrega autenticación e `id_token` sobre OAuth2. |
| ¿Qué es Pub/Sub? | Mensajería gestionada de GCP: publishers a un topic, subscriptions que reciben copias; at-least-once, por lo que el consumidor debe ser idempotente. |
| ¿Virtual Threads? | Hilos livianos de la JVM (Java 21) que permiten código bloqueante con escalabilidad reactiva; uno por tarea, sin pool, ideales para I/O. |
| ¿Docker vs VM? | El contenedor comparte kernel y aísla procesos; la VM emula hardware con su propio SO. Segundos y MBs vs minutos y GBs. |
| ¿Para qué Kubernetes? | Orquestar contenedores: escalado, autorreparación, descubrimiento y despliegues sin downtime de forma declarativa. |
| ¿Liveness vs readiness? | Liveness reinicia el pod; readiness lo saca del balanceador sin reiniciarlo. |
| ¿SQL o NoSQL? | SQL para datos relacionales, integridad y consultas variadas; NoSQL para escala, esquema flexible y patrones de consulta conocidos de antemano. |
| ¿Qué patrones usas? | Strategy para algoritmos intercambiables, Builder para objetos complejos y Circuit Breaker para resiliencia — con ejemplos reales. |
| ¿`switchMap` vs `mergeMap`? | `switchMap` cancela la petición anterior (búsquedas); `mergeMap` las ejecuta todas en paralelo. |
| ¿Qué es la daily? | Sincronización de 15 minutos del equipo para alinear el trabajo del día y levantar bloqueos, no un reporte de estado al jefe. |

## 19.2 Comandos rápidos
```bash
# Docker
docker build -t app:1.0 . && docker run -p 8080:8080 app:1.0
docker logs -f <id>   |   docker exec -it <id> sh

# Kubernetes
kubectl get pods -n prod
kubectl describe pod <pod>   |   kubectl logs -f <pod> --previous
kubectl rollout undo deployment/api

# GCP
gcloud pubsub topics publish mi-topic --message='{"id":1}'
gcloud run deploy api --image gcr.io/proj/api:sha --region us-central1
gcloud functions deploy procesar --trigger-topic mi-topic

# Maven
mvn clean verify   |   mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

## 19.3 Checklist antes de la entrevista
- [ ] Puedo dibujar el flujo OAuth2/JWT en una hoja.
- [ ] Tengo 3 patrones de diseño con ejemplo real propio.
- [ ] Tengo 4 historias STAR listas.
- [ ] Puedo explicar la diferencia entre CI, Delivery y Deployment.
- [ ] Puedo explicar Virtual Threads y su trampa (pinning, no hacer pool).
- [ ] Sé decir mi rango salarial sin titubear, aclarando líquido.
- [ ] Tengo 5 preguntas preparadas para ellos.
- [ ] Probé cámara, micrófono y conexión.
