---
created: 2026-09-24
tags: [resource, blockchain, inteligencia-artificial, aprendizaje]
source: Jake Van Clief (Curso en Skool)
---

# Blockchain e Inteligencia Artificial — Jake Van Clief

## Contexto y Objetivo
- **Propósito:** Aprender los fundamentos y aplicaciones de blockchain para comprender a fondo la visión y el punto de vista de Jake Van Clief sobre la Inteligencia Artificial.
- **Plataforma:** Curso en Skool.
- **Cuenta / Registro:** `geoviedo.sevenit@gmail.com`.
- **Dinámica:** Nota viva que iremos completando y afinando de forma progresiva.

## Ejes a Explorar
- [[Blockchain]]: conceptos base, descentralización, incentivos y contratos inteligentes.
- [[Inteligencia Artificial]]: intersección con blockchain según la tesis de Jake Van Clief.
- Modelos de pensamiento, arquitectura y casos de uso planteados en el curso.

## Apuntes y Desarrollo

### Módulo: Contexto para Sistemas Agénticos (Triada de Archivos)
- **Fuente / Referencia:** [Lección en Skool (Cliefnotes)](https://www.skool.com/cliefnotes/classroom/036893d9?md=fdee3f73c53b46049078494c0cfb2e54)

#### Concepto Central
Para que cualquier agente de IA (como **Claude Code**, **Gemini**, Cursor, Roo Code, etc.) opere con máxima precisión y contexto, se estructura el proyecto con tres archivos fundamentales que definen el marco de trabajo completo:

1. **`AGENTS.md` (*Reglas* y Comportamiento):**
   - Instrucciones directas de cómo debe comportarse el agente.
   - Restricciones explícitas y directrices de lo que **debe evitar**.
   - Procedimientos paso a paso y estándares de codificación/operación.

Este es un ejemplo que puedes escribir:

 - Escribe en un lenguaje plano y claro.
  - Pregunta si no entiendes  algo antes de generar alucinaciones, asumir palabras, hechos, textos, etc.
   - Si estás inseguro, dilo, coméntalo o pregunta.

2. **`CONTEXT.md` (Visión del Proyecto):**
   - De qué se trata el proyecto en su totalidad.
   - Cómo luce la solución, su arquitectura y el alcance general.
   - Estado actual y visión macro.

 Aquí los puntos podrían ser:

-  Que estamos construyendo
-  Que lo hace correcto
-  Que debemos evitar.

3. **`REFERENCES.md` (Fuentes y Documentación):**
   - Documentación técnica, librerías, APIs y enlaces externos.
   - Modelos de datos, especificaciones o contratos previos de referencia.

 Aquí podemos tener lo siguiente:

- Ejemplos de un buen trabajo.
- Links relevantes.
-  Notas.

> [!tip] Dinámica Viva
> Estos archivos no son estáticos: **se editan y refinan en el momento que sea necesario**, permitiendo que el agente siempre mantenga el mejor contexto actualizado sin depender de prompts repetitivos o volátiles.

---

### Parte 1.3: Cómo estructurar cualquier Prompt
- **Fuente / Referencia:** [Lección en Skool (Cliefnotes)](https://www.skool.com/cliefnotes/classroom/036893d9?md=05230de8023d463f8d38fddc19152ae2)

#### Conexión con la Base de Contexto
Se apoya en la estructura inicial (Agente, Contexto y Referencias). Esto proporciona al agente la visión global de:
- Quién soy.
- Qué busco.
- De qué se trata el proyecto.
- Cómo debe lucir el resultado final.

#### Componentes del Prompt

1. **Identidad (Identity):**
   - Establece el perfil y rol del agente.
   - Formaliza el **vocabulario técnico**, el **nivel de profundidad** de las explicaciones y las **asunciones/premisas** que debe dar por sentadas.

2. **Tareas (Tasks) — ¿Qué se necesita hacer?:**
   Una tarea efectiva cuenta con tres características indispensables:
   - **Acción clara:** Verbo y operación concreta a ejecutar.
   - **Alcance definido:** Límites precisos de dónde empieza y dónde termina el trabajo.
   - **Detalle suficiente:** Claridad total para que alguien no familiarizado con el proyecto pueda comprenderla.

> [!important] La Prueba del Extraño (*Stranger Test*)
> Si le presentas la tarea a un extraño y no la entiende de inmediato o necesita hacerte preguntas de clarificación, **la tarea es demasiado vaga**. Debe tener la especificidad necesaria para ser autónoma y autoexplicativa.

3. **Contexto (Context) — ¿Qué necesita saber el agente?:**
   - Proporciona toda la información situacional que el modelo **no puede saber por sí mismo** (composición del equipo, tipo de cliente, reglas de negocio o estado del producto).
   - *Ejemplo ilustrativo:* *"Somos un equipo de 15 personas, nuestros clientes son directores de Recursos Humanos y hemos lanzado una funcionalidad que automatiza tareas diarias [detallando características específicas]"*.
   - **Principio operativo:** A mayor contexto relevante suministrado, menos margen tiene el agente (Claude Code, Gemini, etc.) para tener que **adivinar**.

> [!warning] El peligro de hacer que la IA adivine
> Si la IA se ve obligada a adivinar por falta de información, **se irá por las ramas, inventará requerimientos o tomará rumbos desenfocados**.
> 
> **Regla de oro:** *Si la respuesta o el código generado está fuera de foco, la solución casi siempre es **darle más contexto relevante**, no intentar redactar un prompt más rebuscado.*

4. **Restricciones (Constraints) — ¿Qué debe evitar el agente?:**
   - Establece los límites negativos: **decirle al agente explícitamente lo que NO quieres suele ser más útil y determinante que solo pedirle lo que quieres**.
   - Reduce drásticamente la tasa de error al podar caminos indeseados antes de que la IA empiece a generar.
   - *Ejemplos prácticos según el tipo de restricción:*
     - **Vocabulario y tono:** *"No uses jerga compleja; escribe esto adaptado para un estudiante de 8° básico"*.
     - **Herramientas y presupuesto:** *"No sugieras soluciones que requieran APIs o servicios de pago; utiliza únicamente herramientas gratuitas / open source"*.
     - **Formato y extensión:** *"Mantén la salida por debajo de 300 palabras"*, *"No utilices listas con viñetas o bullet points, redacta exclusivamente en párrafos corridos"*.
     - **Estilo y aperturas:** *"No inicies los textos con frases genéricas como 'Hola, ¿cómo estás?'; comienza siempre diciendo 'Hola, mi nombre es...'"*.

> [!tip] Regla clave: Las restricciones ahorran tiempo de edición
> **Cada restricción que colocas es un error que el agente no cometerá.** Las restricciones previenen el retrabajo y ahorran tiempo manual de corrección.
> 
> **Ejercicio práctico mental:** Recuerda las últimas veces que la IA cometió un fallo que te molestó: *esos errores que te frustraron son, en realidad, restricciones que tú no habías definido*. Incorpóralas a tus archivos de reglas y evitarás que se repitan.

5. **Formato de Salida (Output Format) — ¿Cómo debe verse el resultado?:**
   - Especifica con exactitud la estructura visual, sintáctica o esquemática de la respuesta final.
   - *Alternativas habituales según el objetivo:*
     - ¿Una **lista** o viñetas?
     - ¿Una **tabla** comparativa estructurada?
     - ¿Opciones o alternativas predefinidas para elegir?
     - ¿Un **bloque de código** con comentarios explicativos?
     - ¿Un esquema estructurado en **JSON** para integración técnica?
     - ¿Un formato exportable (Markdown, PDF, HTML)?
   - *Ejemplos prácticos:*
     - **Resumen de noticias:** *"Genera los titulares en menos de 10 palabras, seguidos inmediatamente de una viñeta con un resumen en una sola sentencia (estilo TL;DR)"*.
     - **Plantillas reutilizables (Templates):** *"Estructura el correo dejando placeholders entre corchetes para fácil reemplazo (ej. `[Mi nombre es: Gonzalo Oviedo / Patricio Oviedo]`, `[Empresa del cliente]`)"*.

> [!tip] Regla clave: El formato correcto elimina el retrabajo
> **Obtener la salida en el formato deseado desde el primer intento evita tener que reformatear manualmente cada respuesta.** Te ahorra tiempo continuo de post-procesamiento y deja el resultado listo para su uso directo.

---

### ¿Cuándo usar cada componente? (El Atajo Estratégico)

**No necesitas incluir los 5 componentes en todos los prompts.** Usar todos en tareas menores es sobreingeniería; omitirlos en tareas críticas genera respuestas deficientes. 

#### Matriz de Decisión Rápida

| Tipo de Tarea | Componentes recomendados | Razón / Enfoque |
| :--- | :--- | :--- |
| **Simple y rápida**<br>*(ej. renombrar una variable, corregir un typo)* | **Solo la Tarea** | Es una instrucción unívoca; no requiere contexto ni identidad para ejecutarse bien. |
| **Creativa**<br>*(ej. redactar un artículo, diseñar un flujo de interfaz)* | **Identidad + Tarea + Restricciones + Formato de Salida** | Necesita adoptar un tono específico, saber qué clichés o sesgos evitar y estructurar el diseño o texto final. |
| **Compleja**<br>*(ej. diseñar una arquitectura, analizar datos de negocio)* | **Los 5 componentes**<br>*(Identidad + Tarea + Contexto + Restricciones + Formato)* | Requiere panorama integral para no asumir nada ni desviarse del estado actual del sistema. |
| **Continua / Proyecto en marcha**<br>*(un proyecto a lo largo de muchas sesiones o mensajes)* | **Archivos locales (`.md`)** para Identidad y Contexto.<br>**Prompts puntuales** para Tarea, Restricciones del momento y Formato. | Evita el copy-paste constante y aprovecha la persistencia de los archivos del proyecto. |

---

#### El Puente entre la Carpeta y el Prompt

Aquí es donde se conectan la estructura de archivos (`AGENTS.md`, `CONTEXT.md`, `REFERENCES.md`) y el framework de prompting:

- **Tus archivos guardan lo persistente:** Quién eres, de qué trata el proyecto, las reglas generales y la arquitectura base.
- **Tus prompts entregan lo inmediato:** Qué hacer en este momento específico, qué evitar en esta iteración y la forma exacta de la respuesta.

> [!important] Principio Fundamental de los Sistemas Agénticos
> **«La carpeta es la memoria. El prompt es la dirección. Ambos trabajan juntos.»**

---

#### Ejemplos Prácticos de Aplicación

##### 1. Caso Simple (Solo Tarea)
> *"Corrige la ortografía y puntuación de este texto sin alterar el vocabulario ni la estructura de las oraciones."*

##### 2. Caso Creativo (Identidad + Tarea + Restricciones + Formato)
> - **Identidad:** *"Actúa como un copywriter senior especializado en software B2B"*.
> - **Tarea:** *"Escribe un correo en frío para directores de TI ofreciendo auditorías de seguridad agéntica"*.
> - **Restricciones:** *"No comiences con saludos de relleno tipo 'Espero que estés bien'; no uses adjetivos exagerados como 'revolucionario' o 'disruptivo'; no excedas las 120 palabras"*.
> - **Formato de Salida:** *"Estructura la respuesta con: Asunto (máx. 6 palabras), Cuerpo del correo con placeholders `[Nombre] / [Empresa]`, y una llamada a la acción clara en una sola frase"*.

##### 3. Caso Complejo (Los 5 Componentes)
> - **Identidad:** *"Eres un arquitecto de software de alta concurrencia"*.
> - **Contexto:** *"Tenemos una base de datos PostgreSQL con 2 millones de usuarios donde las alertas concurrentes están generando deadlocks en la tabla `notifications`"*.
> - **Tarea:** *"Diseña una estrategia de encolamiento y actualización optimista para procesar las alertas sin bloqueos pesados"*.
> - **Restricciones:** *"No sugieras migrar a NoSQL ni introducir Redis si podemos resolverlo a nivel de diseño SQL y SKIP LOCKED. No uses soluciones que requieran cambiar la infraestructura cloud actual"*.
> - **Formato:** *"Entrega una tabla con pros/contras de las alternativas evaluadas, seguida de un diagrama de flujo en texto y el bloque SQL DDL propuesto con comentarios"*.

##### 4. Caso Continuo (Proyecto con Archivos de Memoria)
> *(El agente ya leyó `AGENTS.md` y `CONTEXT.md` del repositorio, por lo que ya conoce el proyecto, el stack y las reglas)*:
> 
> *"Implementa la función de validación del token JWT en el middleware de autenticación. Restricción: No instales nuevas dependencias en `package.json`, usa únicamente las librerías existentes. Formato: Devuelve solo el archivo `auth.middleware.ts` listo para reemplazar."*
