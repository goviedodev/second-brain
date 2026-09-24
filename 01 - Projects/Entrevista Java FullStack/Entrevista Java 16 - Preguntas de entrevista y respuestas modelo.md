---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 16. Preguntas de entrevista y respuestas modelo

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 16.1 Estructura STAR para preguntas de experiencia 🎯
**S**ituación → **T**area → **A**cción → **R**esultado (con números).

> *"En un servicio de pedidos teníamos un endpoint con p95 de 3 segundos **(S)**. Me pidieron bajarlo sin cambiar el contrato **(T)**. Perfilé con Actuator, encontré un N+1 en el listado, lo resolví con `@EntityGraph` y agregué caché en el catálogo **(A)**. El p95 bajó a 400 ms y los timeouts del front desaparecieron **(R)**."*

Ten preparadas 4 historias: un problema técnico difícil, un conflicto con otra persona, un error que cometiste, y un logro del que estés orgulloso.

## 16.2 Banco de preguntas técnicas

**FullStack / generales**
- Recorre una petición desde el clic en Angular hasta la base de datos.
- ¿Cómo depuras un bug que solo ocurre en producción?
- ¿Qué haces si un endpoint se pone lento?

**Java**
- `==` vs `equals`; contrato `equals`/`hashCode`.
- Checked vs unchecked exceptions.
- `HashMap` por dentro; qué pasa si el `hashCode` es malo.
- Streams: intermedias vs terminales; `map` vs `flatMap`.
- Inmutabilidad y `record`.
- Garbage collection: generaciones, cuándo te preocupa.

**Spring**
- Cómo funciona la inyección por constructor y por qué es la preferida.
- `@Component` vs `@Service` vs `@Repository` (semántica + traducción de excepciones).
- Ciclo de vida de un bean; `@PostConstruct`.
- Por qué falla `@Transactional` en una llamada interna.
- Cómo manejas errores de forma global.

**Seguridad**
- Explica JWT y por qué no guardas datos sensibles en el payload.
- Diferencia OAuth2 / OIDC.
- Cómo revocas un JWT.
- CORS: qué es y cómo lo configuras bien.

**Bases de datos**
- Índices: cuándo ayudan y cuándo estorban.
- Transacciones y niveles de aislamiento; lectura sucia / no repetible / fantasma.
- Cuándo NoSQL en vez de SQL.

**Cloud / DevOps**
- Diferencia entre Cloud Run, Cloud Functions y GKE.
- Cómo despliegas sin downtime.
- Qué haces si un pod está en `CrashLoopBackOff`.

## 16.3 Preguntas que TÚ debes hacer (evalúan mucho esto) 🎯
- ¿Cómo está compuesto el equipo y cómo es el proceso de trabajo (sprints, refinement)?
- ¿Cómo es el pipeline de despliegue? ¿Con qué frecuencia van a producción?
- ¿Qué porcentaje del trabajo es feature nueva vs mantención de legado?
- ¿Cómo miden el éxito de esta posición en los primeros 3 y 6 meses?
- ¿Hay turnos de soporte / on-call?
- ¿Cuál es el mayor desafío técnico del equipo hoy?
- ¿Cuáles son los pasos siguientes del proceso y en qué plazo?

## 16.4 Banderas rojas a evitar en tus respuestas
Hablar mal de empleadores anteriores, responder "no sé" y detenerse (mejor: *"no lo he usado en producción, pero el concepto es X y lo abordaría así"*), inventar experiencia (se detecta al profundizar), respuestas de una palabra, o monólogos de 10 minutos.
