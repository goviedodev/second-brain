---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 5. Programación funcional en Java

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 5.1 Base: interfaces funcionales 🎯

| Interfaz | Firma | Uso |
|---|---|---|
| `Function<T,R>` | `R apply(T)` | `map` |
| `Predicate<T>` | `boolean test(T)` | `filter` |
| `Consumer<T>` | `void accept(T)` | `forEach` |
| `Supplier<T>` | `T get()` | lazy, fábricas |
| `BiFunction<T,U,R>` | `R apply(T,U)` | combinaciones |
| `UnaryOperator<T>` | `T apply(T)` | transformación mismo tipo |

## 5.2 Streams: `filter`, `map`, `reduce`, `collect`

```java
record Pedido(String id, String cliente, BigDecimal monto, Estado estado) {}

// filter + map + collect
List<String> idsPagados = pedidos.stream()
    .filter(p -> p.estado() == Estado.PAGADO)      // Predicate
    .map(Pedido::id)                                // Function
    .toList();

// reduce: suma de montos
BigDecimal total = pedidos.stream()
    .map(Pedido::monto)
    .reduce(BigDecimal.ZERO, BigDecimal::add);

// agrupar
Map<Estado, List<Pedido>> porEstado = pedidos.stream()
    .collect(Collectors.groupingBy(Pedido::estado));

// agrupar + contar
Map<String, Long> porCliente = pedidos.stream()
    .collect(Collectors.groupingBy(Pedido::cliente, Collectors.counting()));

// flatMap: aplanar listas anidadas
List<Item> items = pedidos.stream()
    .flatMap(p -> p.items().stream())
    .toList();
```

## 5.3 Operaciones intermedias vs terminales 🎯

- **Intermedias (lazy):** `filter`, `map`, `flatMap`, `sorted`, `distinct`, `limit`, `peek`. No ejecutan nada hasta que llega una terminal.
- **Terminales:** `collect`, `toList`, `forEach`, `reduce`, `count`, `anyMatch`, `findFirst`.

> Pregunta trampa: *"¿Qué imprime un stream con solo `filter` y `map`?"* → **Nada.** Sin operación terminal no se evalúa.

## 5.4 Optional — cómo usarlo bien

```java
// MAL: Optional con get() es igual de peligroso que null
String nombre = repo.findById(id).get();

// BIEN
String nombre = repo.findById(id)
    .map(Usuario::nombre)
    .orElseThrow(() -> new UsuarioNoEncontradoException(id));

// Con valor por defecto perezoso
Config c = buscar(id).orElseGet(Config::porDefecto);
```
Reglas: `Optional` se usa como **retorno**, no como parámetro ni como campo de entidad.

## 5.5 "Funcional Lambda" (del temario) — dos lecturas
El apunte `Funcional Lambda / Programación Funcional / filter, map` mezcla dos cosas que conviene separar en la entrevista:

1. **Lambdas y streams de Java** (lo de arriba): estilo declarativo, sin mutación, funciones como valores.
2. **Functions as a Service** (Cloud Functions en GCP, Lambda en AWS): unidad de cómputo sin servidor, disparada por eventos (ver §9.3).

Si te preguntan "¿has trabajado con funcional/lambda?", aclara cuál de las dos: *"En Java uso ampliamente lambdas y Streams; en GCP he trabajado Cloud Functions disparadas por Pub/Sub, que es el equivalente a AWS Lambda."*

## 5.6 Principios funcionales que puedes citar
- **Inmutabilidad:** `record`, `List.copyOf`, no mutar el objeto de entrada → menos bugs, seguro en concurrencia.
- **Funciones puras:** mismo input → mismo output, sin efectos secundarios → testeables.
- **Composición:** `f.andThen(g)`, `predicado.and(otro)`.
- **Declarativo sobre imperativo:** el *qué* en lugar del *cómo*.

## 5.7 Cuándo NO usar streams
Loops con mucha mutación de estado, salidas tempranas complejas, o código en un hot path donde el `for` es más legible y rápido. Un stream ilegible es peor que un `for` claro.
