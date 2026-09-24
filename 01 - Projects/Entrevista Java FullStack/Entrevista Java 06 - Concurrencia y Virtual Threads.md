---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 6. Concurrencia y Virtual Threads

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 6.1 Modelo clásico 🎯

- **Thread de plataforma** = hilo del SO. Costoso (~1 MB de stack). Por eso usamos **pools** (`ExecutorService`) en vez de crear hilos a mano.
- Problema histórico: en aplicaciones web **la mayoría del tiempo el hilo está bloqueado esperando I/O** (base de datos, HTTP). Con 200 hilos en el pool, 200 requests concurrentes lentos saturan el servidor.
- Soluciones previas: programación reactiva (WebFlux, `Mono`/`Flux`) — escala, pero el código es difícil de leer y depurar.

```java
ExecutorService pool = Executors.newFixedThreadPool(10);
Future<String> f = pool.submit(() -> servicio.llamar());
String r = f.get();           // bloquea
pool.shutdown();
```

## 6.2 Virtual Threads (Project Loom, estable desde Java 21) 🎯

**Qué son:** hilos ligerísimos gestionados por la JVM, no por el SO. Puedes tener **millones**. Cuando un virtual thread se bloquea en I/O, la JVM lo "desmonta" del hilo portador (carrier thread) y deja ese hilo del SO libre para otro virtual thread.

**Por qué importan:** permiten escribir código **bloqueante y secuencial** (fácil de leer y depurar) con la escalabilidad del modelo reactivo.

```java
// Crear uno
Thread.startVirtualThread(() -> procesar(pedido));

// Executor: un virtual thread por tarea
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<Resultado>> futuros = pedidos.stream()
        .map(p -> executor.submit(() -> procesar(p)))
        .toList();
}   // el close() espera a que todas terminen
```

**En Spring Boot 3.2+** se habilita con una línea:
```properties
spring.threads.virtual.enabled=true
```
Cada request HTTP pasa a atenderse en un virtual thread.

## 6.3 Trampas que te pueden preguntar 🎯

| Tema | Detalle |
|---|---|
| **No hacer pool de virtual threads** | Son baratos: uno por tarea. Un pool fijo anula el beneficio. |
| **Pinning** | Si el hilo se bloquea dentro de un `synchronized` (o JNI), queda "clavado" al carrier thread. Solución: usar `ReentrantLock`. (En Java 24+ el pinning por `synchronized` se eliminó). |
| **No sirven para CPU-bound** | Su ventaja es I/O. Para cálculo intensivo el límite siguen siendo los núcleos. |
| **ThreadLocal** | Sigue funcionando, pero con millones de hilos puede ser costoso; alternativa: `ScopedValue`. |
| **El cuello de botella se mueve** | Si atiendes 10.000 requests pero el pool de conexiones a BD tiene 20, el problema ahora es la BD. |

## 6.4 Structured Concurrency (preview)

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var usuario = scope.fork(() -> usuarioService.buscar(id));
    var pedidos = scope.fork(() -> pedidoService.listar(id));
    scope.join().throwIfFailed();
    return new Vista(usuario.get(), pedidos.get());
}   // si una falla, la otra se cancela automáticamente
```

## 6.5 Fundamentos que siguen cayendo en entrevista
- **`synchronized` vs `ReentrantLock`:** el segundo permite `tryLock`, timeout, equidad e interrupción.
- **`volatile`:** garantiza visibilidad entre hilos, **no** atomicidad.
- **Atomics:** `AtomicInteger`, `LongAdder` para contadores sin bloqueo (CAS).
- **Colecciones concurrentes:** `ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue`.
- **Race condition / deadlock / starvation:** define los tres y cómo los evitas (orden consistente de locks, timeouts, inmutabilidad).
- **`CompletableFuture`:** composición asíncrona (`thenApply`, `thenCompose`, `allOf`).
