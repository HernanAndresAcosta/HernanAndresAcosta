# ☸️ Desafío N°11 — Kubernetes: Migración de Docker Compose a Minikube
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT  
**Fecha:** 09/04/2025

---

## 📌 Descripción

Migración de la aplicación **NestJS + MongoDB** (previamente contenerizada con Docker Compose en el Desafío N°10) a un entorno **Kubernetes** ejecutado localmente con **Minikube**. Se definen e implementan manifiestos YAML para Deployments y Services, con buenas prácticas de límites de recursos y separación de responsabilidades.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **Kubernetes (Minikube)** | Orquestador de contenedores local |
| **kubectl** | CLI para gestión del clúster |
| **Docker Hub** | Registro de imágenes |
| **NestJS** | Aplicación (`hernan1305/educacionit-app:v1`) |
| **MongoDB 7.0** | Base de datos (`mongo:7.0`) |
| **Minikube Dashboard** | Interfaz gráfica de monitoreo |

---

## 🏗️ Arquitectura Kubernetes

```
Minikube (nodo único)
────────────────────────────────────────────────
  Namespace: default

  ┌─────────────────────────────────────┐
  │  Deployment: educacionit-app        │
  │  Image: hernan1305/educacionit-app:v1│
  │  Replicas: 1                        │
  │  Port: 3000                         │
  └──────────────┬──────────────────────┘
                 │
  ┌──────────────▼──────────────────────┐
  │  Service: educacionit-app-service   │
  │  Type: NodePort                     │
  │  ClusterIP: 10.108.155.205          │
  │  Port: 3000 → NodePort: 30538       │
  └──────────────┬──────────────────────┘
                 │ acceso externo
         http://192.168.49.2:30538

  ┌─────────────────────────────────────┐
  │  Deployment: mongo                  │
  │  Image: mongo:7.0                   │
  │  Replicas: 1                        │
  │  Port: 27017                        │
  └──────────────┬──────────────────────┘
                 │
  ┌──────────────▼──────────────────────┐
  │  Service: mongo-service             │
  │  Type: ClusterIP (interno)          │
  │  ClusterIP: 10.110.84.46            │
  │  Port: 27017                        │
  └─────────────────────────────────────┘
```

---

## 📁 Estructura del Proyecto

```
educacionit-app/
├── kubernetes/
│   ├── app-deployment.yaml           ← Deployment de NestJS
│   ├── app-service.yaml              ← Service NodePort para acceso externo
│   ├── mongo-deployment.yaml         ← Deployment de MongoDB
│   ├── mongo-service.yaml            ← Service ClusterIP interno
│   ├── educacionit-app-exported.yaml ← Manifiesto exportado real (limpio)
│   └── mongo-exported.yaml           ← Manifiesto exportado real (limpio)
├── src/
├── Dockerfile
├── docker-compose.yaml
└── package.json
```

---

## ⚙️ Manifiestos Kubernetes

### `app-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: educacionit-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: educacionit-app
  template:
    metadata:
      labels:
        app: educacionit-app
    spec:
      containers:
        - name: educacionit-app
          image: hernan1305/educacionit-app:v1
          ports:
            - containerPort: 3000
          env:
            - name: PORT
              value: "3000"
            - name: MONGO_DB_URI
              value: "mongodb://mongo-service:27017"
          resources:
            limits:
              cpu: "500m"
              memory: "256Mi"
            requests:
              cpu: "250m"
              memory: "128Mi"
```

### `app-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: educacionit-app-service
spec:
  type: NodePort
  selector:
    app: educacionit-app
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30538
```

### `mongo-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:7.0
          ports:
            - containerPort: 27017
```

### `mongo-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  type: ClusterIP    # Solo accesible internamente
  selector:
    app: mongo
  ports:
    - port: 27017
      targetPort: 27017
```

---

## 🚀 Proceso de Despliegue

### 1. Publicar imagen en Docker Hub
```bash
docker build -t hernan1305/educacionit-app:v1 .
docker push hernan1305/educacionit-app:v1
```

### 2. Aplicar manifiestos en Minikube
```bash
kubectl apply -f kubernetes/

# deployment.apps/educacionit-app created
# service/educacionit-app-service created
# deployment.apps/mongo created
# service/mongo-service created
```

### 3. Verificar recursos activos
```bash
kubectl get pods
# NAME                                 READY   STATUS    RESTARTS   AGE
# educacionit-app-df5f967f9-dz4gv      1/1     Running   0          3m14s
# mongo-59b865bc45-kllx4               1/1     Running   0          3m14s

kubectl get services
# NAME                      TYPE        CLUSTER-IP       PORT(S)
# educacionit-app-service   NodePort    10.108.155.205   3000:30538/TCP
# mongo-service             ClusterIP   10.110.84.46     27017/TCP
```

### 4. Acceder a la aplicación
```bash
minikube service educacionit-app-service
# URL: http://192.168.49.2:30538 → "Hello World!" ✅
```

---

## 📊 Monitoreo con Minikube Dashboard

```bash
minikube dashboard
```

Desde el dashboard se verificó:
- **Pods:** `educacionit-app` y `mongo` en estado `Running` ✅
- **Services:** `NodePort` para la app (acceso externo) y `ClusterIP` para MongoDB (interno)
- **Nodo:** Minikube v1.35 — 1 nodo — CPU 1,25/4,00 — Memory 426Mi/682Mi

---

## 🧹 Manifiestos Exportados y Limpios

Se exportaron los manifiestos reales desde Kubernetes y se limpiaron los metadatos dinámicos innecesarios:

```bash
kubectl get deployment educacionit-app -o yaml > educacionit-app-exported.yaml
kubectl get deployment mongo -o yaml > mongo-exported.yaml
```

**Campos eliminados** (no necesarios para reutilizar el manifiesto):

| Campo | Motivo |
|-------|--------|
| `metadata.annotations` | Solo auditoría |
| `metadata.uid` | Generado automáticamente cada vez |
| `metadata.creationTimestamp` | Cambia constantemente |
| `status: ...` | Estado actual, no configuración |
| `resourceVersion`, `generation` | Internos de control de K8s |

---

## 🧠 Conceptos Aplicados

- **Deployment** — gestión declarativa de pods con réplicas y rolling updates
- **Service NodePort** — exposición externa de la app desde Minikube
- **Service ClusterIP** — comunicación interna segura entre app y MongoDB
- **Resource limits** — CPU y memoria máximos por contenedor
- **Resource requests** — recursos garantizados para el scheduling
- **Docker Hub** como registro de imágenes para Kubernetes
- **Minikube Dashboard** — observabilidad gráfica del clúster
- **kubectl apply -f** — despliegue declarativo desde manifiestos YAML

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
