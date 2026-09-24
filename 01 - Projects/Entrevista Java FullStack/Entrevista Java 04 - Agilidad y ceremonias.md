---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 4. Agilidad y ceremonias

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 4.1 Scrum en una pantalla 🎯

**Roles**
- **Product Owner:** dueño del *qué* y del valor; prioriza el Product Backlog.
- **Scrum Master:** facilita, remueve impedimentos, cuida el proceso. No es jefe.
- **Development Team:** autoorganizado, multifuncional, dueño del *cómo*.

**Artefactos**
- **Product Backlog** — lista priorizada de todo.
- **Sprint Backlog** — lo comprometido para este sprint.
- **Incremento** — software potencialmente entregable + *Definition of Done*.

**Ceremonias**

| Ceremonia | Duración (sprint 2 sem.) | Propósito |
|---|---|---|
| **Sprint Planning** | ≤ 4 h | Definir objetivo del sprint y seleccionar historias |
| **Daily Standup** | 15 min | Sincronizar, detectar bloqueos |
| **Sprint Review** | ≤ 2 h | Mostrar el incremento a stakeholders, recibir feedback |
| **Retrospectiva** | ≤ 1,5 h | Mejorar el proceso: qué mantener, qué cambiar |
| **Refinement** (continuo) | ~10% del sprint | Detallar y estimar historias futuras |

## 4.2 La Daily 🎯
Tres preguntas: **¿qué hice ayer?**, **¿qué haré hoy?**, **¿qué me bloquea?**

Buenas prácticas que puedes mencionar y te hacen ver senior:
- Es una **sincronización del equipo**, no un reporte al jefe.
- Los problemas se *detectan* en la daily y se *resuelven después* ("parking lot").
- 15 minutos, de pie o cámara encendida, misma hora siempre.
- Se habla del **tablero**, no de las personas: se recorren las historias de derecha a izquierda (lo más cerca de "Done" primero).

## 4.3 Estimación
- **Story points** con Fibonacci (1,2,3,5,8,13): miden *complejidad + incertidumbre + esfuerzo*, no horas.
- **Planning Poker** para estimar en equipo y aflorar supuestos distintos.
- **Velocidad** = puntos completados por sprint; sirve para *pronosticar*, no para comparar equipos ni presionar.

## 4.4 Kanban (por si trabajan con flujo continuo)
- Visualizar el flujo, **limitar WIP**, gestionar el flujo, políticas explícitas, mejora continua.
- Métricas: **lead time**, **cycle time**, **throughput**, diagrama de flujo acumulado.
- Se usa mucho en equipos de soporte/mantención donde no se puede comprometer un sprint fijo.

## 4.5 Definiciones que suelen preguntar
- **DoR (Definition of Ready):** criterios para que una historia pueda entrar al sprint (criterios de aceptación claros, dependencias resueltas, estimada).
- **DoD (Definition of Done):** criterios para decir que está terminada (código revisado, tests, desplegado en DEV, documentado).
- **Historia de usuario:** *Como \<rol\>, quiero \<acción\>, para \<beneficio\>* + criterios de aceptación (Gherkin: Dado/Cuando/Entonces).

## 4.6 Preguntas típicas
- *"¿Qué haces si una historia no alcanza a terminarse en el sprint?"* → Vuelve al backlog, se re-estima el remanente; no se "extiende" el sprint.
- *"¿Qué haces si el PO agrega trabajo a mitad de sprint?"* → Se conversa el impacto en el objetivo del sprint; si entra algo, sale algo.
- *"¿Cómo aporta un dev a la agilidad?"* → Historias pequeñas, PRs chicos, integración diaria, avisar bloqueos temprano, participar del refinement.
