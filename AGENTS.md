# AGENTS.md — Instrucciones para trabajar en este Vault

Este directorio es un **Vault de Obsidian** que funciona como un **Second Brain** según la metodología de **Tiago Forte** (*Building a Second Brain*): método **CODE** para el flujo de trabajo y **PARA** para la organización.

Cualquier agente (Claude, Codex, etc.) que trabaje aquí debe seguir estas reglas.

---

## 1. Estructura del Vault (PARA)

```
00 - Inbox/      → Captura. Todo lo nuevo entra aquí primero.
01 - Projects/   → Proyectos: esfuerzos con un objetivo concreto y una fecha límite.
02 - Areas/      → Áreas: responsabilidades continuas, sin fecha de término, con un estándar a mantener.
03 - Resources/  → Recursos: temas de interés o referencia útiles a futuro.
04 - Archive/    → Archivo: elementos inactivos de las tres categorías anteriores.
```

- **No crear nuevas carpetas en la raíz.** Dentro de cada categoría PARA sí se pueden crear subcarpetas (una por proyecto, área o tema), p. ej. `01 - Projects/Lanzar blog/`.
- `.obsidian/` es configuración de Obsidian: **no modificar** salvo que el usuario lo pida.
- `AGENTS.md` y `CLAUDE.md` son los únicos archivos permitidos en la raíz.

## 2. Captura (C de CODE) — guardar notas

**Regla principal: toda nota nueva que el agente cree se guarda en `00 - Inbox/`**, sin importar lo obvio que parezca su destino final. La clasificación se hace después, solo cuando el usuario pida depurar.

- Nombre de archivo: título descriptivo en español, legible por humanos, p. ej. `Idea app de gastos.md`. Evitar caracteres problemáticos (`/ \ : * ? " < > |`).
- Las notas diarias (`YYYY-MM-DD.md`) también viven en el Inbox (configurado en `.obsidian/daily-notes.json`).
- Si ya existe una nota con ese nombre, añadir contenido a la existente en lugar de sobrescribirla, o usar un nombre distinto. **Nunca sobrescribir notas del usuario.**
- Capturar lo que "resuena": ideas, citas, enlaces, aprendizajes, tareas. Mejor capturar rápido que perfecto.

### Formato de nota nueva

```markdown
---
created: YYYY-MM-DD
tags: [inbox]
source: <URL, libro, persona, o "propia">
---

# Título

Contenido.
```

- Usar enlaces internos de Obsidian `[[Nombre de nota]]` para conectar ideas; se permiten enlaces a notas que aún no existen.
- Los tags van en minúsculas y con guiones: `#desarrollo-personal`.

## 3. Organizar (O de CODE) — depurar el Inbox

Solo cuando el usuario lo pida ("depura", "procesa el inbox", "organiza las notas", etc.). Para cada nota del Inbox, decidir su destino con estas preguntas **en orden**, eligiendo la **primera** que aplique (*organizar por accionabilidad, no por tema*):

1. **¿Sirve para un proyecto activo?** (objetivo concreto + fecha límite) → `01 - Projects/<Proyecto>/`
2. **¿Sirve para un área de responsabilidad?** (salud, finanzas, trabajo, familia, casa…) → `02 - Areas/<Área>/`
3. **¿Es un tema de interés o referencia a futuro?** → `03 - Resources/<Tema>/`
4. **¿Nada de lo anterior / ya no es relevante?** → `04 - Archive/`

Procedimiento:

1. Listar las notas del Inbox y leer cada una completa.
2. Revisar primero las subcarpetas existentes en PARA para reutilizarlas antes de crear nuevas.
3. Si el destino es ambiguo o requiere crear un proyecto/área nuevo, **preguntar al usuario** en lugar de adivinar.
4. Presentar el plan de movimientos (nota → destino, con el motivo) y esperar confirmación antes de mover, salvo que el usuario haya pedido explícitamente hacerlo sin preguntar.
5. Al mover:
   - Usar `mv` conservando el nombre del archivo, para que Obsidian no rompa los `[[enlaces]]` (los wikilinks resuelven por nombre).
   - Actualizar el frontmatter: quitar el tag `inbox` y añadir `project`, `area` o `resource` según corresponda.
   - Las notas diarias con contenido mixto: extraer cada idea a su propia nota en el destino adecuado y dejar un `[[enlace]]` en la nota diaria; la nota diaria se archiva en `04 - Archive/Daily/` cuando ya no tenga nada pendiente.
6. Al terminar, reportar qué se movió y a dónde, y qué quedó en el Inbox (y por qué).

**Nunca borrar notas.** Lo que no sirve va a `04 - Archive/`.

## 4. Destilar (D de CODE)

Cuando el usuario pida resumir o destilar una nota, aplicar **Resumen Progresivo** (*Progressive Summarization*) sin eliminar el contenido original:

- Capa 1: nota original capturada.
- Capa 2: pasajes clave en **negrita**.
- Capa 3: lo más importante dentro de lo negrita, ==resaltado==.
- Capa 4: resumen ejecutivo con tus propias palabras al inicio de la nota, bajo `## Resumen`.

## 5. Expresar (E de CODE)

Cuando el usuario pida crear algo (artículo, presentación, plan), usar como materia prima las notas del Vault ("Intermediate Packets"), enlazarlas con `[[...]]`, y guardar el borrador resultante en el **Inbox** para que luego se clasifique (normalmente en el proyecto correspondiente).

## 6. Ciclo de vida y revisiones

- **Proyecto terminado o abandonado** → mover su carpeta completa de `01 - Projects/` a `04 - Archive/`.
- **Área que deja de ser responsabilidad** → `04 - Archive/`.
- **Revisión semanal** (si el usuario la pide): vaciar el Inbox, revisar proyectos activos, archivar lo terminado, y sugerir próximos pasos.
- **Revisión mensual**: revisar Áreas y Recursos, detectar proyectos nuevos o áreas desatendidas.

## 7. Reglas generales

- Idioma de las notas: **español**, salvo que el contenido original esté en otro idioma.
- Formato: Markdown compatible con Obsidian (wikilinks, frontmatter YAML, callouts `> [!note]`).
- No inventar contenido: si falta información, dejarlo explícito o preguntar.
- No mover, renombrar ni editar notas del usuario fuera de una depuración o de una petición explícita.
- Mantener las notas atómicas: una idea principal por nota cuando sea posible.
- **Comando "versiona":** Cada vez que el usuario indique la palabra o instrucción "versiona", se debe hacer automáticamente un `git commit` descriptivo de los cambios pendientes y un `git push` al repositorio remoto.

