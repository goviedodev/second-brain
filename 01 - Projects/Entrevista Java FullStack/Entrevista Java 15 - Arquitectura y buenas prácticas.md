---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 15. Arquitectura y buenas prácticas

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 15.1 REST bien hecho 🎯
- Sustantivos en plural: `/api/v1/pedidos/{id}/items`. Sin verbos en la URL.
- Verbos HTTP: `GET` (leer, seguro), `POST` (crear), `PUT` (reemplazar, idempotente), `PATCH` (parcial), `DELETE` (idempotente).
- Códigos: `200`, `201` + `Location`, `204`, `400`, `401`, `403`, `404`, `409` (conflicto), `422`, `429` (rate limit), `500`, `503`.
- Paginación (`?page=&size=` o cursor), filtros, ordenamiento.
- Versionado en la URL (`/v1/`) o por header; versiona antes de romper contratos.
- Errores consistentes (RFC 7807 *Problem Details*).
- **Idempotencia** en `POST` sensibles vía `Idempotency-Key`.

## 15.2 Observabilidad
Los tres pilares: **logs** (estructurados en JSON, con `traceId`), **métricas** (Micrometer/Prometheus: latencia p95/p99, tasa de error, throughput), **trazas** (OpenTelemetry a través de servicios). Alertas sobre SLO, no sobre CPU.

## 15.3 Rendimiento
Cachés (`@Cacheable`, Redis), paginación obligatoria, índices en BD, evitar N+1, compresión, CDN para estáticos, conexiones agrupadas (HikariCP), timeouts en **toda** llamada externa.

## 15.4 Calidad
Cobertura ≥ 80% con tests que valen (no tests triviales), PRs pequeños, revisión de código, análisis estático (SonarQube), escaneo de dependencias, logs sin datos personales, *feature flags* para desacoplar deploy de release.
