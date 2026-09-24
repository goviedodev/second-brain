---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 18. Plan de repaso de 7 días

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

| Día | Foco | Entregable de práctica |
|---|---|---|
| 1 | **JWT + OAuth2 + Spring Security** (prioridad máxima) | Dibujar el flujo Authorization Code + PKCE de memoria; escribir un `SecurityFilterChain` sin mirar |
| 2 | Spring Boot + JPA + patrones de diseño | Implementar Strategy con inyección de `List<T>`; explicar N+1 y su fix |
| 3 | Programación funcional + concurrencia + Virtual Threads | Resolver 5 ejercicios de Streams; explicar pinning y por qué no se hace pool |
| 4 | GCP: Pub/Sub, Cloud Functions, Firestore + Kafka | Diseñar un flujo evento→función→Firestore; comparar Kafka vs Pub/Sub en voz alta |
| 5 | Docker + Kubernetes + CI/CD | Escribir un Dockerfile multi-stage y un Deployment de memoria; explicar liveness vs readiness |
| 6 | Angular + agilidad | Explicar `switchMap` vs `mergeMap`, OnPush, interceptor JWT; repasar ceremonias |
| 7 | Simulacro completo + negociación | Responder 20 preguntas en voz alta cronometrado; ensayar los guiones de sueldo |

**Método:** cada tema no está listo hasta que puedas **explicarlo en voz alta en 2 minutos sin leer**. Grábate: si dudas, vuelve a esa sección.
