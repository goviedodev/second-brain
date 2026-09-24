---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 7. Spring Boot

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 7.1 Por qué Spring Boot
Autoconfiguración, servidor embebido, *starters*, `application.yml` por perfil, Actuator para observabilidad. Convención sobre configuración.

## 7.2 Arquitectura en capas

```
Controller (REST, DTOs, validación)
   ↓
Service (lógica de negocio, @Transactional)
   ↓
Repository (Spring Data JPA)
   ↓
Entity / Base de datos
```
**Regla:** las entidades JPA **no** salen por el controller. Se mapean a DTOs (MapStruct o mapeo manual) para no filtrar el modelo interno ni provocar lazy-loading fuera de transacción.

## 7.3 Controller completo

```java
@RestController
@RequestMapping("/api/v1/pedidos")
@RequiredArgsConstructor
public class PedidoController {

    private final PedidoService service;

    @GetMapping("/{id}")
    public PedidoResponse obtener(@PathVariable Long id) {
        return service.obtener(id);
    }

    @GetMapping
    public Page<PedidoResponse> listar(@PageableDefault(size = 20) Pageable pageable) {
        return service.listar(pageable);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public PedidoResponse crear(@Valid @RequestBody CrearPedidoRequest req) {
        return service.crear(req);
    }
}
```

## 7.4 Manejo global de errores 🎯

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RecursoNoEncontradoException.class)
    public ResponseEntity<ApiError> noEncontrado(RecursoNoEncontradoException e) {
        return ResponseEntity.status(404).body(new ApiError("NOT_FOUND", e.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiError> validacion(MethodArgumentNotValidException e) {
        var detalles = e.getBindingResult().getFieldErrors().stream()
            .map(f -> f.getField() + ": " + f.getDefaultMessage())
            .toList();
        return ResponseEntity.badRequest().body(new ApiError("VALIDATION_ERROR", detalles));
    }
}
```
Nunca devolver el stacktrace al cliente: filtra información sensible.

## 7.5 Inyección de dependencias 🎯
- **Por constructor** (recomendado): permite `final`, facilita tests, detecta dependencias circulares al arrancar.
- Por setter: dependencias opcionales.
- Por campo (`@Autowired` en el atributo): desaconsejado, no testeable sin contenedor.

**Scopes:** `singleton` (default), `prototype`, `request`, `session`.

## 7.6 Transacciones 🎯
```java
@Transactional
public void transferir(Long origen, Long destino, BigDecimal monto) { ... }
```
- Por defecto hace rollback ante `RuntimeException`, **no** ante excepciones *checked* (`rollbackFor = Exception.class` para cambiarlo).
- **Self-invocation:** llamar a un método `@Transactional` desde otro método de la misma clase **no** activa el proxy → la transacción no se aplica. Pregunta clásica.
- Propagación: `REQUIRED` (default), `REQUIRES_NEW`, `SUPPORTS`, `MANDATORY`.

## 7.7 N+1 queries 🎯
Síntoma: una consulta por cada elemento de una lista.
Soluciones: `JOIN FETCH` en JPQL, `@EntityGraph`, `@BatchSize`, o proyecciones DTO.

```java
@Query("select p from Pedido p join fetch p.items where p.estado = :estado")
List<Pedido> buscarConItems(@Param("estado") Estado estado);
```

## 7.8 Perfiles y configuración
```yaml
# application.yml
spring:
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}
---
spring:
  config.activate.on-profile: prod
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}   # desde Secret Manager / env, nunca hardcodeado
```

## 7.9 Actuator y observabilidad
`/actuator/health` (liveness/readiness para Kubernetes), `/actuator/metrics`, `/actuator/prometheus`. Trazas distribuidas con Micrometer Tracing + OpenTelemetry.

## 7.10 Testing
- `@SpringBootTest` — contexto completo (lento, para integración).
- `@WebMvcTest` — solo la capa web, con `MockMvc`.
- `@DataJpaTest` — solo la capa de persistencia.
- **Testcontainers** — levanta PostgreSQL/Kafka real en Docker para tests de integración. Mencionarlo suma puntos.
