# 🔄 Desafío N°13 — ArgoCD: GitOps en Minikube
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT  
**Fecha:** 07/05/2025

---

## 📌 Descripción

Implementación de **ArgoCD** en un entorno local Kubernetes con Minikube, estableciendo un flujo de trabajo **GitOps completo**: el repositorio Git actúa como única fuente de verdad, y ArgoCD sincroniza automáticamente el estado del clúster con los manifiestos declarados en el repo.

> **Principio GitOps:** *"La verdad está en el repositorio"* — cualquier cambio en la infraestructura pasa primero por Git.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **ArgoCD** | Operador GitOps — sincroniza Git ↔ Kubernetes |
| **Minikube** | Clúster Kubernetes local |
| **GitHub** | Repositorio de manifiestos (`hernan130/argocd-apps`) |
| **Helm** | Empaquetado para la aplicación `helm-app` |
| **kubectl / argocd CLI** | Gestión y verificación |

---

## 🏗️ Arquitectura GitOps

```
Desarrollador
     ↓  git push (manifiestos YAML o Helm chart)
GitHub (hernan130/argocd-apps)
     ↓  ArgoCD detecta cambios (polling/webhook)
ArgoCD (namespace: argocd)
     ↓  sincroniza estado deseado
Minikube Cluster
     ├── Namespace: default
     │   ├── Deployment: guestbook-ui    (manifiestos YAML)
     │   └── Service: guestbook-ui
     └── Namespace: default
         └── helm-app                   (Helm Chart)
```

---

## 🚀 Instalación y Configuración

### 1. Crear namespace y desplegar ArgoCD
```bash
kubectl create namespace argocd

kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Acceder al Dashboard
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# → https://localhost:8080
```

### 3. Obtener contraseña inicial
```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 --decode
```

### 4. Login via CLI
```bash
argocd login localhost:8080 --username admin --password <contraseña-obtenida>
```

---

## 📦 Aplicaciones Desplegadas

### App 1 — `guestbook` (manifiestos YAML nativos)

**Creación via CLI:**
```bash
argocd app create guestbook \
  --repo https://github.com/hernan130/argocd-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

**Sincronización:**
```bash
argocd app sync guestbook
```

**Estado:** ✅ `Synced` | ✅ `Healthy`

#### Manifiestos del repositorio

`deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook-ui
spec:
  replicas: 1
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: guestbook-ui
  template:
    metadata:
      labels:
        app: guestbook-ui
    spec:
      containers:
        - image: gcr.io/heptio-images/ks-guestbook-demo:0.2
          name: guestbook-ui
          ports:
            - containerPort: 80
```

`svc.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: guestbook-ui
spec:
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: guestbook-ui
```

---

### App 2 — `helm-app` (Helm Chart)

Configurada desde la **UI de ArgoCD**, demostrando la flexibilidad de ArgoCD para trabajar tanto con YAML nativo como con Helm.

`Chart.yaml`:
```yaml
apiVersion: v2
name: helm-guestbook
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0"
```

`values.yaml` (extracto):
```yaml
replicaCount: 1
image:
  repository: gcr.io/heptio-images/ks-guestbook-demo
  tag: 0.1
  pullPolicy: IfNotPresent
containerPort: 80
service:
  type: ClusterIP
  port: 80
```

---

## ✅ Verificación del Despliegue

```bash
# Ver aplicaciones en ArgoCD
kubectl get applications -n argocd

# Ver pods desplegados
kubectl get pods -n default

# Ver estado desde CLI de ArgoCD
argocd app list
argocd app get guestbook
```

---

## 📁 Estructura del Repositorio Git

```
hernan130/argocd-apps/
├── guestbook/                  ← App con manifiestos YAML nativos
│   ├── deployment.yaml
│   └── svc.yaml
└── helm-guestbook/             ← App con Helm Chart
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
```

---

## 🧠 Conceptos Aplicados

- **GitOps** — Git como única fuente de verdad para el estado del clúster
- **ArgoCD** — operador de sincronización continua Git ↔ Kubernetes
- **Sync automático** — ArgoCD detecta drift entre el repo y el clúster y reconcilia
- **Estado Synced/Healthy** — verificación dual: ¿está aplicado? ¿funciona?
- **Multi-metodología** — despliegue con YAML nativo y con Helm desde el mismo operador
- **`revisionHistoryLimit`** — control del historial de rollbacks disponibles
- **Namespace dedicado** — ArgoCD aislado en `argocd`, apps en `default`
- **Port-forward** — acceso local al dashboard sin exposición externa
- **Secrets iniciales** — recuperación segura de credenciales via `kubectl get secret`

---

## 💡 Lecciones Aprendidas

- El **port-forward debe mantenerse activo** mientras se usa el dashboard o la CLI
- La contraseña inicial de ArgoCD se almacena en un Secret y debe guardarse tras el primer login
- Los **namespaces dedicados** para herramientas de administración evitan conflictos con las apps
- ArgoCD detecta automáticamente cambios en el repositorio — cualquier `git push` puede desencadenar una sincronización

---

## 📈 Evolución GitOps en el Bootcamp

| Desafío | Herramienta | Gestión de despliegues |
|---------|-------------|----------------------|
| N°7 | Jenkins + Ansible | Pipeline CI/CD con ramas |
| N°8 | Terraform + GitHub Actions | IaC automatizado |
| N°11 | kubectl | Manifiestos manuales |
| N°12 | Helm | Configuración parametrizada |
| **N°13** | **ArgoCD** | **GitOps — Git como fuente de verdad** |

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
