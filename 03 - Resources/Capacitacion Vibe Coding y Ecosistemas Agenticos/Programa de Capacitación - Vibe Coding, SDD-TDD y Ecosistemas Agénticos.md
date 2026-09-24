---
created: 2026-09-24
tags: [resource, capacitacion, vibe-coding, sdd, tdd, agentes-ia, azure-devops]
source: programa-capacitacion-vibe-coding.docx
---

# Programa de Capacitación: Vibe Coding, SDD/TDD y Ecosistemas Agénticos (15 Horas)

## 1. Objetivo del Curso
Capacitar al equipo de TI en metodologías de desarrollo agéntico, estandarizando el uso de **SDD** (*Specification-Driven Development*) y **TDD** (*Test-Driven Development*). Los participantes aprenderán a crear **Test Harnesses** (arneses de pruebas) para evaluar el código generado por IA y utilizarán el ecosistema de **Azure DevOps** en conjunto con agentes locales para construir y desplegar soluciones institucionales robustas y agnósticas al modelo de lenguaje.

---

## 2. Modalidad y Duración
- **Duración:** 15 horas cronológicas.
- **Formato:** Taller/Laboratorio 100% práctico (*Sugerencia: 5 sesiones de 3 horas*).

---

## 3. Entorno Tecnológico Esperado
- **IDEs potenciados por IA:** VSCode, Cursor, Roo Code/Cline integrados de forma local.
- **Gestión de repositorios y CI/CD:** Azure DevOps (Pipelines, Repos, Boards).
- **Bases de Datos:** Integración con motores de bases de datos SQL relacionales (PostgreSQL, MySQL o SQL Server) y/o servicios BaaS.
- **Contenedores:** Docker para el empaquetado de las soluciones.

---

## 4. Estructura Curricular (15 Horas)

### Módulo I: Fundamentos Agénticos y Ecosistema de Trabajo (2.5 Horas)
- Transición de desarrollador tradicional a "Orquestador": El paradigma del *Vibe Coding*.
- Configuración del entorno de desarrollo local integrado con los repositorios de Azure DevOps.
- Estrategias de Prompting para desarrollo (Las 3 C's, *Prompt Chaining*).

### Módulo II: SDD y Harness Engineering (Ingeniería de Arneses) (3.5 Horas)
- **Specification-Driven Development (SDD):** Redacción de requerimientos como el "Contexto Supremo" para los agentes.
- **Harness Engineering y TDD:** Diseño de entornos de pruebas (*Test Harnesses*) automatizados. Cómo construir el andamiaje de aserciones lógicas, mocks y validaciones perimetrales antes de que la IA genere la lógica de negocio, obligando al modelo a iterar hasta cumplir las pruebas.

### Módulo III: Ampliación de Capacidades (Skills y MCP) (3 Horas)
- Uso de **Skills** para estandarizar y automatizar tareas repetitivas del equipo.
- Implementación de **Model Context Protocol (MCP)**: Conectar a los agentes con fuentes de datos locales, motores SQL de la institución y documentación interna para enriquecer el contexto.

### Módulo IV: Orquestación Avanzada y Despliegue CI/CD (2.5 Horas)
- Gestión de múltiples tareas en paralelo (Orquestación tipo *Firstmate*): agentes auditando código mientras otros generan pruebas o documentan.
- **Seguridad (Checklist ACT)** y auditoría del código generado para prevenir alucinaciones lógicas o inyecciones.
- Conexión del flujo agéntico con pipelines de Azure DevOps para construcción de contenedores Docker y despliegue automatizado.

### Módulo V: Proyecto Práctico Institucional (Hands-On) (3.5 Horas)
- **Desarrollo End-to-End:** Todo el equipo trabajará en un micro-proyecto estandarizado para "hablar el mismo idioma" técnico.
- **Caso Práctico Dinámico:** El requerimiento específico a desarrollar será definido durante el transcurso del curso mediante un acuerdo entre la Jefatura de TI y el relator.
- **Complejidad Requerida:** Independiente del caso de uso seleccionado, el requerimiento técnico exigirá que el agente IA resuelva e implemente una lógica de negocio compleja que involucre concurrencia en bases de datos SQL y la generación de alertas dinámicas sin realizar bloqueos estrictos (por ejemplo, cruce de datos y notificaciones concurrentes).
- Construcción de la interfaz, el backend SQL y su empaquetado en contenedores.

---

## 5. Entregables Esperados por los Alumnos
1. Repositorio en Azure DevOps con el código fuente del sistema desarrollado.
2. Documentación SDD generada para el proyecto.
3. Suite de pruebas automatizadas (*Test Harness*) funcional.
4. Archivo de configuración de entorno y despliegue (Dockerfile / Pipeline).
