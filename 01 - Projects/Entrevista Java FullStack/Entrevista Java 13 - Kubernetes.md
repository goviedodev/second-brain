---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 13. Kubernetes

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 13.1 Qué resuelve
Orquesta contenedores: despliegue declarativo, autoescalado, autorreparación, descubrimiento de servicios, balanceo y rolling updates.

## 13.2 Objetos esenciales 🎯

| Objeto | Para qué |
|---|---|
| **Pod** | Unidad mínima: uno o más contenedores que comparten red y volúmenes |
| **ReplicaSet** | Mantiene N réplicas de un pod |
| **Deployment** | Gestiona ReplicaSets: rolling updates y rollback |
| **Service** | IP estable y balanceo hacia los pods (ClusterIP / NodePort / LoadBalancer) |
| **Ingress** | Enrutamiento HTTP(S) externo, TLS, por host/path |
| **ConfigMap** | Configuración no sensible |
| **Secret** | Datos sensibles (base64, cifrar en reposo / usar Secret Manager) |
| **HPA** | Autoescalado horizontal por CPU/memoria/métricas |
| **Namespace** | Aislamiento lógico (dev/qa/prod) |
| **StatefulSet** | Cargas con estado e identidad estable (bases de datos) |
| **Job / CronJob** | Tareas puntuales o programadas |

## 13.3 Deployment comentado

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }   # cero downtime
  selector:
    matchLabels: { app: api }
  template:
    metadata:
      labels: { app: api }
    spec:
      containers:
        - name: api
          image: europe-docker.pkg.dev/proj/repo/api:1.4.2
          ports: [{ containerPort: 8080 }]
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: prod
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef: { name: db-secret, key: password }
          resources:
            requests: { cpu: "250m", memory: "512Mi" }   # lo que reserva
            limits:   { cpu: "1",    memory: "1Gi" }     # el techo
          readinessProbe:      # ¿puede recibir tráfico?
            httpGet: { path: /actuator/health/readiness, port: 8080 }
            initialDelaySeconds: 20
          livenessProbe:       # ¿hay que reiniciarlo?
            httpGet: { path: /actuator/health/liveness, port: 8080 }
            initialDelaySeconds: 40
---
apiVersion: v1
kind: Service
metadata: { name: api }
spec:
  selector: { app: api }
  ports: [{ port: 80, targetPort: 8080 }]
  type: ClusterIP
```

## 13.4 Probes 🎯
- **liveness:** si falla, Kubernetes **reinicia** el contenedor (proceso colgado).
- **readiness:** si falla, lo **saca del balanceador** sin reiniciarlo (aún calentando o dependencia caída).
- **startup:** para apps de arranque lento; evita que liveness mate el pod durante el boot.

Confundirlas es un error clásico: un liveness mal configurado provoca reinicios en cascada bajo carga.

## 13.5 HPA
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: api }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: api }
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 70 } }
```

## 13.6 Troubleshooting (lo que preguntan en entrevistas prácticas) 🎯
```bash
kubectl get pods -n prod
kubectl describe pod api-xxx          # eventos: por qué no arranca
kubectl logs -f api-xxx --previous    # logs del contenedor que murió
kubectl exec -it api-xxx -- sh
kubectl rollout status deployment/api
kubectl rollout undo deployment/api   # rollback
kubectl top pods                      # consumo
```

| Estado | Causa típica |
|---|---|
| `ImagePullBackOff` | Imagen/tag inexistente o sin credenciales del registry |
| `CrashLoopBackOff` | La app muere al arrancar (config, secreto faltante, puerto) |
| `Pending` | No hay nodo con recursos suficientes / PVC sin enlazar |
| `OOMKilled` | Excedió el `limits.memory` → ajustar límite o heap de la JVM |

## 13.7 Complementos
Helm (empaquetar manifiestos), Kustomize (overlays por entorno), ArgoCD/Flux (**GitOps**: el repo es la fuente de verdad del clúster), service mesh (Istio/Linkerd) para mTLS, retries y canary.
