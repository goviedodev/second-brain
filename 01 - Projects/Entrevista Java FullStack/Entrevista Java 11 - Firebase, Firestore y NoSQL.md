---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 11. Firebase / Firestore y NoSQL

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 11.1 Qué es
**Firestore** es una base de datos **NoSQL documental**, serverless, con escalado automático, sincronización en tiempo real y soporte offline. Parte de Firebase y disponible también dentro de GCP.

**Jerarquía:** `Colección → Documento → Campos` (y subcolecciones anidadas).

```
usuarios (colección)
  └── user_123 (documento)
        ├── nombre: "Ana"
        ├── email: "ana@x.cl"
        └── pedidos (subcolección)
              └── ped_1 { total: 15000, estado: "PAGADO" }
```

## 11.2 SQL vs NoSQL 🎯

| | Relacional (PostgreSQL) | Documental (Firestore) |
|---|---|---|
| Esquema | Rígido, migraciones | Flexible por documento |
| Relaciones | JOIN nativo | Sin JOIN: se desnormaliza o se hacen varias lecturas |
| Transacciones | ACID completo | Transacciones y batch, con límites |
| Consultas | SQL arbitrario, agregaciones | Consultas limitadas, requieren índices |
| Escalado | Vertical, réplicas de lectura | Horizontal automático |
| Cuándo | Datos relacionales, reportes, integridad | Alta escala, tiempo real, esquema cambiante |

**Regla mental:** en SQL modelas según **cómo se relacionan los datos**; en NoSQL modelas según **cómo vas a consultarlos**.

## 11.3 Modelado en Firestore
- **Desnormalizar** es normal: duplicar el nombre del usuario dentro del pedido para evitar una segunda lectura.
- Documento máximo 1 MiB; evitar arrays que crecen sin límite.
- **Hotspotting:** IDs secuenciales (timestamps monotónicos) concentran escrituras en un rango → usar IDs aleatorios.
- **Contadores distribuidos** (sharded counters) cuando hay muchas escrituras al mismo documento (límite ~1 escritura/seg por documento).

## 11.4 Consultas e índices

```javascript
// Angular / Web SDK
const q = query(
  collection(db, 'pedidos'),
  where('estado', '==', 'PAGADO'),
  where('total', '>=', 10000),
  orderBy('total', 'desc'),
  limit(20)
);
const snap = await getDocs(q);
```
- Índices **simples automáticos**; **compuestos manuales** (la consola te da el link para crearlos cuando falla).
- Sin `OR` nativo entre campos distintos (se resuelve con múltiples consultas o `in`).
- Paginación con `startAfter(ultimoDoc)`, no con offset.
- **Costo por operación**: se paga por lectura/escritura/borrado de documento, no por CPU. Consultar con `limit` no es solo rendimiento: es dinero.

## 11.5 Tiempo real y offline
```javascript
onSnapshot(q, snap => {
  this.pedidos.set(snap.docs.map(d => ({ id: d.id, ...d.data() })));
});
```
El SDK mantiene caché local y reintenta cuando vuelve la conexión — es la razón principal para elegir Firestore en apps móviles.

## 11.6 Security Rules 🎯
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /usuarios/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    match /pedidos/{pedidoId} {
      allow read: if request.auth != null
                  && resource.data.usuarioId == request.auth.uid;
      allow create: if request.auth != null
                    && request.resource.data.total is number;
    }
  }
}
```
Las reglas son la capa de autorización cuando el cliente accede **directamente** a Firestore. Si el acceso pasa por tu backend con el Admin SDK, las reglas **se saltan** y la autorización es responsabilidad del backend.

## 11.7 Otros servicios Firebase que suelen aparecer
Authentication (integra con OIDC/Google/email), Cloud Messaging (push), Remote Config (feature flags), Hosting, Crashlytics, Cloud Functions for Firebase.
