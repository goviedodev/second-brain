---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 14. Patrones de diseño

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

> El temario dice literalmente **"¿Qué patrones de diseño has trabajado?"**. Prepara **3 patrones que realmente hayas usado**, con el problema concreto que resolviste. Es mucho mejor que recitar los 23 del GoF.

## 14.1 Creacionales

| Patrón | Problema que resuelve | Ejemplo real |
|---|---|---|
| **Singleton** | Una única instancia compartida | Los beans de Spring son singleton por defecto |
| **Factory Method** | Crear sin acoplar al tipo concreto | `ProcesadorPagoFactory.crear(tipo)` |
| **Abstract Factory** | Familias de objetos relacionados | Drivers por proveedor cloud |
| **Builder** | Construir objetos con muchos parámetros opcionales | `Pedido.builder().cliente(x).item(y).build()`, Lombok `@Builder` |
| **Prototype** | Clonar objetos costosos | Copias de configuraciones base |

```java
// Builder — el más citable en Java
Pedido pedido = Pedido.builder()
    .cliente("ana")
    .items(List.of(item1, item2))
    .cupon("VERANO20")
    .build();
```

## 14.2 Estructurales

| Patrón | Problema | Ejemplo |
|---|---|---|
| **Adapter** | Adaptar una interfaz externa a la tuya | Envolver el SDK de un banco tras tu propia interfaz |
| **Decorator** | Agregar comportamiento sin heredar | `BufferedReader`, caching sobre un repositorio |
| **Facade** | Simplificar un subsistema complejo | Un `PagoService` que esconde 4 integraciones |
| **Proxy** | Controlar el acceso | Proxies de Spring AOP (`@Transactional`, `@Cacheable`) |
| **Composite** | Árboles de objetos tratados uniformemente | Menús, estructuras de permisos |

## 14.3 De comportamiento

| Patrón | Problema | Ejemplo |
|---|---|---|
| **Strategy** | Intercambiar algoritmos en runtime | Medios de pago, reglas de descuento |
| **Observer** | Notificar a N interesados | `ApplicationEventPublisher` de Spring, RxJS |
| **Template Method** | Esqueleto fijo con pasos variables | `JdbcTemplate`, clases abstractas de proceso |
| **Chain of Responsibility** | Cadena de manejadores | Filtros de Spring Security, interceptores |
| **Command** | Encapsular una acción como objeto | Tareas encoladas, undo |
| **State** | Comportamiento según estado | Máquina de estados de un pedido |

**Strategy con Spring — el ejemplo que mejor queda en entrevista:**

```java
public interface ProcesadorPago {
    boolean soporta(MedioPago medio);
    Recibo procesar(Pago pago);
}

@Component class ProcesadorTarjeta implements ProcesadorPago { ... }
@Component class ProcesadorTransferencia implements ProcesadorPago { ... }

@Service
@RequiredArgsConstructor
public class PagoService {
    private final List<ProcesadorPago> procesadores;   // Spring inyecta todas

    public Recibo pagar(Pago pago) {
        return procesadores.stream()
            .filter(p -> p.soporta(pago.medio()))
            .findFirst()
            .orElseThrow(() -> new MedioNoSoportadoException(pago.medio()))
            .procesar(pago);
    }
}
```
> Valor que comunica: *"agregar un medio de pago nuevo es crear una clase; no toco el código existente"* → **principio abierto/cerrado**.

## 14.4 Patrones de arquitectura y microservicios 🎯

| Patrón | Qué resuelve |
|---|---|
| **Circuit Breaker** | Deja de llamar a un servicio caído para no propagar la falla (Resilience4j) |
| **Retry + Backoff** | Reintenta fallas transitorias con espera creciente y jitter |
| **Bulkhead** | Aísla recursos para que un servicio lento no consuma todos los hilos |
| **API Gateway** | Punto único de entrada: auth, rate limiting, routing |
| **Saga** | Transacciones distribuidas por pasos con compensación (no hay 2PC) |
| **Outbox** | Publicar eventos de forma consistente con la escritura en BD |
| **CQRS** | Separar el modelo de escritura del de lectura |
| **Event Sourcing** | Guardar los eventos, no solo el estado final |
| **Sidecar / Ambassador** | Funcionalidad transversal junto al contenedor principal |
| **Strangler Fig** | Migrar un monolito por partes sin un big-bang |
| **Idempotency Key** | Evitar efectos duplicados en reintentos |

```java
@CircuitBreaker(name = "bancoApi", fallbackMethod = "fallback")
@Retry(name = "bancoApi")
public Saldo consultar(String cuenta) { return client.get(cuenta); }

private Saldo fallback(String cuenta, Throwable t) {
    return Saldo.noDisponible();   // degradación elegante
}
```

## 14.5 SOLID (te lo pueden pedir junto con patrones) 🎯
- **S**ingle Responsibility — una razón para cambiar.
- **O**pen/Closed — abierto a extensión, cerrado a modificación (Strategy).
- **L**iskov — un subtipo debe poder sustituir a su tipo base sin romper nada.
- **I**nterface Segregation — interfaces pequeñas y específicas.
- **D**ependency Inversion — depender de abstracciones, no de implementaciones (la base de la DI de Spring).

## 14.6 Anti-patrones que puedes nombrar
God Object, Anemic Domain Model, Singleton con estado mutable global, inyección por campo, `catch (Exception e) {}` vacío, *lasagna/spaghetti architecture*, microservicios distribuidos que comparten base de datos.
