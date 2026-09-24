---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java
---

# Preparación entrevista Java FullStack

Guía de repaso técnico para una entrevista FullStack (Java/Spring + Angular + GCP).

- **Material original:** `/home/goviedo/proyectos/entrevistas/preparacion-java`
  - `guia-entrevista.md` / `.html` — la guía completa (fuente de verdad; el HTML se genera con `node build-html.js`).
  - `integracion.txt` — el temario original de la entrevista.
  - `gem-gemini.md` y `comandos-gem.md` — la configuración y los comandos del Gem de Gemini «Sparring Técnico FullStack».
- **Cómo estudiar:** cada tema está listo cuando puedes **explicarlo en voz alta en 2 minutos sin leer**. Márcalo en el checklist cuando lo esté.

## Temas de estudio

- [ ] [[Entrevista Java 01 - Perfil FullStack|1. Perfil FullStack]]
- [ ] [[Entrevista Java 02 - Angular|2. Angular]]
- [ ] [[Entrevista Java 03 - CI-CD e Integración Continua|3. CI/CD e Integración Continua]]
- [ ] [[Entrevista Java 04 - Agilidad y ceremonias|4. Agilidad y ceremonias]]
- [ ] [[Entrevista Java 05 - Programación funcional en Java|5. Programación funcional en Java]]
- [ ] [[Entrevista Java 06 - Concurrencia y Virtual Threads|6. Concurrencia y Virtual Threads]]
- [ ] [[Entrevista Java 07 - Spring Boot|7. Spring Boot]]
- [ ] [[Entrevista Java 08 - Seguridad - JWT, OAuth2 y Spring Security|8. Seguridad: JWT, OAuth2 y Spring Security]]
- [ ] [[Entrevista Java 09 - GCP - Pub-Sub, Functions, Cloud Run|9. GCP: Pub/Sub, Functions, Cloud Run]]
- [ ] [[Entrevista Java 10 - Kafka|10. Kafka]]
- [ ] [[Entrevista Java 11 - Firebase, Firestore y NoSQL|11. Firebase / Firestore y NoSQL]]
- [ ] [[Entrevista Java 12 - Docker|12. Docker]]
- [ ] [[Entrevista Java 13 - Kubernetes|13. Kubernetes]]
- [ ] [[Entrevista Java 14 - Patrones de diseño|14. Patrones de diseño]]
- [ ] [[Entrevista Java 15 - Arquitectura y buenas prácticas|15. Arquitectura y buenas prácticas]]
- [ ] [[Entrevista Java 16 - Preguntas de entrevista y respuestas modelo|16. Preguntas de entrevista y respuestas modelo]]
- [ ] [[Entrevista Java 17 - Negociación salarial|17. Negociación salarial]]
- [ ] [[Entrevista Java 18 - Plan de repaso de 7 días|18. Plan de repaso de 7 días]]
- [ ] [[Entrevista Java 19 - Cheat sheets|19. Cheat sheets]]

## Prioridades

1. **JWT / OAuth2 / Spring Security**: el temario dice «esto sí o sí».
2. Patrones de diseño: ten preparada la respuesta a «¿qué patrones de diseño has trabajado?».
3. Plan día a día: [[Entrevista Java 18 - Plan de repaso de 7 días|Plan de repaso de 7 días]].
4. Repaso final: [[Entrevista Java 19 - Cheat sheets|Cheat sheets]] y el checklist previo a la entrevista.

## Anexo: cómo continuar este documento con otra IA

Pega este prompt junto con el archivo `guia-entrevista.md`:

> "Este es mi documento de repaso para una entrevista técnica FullStack (Java/Spring Boot, Angular, GCP, Docker/Kubernetes, CI/CD, agilidad, seguridad JWT/OAuth2). Quiero que lo continúes manteniendo exactamente el mismo formato: secciones numeradas, tablas comparativas, bloques de código comentados, marcador 🎯 para lo más preguntado, y una subsección de 'preguntas típicas' al final de cada tema. Amplía [TEMA] con el mismo nivel de detalle y no reescribas lo ya existente."

Temas naturales para ampliar más adelante: pruebas de rendimiento, arquitectura hexagonal en detalle, GraphQL, WebSockets, observabilidad con OpenTelemetry, Terraform, algoritmos y estructuras de datos para la ronda de live coding, y system design (diseñar un acortador de URLs, un sistema de notificaciones, un checkout).

## Anexo B: comandos del Gem de repaso

Modos de operación del Gem **Sparring Técnico FullStack** (Gemini). Referencia detallada en `comandos-gem.md`; configuración del Gem en `gem-gemini.md`.

| Comando | Para qué sirve | Cuándo usarlo |
|---|---|---|
| `/diagnostico` | 8 preguntas de distintos temas + plan priorizado | Al empezar, o cada 3–4 días para medir avance |
| `/simulacro [tema] [nivel]` | Entrevista simulada, una pregunta a la vez, con nota y feedback | Cuando ya repasaste y quieres presión real |
| `/repasar <tema>` | Explicación completa con código y preguntas típicas | Tema que no manejas o que viste hace tiempo |
| `/profundizar <tema>` | Trade-offs, casos borde, qué falla en producción | Tema que ya entiendes y quieres defender a fondo |
| `/ampliar-guia <tema>` | Sección nueva en Markdown para pegar en este documento | Cuando detectas un hueco en la guía |
| `/flashcards <tema> [n]` | Tarjetas pregunta/respuesta de 3 líneas | Repaso rápido, metro, día previo |
| `/codigo <ejercicio>` | Live coding con solución comparada | Preparar la ronda de código |
| `/design <problema>` | System design guiado | Entrevistas de arquitectura o cargos senior |
| `/star <situación>` | Tu experiencia convertida en respuesta STAR con números | Preguntas de comportamiento |
| `/sueldo <situación>` | Guion textual de negociación, listo para decir | Antes de RRHH o al recibir una oferta |

## Rutinas

**Sesión de 30 minutos:** `/flashcards <tema> 10` → `/repasar <lo que fallaste>` → `/simulacro <ese tema>`.

**Día previo:** `/diagnostico` → `/flashcards seguridad 20` → `/sueldo` → `/simulacro` cronometrado.

**Al encontrar un hueco:** `/ampliar-guia <tema>` → pegar en `guia-entrevista.md` → `node build-html.js`.
