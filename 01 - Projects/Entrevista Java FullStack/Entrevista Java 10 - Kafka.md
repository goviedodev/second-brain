---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 10. Kafka

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 10.1 Conceptos 🎯

- **Topic:** canal lógico de eventos, dividido en **particiones**.
- **Partición:** log **ordenado e inmutable**; el orden se garantiza **dentro de una partición**, no en el topic completo.
- **Offset:** posición del consumidor en la partición. Kafka no borra al consumir: retiene por tiempo/tamaño.
- **Producer key:** determina la partición (`hash(key) % nPart`) → misma clave = misma partición = orden garantizado para esa entidad.
- **Consumer Group:** cada partición es consumida por **un solo consumidor** del grupo. Paralelismo máximo = número de particiones.
- **Rebalance:** al entrar o salir un consumidor se redistribuyen las particiones.
- **Broker / Cluster / Replication factor / ISR:** replicación entre brokers para tolerancia a fallos.

## 10.2 Garantías de entrega 🎯
- **At-most-once:** commit del offset antes de procesar (puedes perder mensajes).
- **At-least-once:** procesar y luego commitear (puedes duplicar) — **el más usado**.
- **Exactly-once (EOS):** productor idempotente + transacciones (`transactional.id`, `read_committed`). Tiene costo de rendimiento.

## 10.3 Kafka vs Pub/Sub 🎯

| | Kafka | Pub/Sub |
|---|---|---|
| Operación | Lo administras tú (o Confluent) | Totalmente gestionado |
| Orden | Por partición | Solo con ordering keys |
| Retención / replay | Log persistente, replay nativo por offset | Retención limitada + snapshots |
| Escalado | Manual (particiones) | Automático |
| Caso fuerte | Streaming, event sourcing, alto throughput | Desacople y eventos en GCP sin ops |

## 10.4 Spring Kafka

```java
@Component
public class PedidoConsumer {
    @KafkaListener(topics = "pedidos", groupId = "facturacion")
    public void consumir(ConsumerRecord<String, PedidoEvento> record, Acknowledgment ack) {
        try {
            servicio.procesar(record.value());
            ack.acknowledge();                  // commit manual
        } catch (Exception e) {
            log.error("Fallo offset {}", record.offset(), e);
            throw e;                            // va a retry/DLT
        }
    }
}

@Bean
public NewTopic pedidos() {
    return TopicBuilder.name("pedidos").partitions(6).replicas(3).build();
}
```
Configuración típica: `enable.auto.commit=false`, `ack-mode: MANUAL`, `DefaultErrorHandler` con `DeadLetterPublishingRecoverer`.

## 10.5 Preguntas frecuentes
- *"¿Cómo garantizas el orden de eventos de un cliente?"* → clave de partición = id del cliente.
- *"¿Qué pasa si hay más consumidores que particiones?"* → los sobrantes quedan ociosos.
- *"¿Cómo manejas un mensaje que siempre falla (poison pill)?"* → reintentos con backoff + Dead Letter Topic.
- *"¿Cómo evitas procesar dos veces?"* → idempotencia con id de evento o EOS transaccional.
