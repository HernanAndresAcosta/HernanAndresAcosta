# 🚀 Desafío N°14 — ArgoCD: Despliegue Automatizado con Helm Chart Propio
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT  
**Fecha:** 12/05/2025

---

## 📌 Descripción

Implementación de un **flujo GitOps completo y automatizado** usando ArgoCD para desplegar la aplicación NestJS + MongoDB empaquetada como Helm Chart (Desafío 12). Cualquier `git push` al repositorio se refleja automáticamente en el clúster — sin intervención manual.

> Este desafío integra todo lo aprendido: Docker → Kubernetes → Helm → GitOps con ArgoCD.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **ArgoCD** | Operador GitOps con sync automatizado |
| **Helm Chart** | Empaquetado de la app (Desafío 12) |
| **GitHub** | Fuente de verdad (`hernan130/educacionit-app`) |
| **Minikube** | Clúster Kubernetes local |
| **kubectl** | Verificación de recursos |

---

## 🏗️ Arquitectura GitOps Completa

```
Desarrollador
     ↓  git push (values.yaml o templates)
GitHub: hernan130/educacionit-app
     │  carpeta: educacionit-chart/
     ↓
ArgoCD (namespace: argocd)
     │  syncPolicy: automated
     │  selfHeal: true
     │  prune: true
     ↓  aplica Helm Chart automáticamente
Namespace: educacionit-chart
     ├── Deployment: educacionit-app   (NestJS)
     ├── Service: NodePort :31000
     ├── Deployment: mongodb
     ├── Service: ClusterIP
     └── PVC: persistencia MongoDB
```

---

## 📁 Estructura del Repositorio

```
hernan130/educacionit-app/
├── educacionit-chart/          ← Helm Chart (fuente de verdad)
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── mongo-deployment.yaml
│       └── mongo-service.yaml
└── application.yaml            ← Recurso ArgoCD Application
```

---

## ⚙️ Recurso ArgoCD — `application.yaml`

El manifiesto que registra la aplicación en ArgoCD y activa la sincronización automática:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: educacionit-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/hernan130/educacionit-app.git
    targetRevision: main
    path: educacionit-chart
    helm:
      values: values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: educacionit-chart
  syncPolicy:
    automated:
      prune: true       # Elimina recursos que ya no están en Git
      selfHeal: true    # Revierte cambios manuales no declarados en Git
    syncOptions:
      - CreateNamespace=true   # Crea el namespace si no existe
```

### Parámetros clave explicados

| Parámetro | Valor | Efecto |
|-----------|-------|--------|
| `automated` | habilitado | Sync automático sin intervención manual |
| `prune: true` | true | Elimina recursos borrados del repo |
| `selfHeal: true` | true | Revierte cambios manuales en el clúster |
| `CreateNamespace` | true | Crea `educacionit-chart` si no existe |

---

## 🚀 Proceso de Despliegue

### Instalación de ArgoCD
```bash
minikube start

kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Exponer dashboard
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Obtener contraseña inicial
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

### Registrar la aplicación en ArgoCD
```bash
kubectl apply -f application.yaml
```

ArgoCD detecta automáticamente el Helm Chart en `educacionit-chart/` y despliega todos los recursos.

---

## ✅ Validación del Despliegue

```bash
# Verificar todos los recursos en el namespace
kubectl get all -n educacionit-chart

# Estado esperado:
# ✅ Pod educacionit-app-*     Running
# ✅ Pod mongodb-*             Running
# ✅ Service NodePort :31000   activo
# ✅ Service ClusterIP MongoDB activo
# ✅ PVC MongoDB               Bound
```

**Desde el Dashboard ArgoCD:**
- Estado: ✅ `Synced`
- Salud: 💚 `Healthy`
- Todos los recursos visibles y sin errores

---

## 🖥️ Creación desde la UI de ArgoCD

Como alternativa al `kubectl apply`, se puede crear la app desde la interfaz web:

1. Acceder a `https://localhost:8080` → login con `admin`
2. Click en **NEW APP**
3. Completar el formulario:

| Campo | Valor |
|-------|-------|
| Application Name | `educacionit-app` |
| Project | `default` |
| Sync Policy | ✅ Auto-Sync + Self Heal |
| Repository URL | `https://github.com/hernan130/educacionit-app.git` |
| Revision | `main` |
| Path | `educacionit-chart` |
| Cluster | `https://kubernetes.default.svc` |
| Namespace | `educacionit-chart` |

4. Click en **Create** → ArgoCD despliega automáticamente

---

## 🔄 Ciclo GitOps en Acción

```bash
# 1. Modificar values.yaml (ej: cambiar replicaCount a 2)
# 2. Commit y push
git add educacionit-chart/values.yaml
git commit -m "Scale app to 2 replicas"
git push origin main

# 3. ArgoCD detecta el cambio automáticamente
# 4. Aplica la nueva configuración al clúster sin intervención manual
# 5. Dashboard muestra: Synced ✅ Healthy ✅
```

---

## 🧹 Comandos de Mantenimiento

```bash
# Verificar instalación manual previa al GitOps
helm install educacionit educacionit-chart \
  --namespace educacionit-chart --create-namespace

# Eliminar release para pruebas limpias
helm uninstall educacionit -n educacionit-chart

# Eliminar aplicación ArgoCD y recrear
kubectl delete application educacionit-app -n argocd
# → Luego recrear desde UI o kubectl apply -f application.yaml
```

---

## 🧠 Conceptos Aplicados

- **GitOps con ArgoCD** — Git como fuente de verdad, sync continuo con el clúster
- **`syncPolicy.automated`** — eliminación de intervención manual en despliegues
- **`selfHeal`** — reconciliación automática ante cambios manuales no deseados
- **`prune`** — limpieza automática de recursos eliminados del repositorio
- **`CreateNamespace`** — infraestructura autodescriptiva sin pasos previos manuales
- **Helm + ArgoCD** — combinación de empaquetado y GitOps para máxima reutilización
- **Application CRD** — ArgoCD extiende Kubernetes con su propio tipo de recurso

---

## 📈 Progresión Completa del Bootcamp

| Desafío | Tecnología | Concepto central |
|---------|------------|-----------------|
| N°1-3 | Jenkins + Bash | Automatización básica |
| N°4-5 | AWS IAM + VPC | Cloud fundamentals |
| N°6 | Ansible | Configuration management |
| N°7 | Jenkins + Ansible + K8s | CI/CD multi-ambiente |
| N°8 | Terraform + GH Actions | IaC automatizado |
| N°9 | GitHub Actions + Docker | CI hacia Docker Hub |
| N°10 | Docker Compose | Entorno local reproducible |
| N°11 | Kubernetes | Orquestación de contenedores |
| N°12 | Helm | Empaquetado parametrizado |
| N°13 | ArgoCD básico | GitOps introducción |
| **N°14** | **ArgoCD + Helm propio** | **GitOps production-ready** |

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
