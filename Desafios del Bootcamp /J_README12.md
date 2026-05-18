# ⎈ Desafío N°12 — Helm Chart Personalizado: NestJS + MongoDB en Kubernetes
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT

---

## 📌 Descripción

Desarrollo de un **Helm Chart personalizado** para desplegar la aplicación NestJS + MongoDB (construida en los Desafíos 10 y 11) en Kubernetes/Minikube, aplicando el principio **DRY** (Don't Repeat Yourself) mediante templates reutilizables y configuración centralizada en `values.yaml`.

> Helm actúa como el "gestor de paquetes" de Kubernetes — similar a `apt-get` para Linux o `npm` para Node.js.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **Helm 3** | Gestor de paquetes para Kubernetes |
| **Minikube** | Clúster Kubernetes local |
| **kubectl** | CLI de Kubernetes |
| **NestJS** | Aplicación (`educacionit-app:1.0.0`) |
| **MongoDB** | Base de datos contenerizada |
| **Kubernetes Secrets** | Gestión segura de credenciales |

---

## 🏗️ Arquitectura Implementada

```
Namespace: educacionit-chart
────────────────────────────────────────────────
  Helm Release: educacionit-app
  │
  ├── Deployment: educacionit-app-educacionit-app
  │   └── Container: NestJS :3000
  │       └── Env: MONGO_URI (via Secret)
  │
  ├── Service: educacionit-app-educacionit-app
  │   └── Type: NodePort → :31000
  │
  ├── Deployment: educacionit-app-educacionit-app-mongodb
  │   └── Container: mongo:latest :27017
  │       └── Env: MONGO_INITDB_ROOT_* (via Secret)
  │
  ├── Service: educacionit-app-educacionit-app-mongodb
  │   └── Type: ClusterIP (interno)
  │
  └── Secret: educacionit-app-educacionit-app-mongodb-secret
      ├── username
      └── password
```

---

## 📁 Estructura del Chart

```
educacionit-app-chart/
├── Chart.yaml                  ← Metadata del chart
├── values.yaml                 ← Configuración centralizada
└── templates/
    ├── _helpers.tpl            ← Funciones reutilizables (labels, nombres)
    ├── deployment.yaml         ← Deployment de la app NestJS
    ├── service.yaml            ← Service NodePort de la app
    ├── mongo-deployment.yaml   ← Deployment de MongoDB
    └── mongo-service.yaml      ← Service ClusterIP de MongoDB
```

---

## ⚙️ Archivos Clave del Chart

### `Chart.yaml`
```yaml
apiVersion: v2
name: educacionit-app
description: Aplicación NestJS en Minikube
version: 0.1.0
appVersion: "1.0.0"
```

### `values.yaml` — Configuración centralizada
```yaml
replicaCount: 1

image:
  repository: educacionit-app
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 3000
  targetPort: 3000
  nodePort: 31000

resources:
  limits:
    cpu: "500m"
    memory: "256Mi"
  requests:
    cpu: "100m"
    memory: "128Mi"

namespace: educacionit-chart

mongodb:
  enabled: true
  image: mongo:latest
  port: 27017
  auth:
    username: admin
    password: <usar-secret-seguro>   # Nunca hardcodear en producción
    database: educacionit-db
```

### `templates/_helpers.tpl` — Funciones reutilizables
```yaml
{{/* Nombre del chart */}}
{{- define "educacionit-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/* Nombre completo del release */}}
{{- define "educacionit-app.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/* Labels comunes */}}
{{- define "educacionit-app.labels" -}}
helm.sh/chart: {{ include "educacionit-app.name" . }}
{{ include "educacionit-app.selectorLabels" . }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/* Selector labels */}}
{{- define "educacionit-app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "educacionit-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

### `templates/deployment.yaml` — App NestJS
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "educacionit-app.fullname" . }}
  namespace: {{ .Values.namespace }}
  labels:
    {{- include "educacionit-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "educacionit-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "educacionit-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.targetPort }}
          env:
            - name: MONGO_URI
              value: "mongodb://admin:$(PASSWORD)@{{ include "educacionit-app.fullname" . }}-mongodb:27017/educacionit-db?authSource=admin"
            - name: PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "educacionit-app.fullname" . }}-mongodb-secret
                  key: password
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

### `templates/mongo-deployment.yaml` — MongoDB
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "educacionit-app.fullname" . }}-mongodb
  namespace: {{ .Values.namespace }}
  labels:
    {{- include "educacionit-app.labels" . | nindent 4 }}
    app.kubernetes.io/component: database
spec:
  replicas: 1
  selector:
    matchLabels:
      {{- include "educacionit-app.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: database
  template:
    spec:
      containers:
        - name: mongodb
          image: {{ .Values.mongodb.image }}
          ports:
            - containerPort: {{ .Values.mongodb.port }}
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: {{ include "educacionit-app.fullname" . }}-mongodb-secret
                  key: username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "educacionit-app.fullname" . }}-mongodb-secret
                  key: password
```

---

## 🚀 Proceso de Despliegue

### 1. Preparar el entorno
```bash
minikube start --driver=docker
eval $(minikube docker-env)   # Conecta terminal al daemon de Docker de Minikube
docker build -t educacionit-app:1.0.0 .   # Construye imagen en Minikube
```

### 2. Crear namespace dedicado
```bash
kubectl create namespace educacionit-chart
```

### 3. Instalar el chart
```bash
helm install educacionit-app ./educacionit-app-chart -n educacionit-chart
```

### 4. Verificar despliegue
```bash
# Pods activos
kubectl get pods -n educacionit-chart

# Servicios
kubectl get svc -n educacionit-chart

# Logs de MongoDB
kubectl logs -n educacionit-chart -l app.kubernetes.io/component=database

# Eventos del namespace
kubectl get events -n educacionit-chart --sort-by='.metadata.creationTimestamp'
```

### 5. Verificar conexión MongoDB
```bash
kubectl exec -n educacionit-chart \
  educacionit-app-educacionit-app-mongodb-7444d6dbbd-qr9hc -- \
  mongosh --eval 'db.runCommand({ping:1})'
```

### 6. Acceder a la aplicación
```bash
minikube service educacionit-app-educacionit-app -n educacionit-chart --url
# → http://192.168.49.2:31000
```

---

## 🔄 Gestión del Release con Helm

```bash
# Ver releases instalados
helm list -n educacionit-chart

# Actualizar configuración sin reinstalar
helm upgrade educacionit-app ./educacionit-app-chart -n educacionit-chart

# Ver historial de versiones
helm history educacionit-app -n educacionit-chart

# Rollback a versión anterior
helm rollback educacionit-app 1 -n educacionit-chart

# Desinstalar completamente
helm uninstall educacionit-app -n educacionit-chart
```

---

## 🧠 Conceptos Aplicados

- **Helm Chart** — paquete reutilizable de recursos Kubernetes
- **Templates con Go templating** — YAML dinámico con variables (`{{ .Values.* }}`)
- **`_helpers.tpl`** — funciones reutilizables para labels y nombres consistentes
- **`values.yaml`** — única fuente de verdad para toda la configuración
- **Principio DRY** — eliminación de YAML repetitivo via templates
- **Kubernetes Secrets** — credenciales MongoDB nunca hardcodeadas en templates
- **Namespace dedicado** — aislamiento de recursos por aplicación
- **`helm upgrade/rollback`** — gestión de versiones de la configuración
- **`eval $(minikube docker-env)`** — build de imagen directo en el daemon de Minikube

---

## 📈 Evolución del Proyecto

| Desafío | Tecnología | Avance |
|---------|------------|--------|
| N°10 | Docker Compose | Entorno local con un comando |
| N°11 | Kubernetes (manifiestos YAML) | Despliegue orquestado en Minikube |
| **N°12** | **Helm Chart** | **Configuración parametrizada y versionada** |

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
