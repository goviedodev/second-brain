---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 9. GCP: Pub/Sub, Functions, Cloud Run

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 9.1 Servicios que debes ubicar

| Categoría | Servicio | Uso |
|---|---|---|
| Cómputo | Compute Engine / GKE / **Cloud Run** / **Cloud Functions** | VM / K8s gestionado / contenedor serverless / función serverless |
| Mensajería | **Pub/Sub** | Eventos asíncronos, desacople |
| Datos | Cloud SQL, **Firestore**, BigQuery, Bigtable, Spanner | Relacional / documental / analítica |
| Almacenamiento | Cloud Storage (GCS) | Archivos, buckets |
| Seguridad | IAM, **Secret Manager**, KMS | Permisos, secretos, llaves |
| Entrega | Cloud Build, **Artifact Registry** | CI/CD, imágenes |
| Observabilidad | Cloud Logging, Monitoring, Trace | Logs, métricas, trazas |

## 9.2 Pub/Sub a fondo 🎯

**Modelo:** *Publisher* → **Topic** → **Subscription(s)** → *Subscriber*. Es **pub/sub** (uno a muchos): cada suscripción recibe una copia de cada mensaje.

```
Publisher ──► Topic ──┬──► Subscription A ──► Cloud Run (push)
                      └──► Subscription B ──► Worker (pull)
```

**Conceptos clave:**
- **Push vs Pull:** en *push* Pub/Sub hace un POST a tu endpoint HTTPS; en *pull* tu servicio pide mensajes. Push es cómodo con Cloud Run; pull da mejor control de flujo.
- **Ack / Nack y `ackDeadline`:** si no confirmas (ack) dentro del plazo, el mensaje se re-entrega.
- **At-least-once:** el modo por defecto **puede entregar duplicados** → tu consumidor **debe ser idempotente**. (Existe *exactly-once delivery* dentro de una región, con restricciones).
- **Orden:** no garantizado salvo que actives *ordering keys* (mensajes con la misma clave llegan en orden).
- **Dead Letter Topic (DLQ):** tras N intentos fallidos el mensaje va a un topic muerto para inspección.
- **Retención y replay:** retención configurable (hasta 7 días por defecto, más con snapshots) y `seek` para reprocesar.

**Publicar y consumir desde Spring:**
```java
// Publicar
@Autowired PubSubTemplate pubSubTemplate;
pubSubTemplate.publish("pedidos-topic", new ObjectMapper().writeValueAsString(evento));

// Consumir (pull con subscriber)
pubSubTemplate.subscribe("pedidos-sub", message -> {
    var evento = parse(message.getPubsubMessage().getData().toStringUtf8());
    if (yaProcesado(evento.id())) { message.ack(); return; }   // idempotencia
    procesar(evento);
    message.ack();
});
```

**Idempotencia — el patrón que debes nombrar:** guardar el `messageId` (o un id de negocio) en una tabla/colección de "procesados" y descartar duplicados; o hacer que la operación sea naturalmente idempotente (`UPSERT` en vez de `INSERT`).

## 9.3 Cloud Functions ("Funcional Lambda")

FaaS: código que se ejecuta ante un evento, sin administrar servidores. Escala a cero.

```java
public class ProcesarPedido implements BackgroundFunction<PubSubMessage> {
    @Override
    public void accept(PubSubMessage message, Context context) {
        String data = new String(Base64.getDecoder().decode(message.data), UTF_8);
        // procesar
    }
}
```

**Disparadores:** HTTP, Pub/Sub, Cloud Storage (archivo subido), Firestore (documento creado/actualizado), Cloud Scheduler (cron).

**Limitaciones que debes conocer:** *cold start* (mitigable con min-instances), timeout máximo, statelessness, límite de memoria. Para cargas más grandes o contenedores propios: **Cloud Run**.

**Cloud Functions vs Cloud Run:** Functions = una función, un evento, runtime gestionado. Cloud Run = tu contenedor Docker, cualquier lenguaje, HTTP o eventos, más control, también escala a cero. Para una app Spring Boot dockerizada, **Cloud Run** es la opción natural.

## 9.4 IAM en una línea
`Principal` (usuario/service account) + `Role` (conjunto de permisos) + `Resource`. Principio de **mínimo privilegio**: una service account por servicio, con los roles justos. Nunca uses la cuenta por defecto con rol Editor en producción.

## 9.5 Equivalencias GCP ↔ AWS (por si preguntan)

| GCP | AWS |
|---|---|
| Cloud Functions | Lambda |
| Cloud Run | Fargate / App Runner |
| Pub/Sub | SNS + SQS |
| Firestore | DynamoDB |
| Cloud Storage | S3 |
| GKE | EKS |
| Cloud SQL | RDS |
| Secret Manager | Secrets Manager |
| Artifact Registry | ECR |
