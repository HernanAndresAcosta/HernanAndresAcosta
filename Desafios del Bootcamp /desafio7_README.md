# 🚀 Desafío N°7 — Pipeline CI/CD Multi-Ambiente
### Trabajo Integrador Final — Bootcamp DevOps Engineer
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Fecha:** Mayo 2025  
**Institución:** EducaciónIT / Manhattan University

---

## 📌 Descripción

Este proyecto es el **trabajo integrador final** del Bootcamp DevOps Engineer de 150 horas.
Integra **todas las tecnologías vistas durante la formación** en un pipeline CI/CD funcional
que despliega infraestructura automáticamente en tres entornos separados.

Cada `git push` a una rama dispara automáticamente el pipeline completo:
configuración de servidores, validación de conectividad y despliegue del sitio web.

---

## 🏗️ Arquitectura del Sistema

```
Usuario
   ↓  git push
GitHub (rama: dev / staging / main)
   ↓  Webhook HTTP POST
Ngrok (túnel seguro — expone Jenkins local a internet)
   ↓
Jenkins en Minikube (Multibranch Pipeline)
   ↓  detecta rama → selecciona inventario
Ansible (playbooks + plantillas Jinja2)
   ↓  SSH Ed25519
VMs en Multipass (Ubuntu 24.04 LTS)
   ├── ansible-dev      → entorno desarrollo
   ├── ansible-staging  → entorno testing
   └── ansible-prod     → entorno producción
```

---

## 🛠️ Stack Tecnológico Utilizado

| Herramienta | Versión / Detalle | Rol en el proyecto |
|-------------|-------------------|--------------------|
| **Jenkins** | LTS en Minikube | Orquestación del pipeline CI/CD |
| **Ansible** | + Jinja2 templates | Configuración e instalación en VMs |
| **Kubernetes** | Minikube local | Plataforma donde corre Jenkins |
| **Docker** | Imagen personalizada | Jenkins con Ansible + Git + SSH preinstalados |
| **Multipass** | Ubuntu 24.04 LTS | Máquinas virtuales objetivo |
| **Ngrok** | Free tier | Exposición segura de Jenkins a internet |
| **GitHub Webhooks** | Push events | Disparador automático del pipeline |
| **SSH Ed25519** | Hardened | Autenticación segura Jenkins ↔ VMs |
| **Git / GitFlow** | Multibranch | Control de versiones y separación de entornos |

---

## 📁 Estructura del Repositorio

```
desafio-7/
├── dev/
│   └── desafio-7/
│       ├── Jenkinsfile               ← Pipeline para entorno dev
│       ├── main.yml                  ← Playbook principal Ansible
│       ├── dev/inventory.ini         ← Hosts del entorno desarrollo
│       ├── files/index.html          ← Sitio web desplegado
│       ├── includes/install-apache2.yml
│       ├── templates/ansible_site.conf.j2
│       └── vars/vars-site.yml
├── staging/
│   └── desafio-7/
│       ├── Jenkinsfile               ← Pipeline para entorno staging
│       ├── main.yml
│       ├── staging/inventory.ini     ← Hosts del entorno testing
│       ├── files/index.html
│       ├── includes/install-apache2.yml
│       ├── templates/ansible_site.conf.j2
│       └── vars/vars-site.yml
└── main/
    └── desafio-7/
        ├── Jenkinsfile               ← Pipeline para entorno producción
        ├── main.yml
        ├── production/inventory.ini  ← Hosts del entorno producción
        ├── files/index.html
        ├── includes/install-apache2.yml
        ├── templates/ansible_site.conf.j2
        └── vars/vars-site.yml
```

---

## ⚙️ Pipeline Jenkins — Lógica Central

El `Jenkinsfile` detecta la rama activa y selecciona automáticamente
el inventario y la IP de destino:

```groovy
pipeline {
  agent any
  environment {
    ANSIBLE_FORCE_COLOR = 'true'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Determine Environment') {
      steps {
        script {
          if (env.BRANCH_NAME == 'dev') {
            env.INVENTORY = 'dev/inventory.ini'
            env.TARGET_IP = '10.x.x.x'
          } else if (env.BRANCH_NAME == 'staging') {
            env.INVENTORY = 'staging/inventory.ini'
            env.TARGET_IP = '10.x.x.x'
          } else if (env.BRANCH_NAME == 'main') {
            env.INVENTORY = 'production/inventory.ini'
            env.TARGET_IP = '10.x.x.x'
          } else {
            error "Rama no válida: ${env.BRANCH_NAME}"
          }
        }
      }
    }
    stage('Run Ansible Playbook') {
      steps {
        sshagent(credentials: ['web1-key']) {
          sh '''
            ssh-keyscan -H ${TARGET_IP} >> ~/.ssh/known_hosts
            ansible -i ${INVENTORY} all -m ping
            ansible-playbook -i ${INVENTORY} main.yml
          '''
        }
      }
    }
  }
}
```

---

## 🔀 Flujo Git → Entorno

| Rama | Entorno | Inventario | Resultado |
|------|---------|------------|-----------|
| `dev` | Desarrollo | `dev/inventory.ini` | Deploy en ansible-dev |
| `staging` | Testing | `staging/inventory.ini` | Deploy en ansible-staging |
| `main` | Producción | `production/inventory.ini` | Deploy en ansible-prod |

---

## 🐳 Imagen Docker Personalizada de Jenkins

Se construyó una imagen custom con las dependencias necesarias para
que Jenkins pueda ejecutar Ansible directamente:

```dockerfile
FROM jenkins/inbound-agent:latest
USER root
RUN apt-get update && apt-get install -y \
    ansible \
    git \
    openssh-client \
    && mkdir -p /home/jenkins/.ssh \
    && chown -R jenkins:jenkins /home/jenkins/.ssh \
    && apt-get clean
USER jenkins
```

> 📦 Publicada en Docker Hub: [`hernan1305/jenkins-ansible:latest`](https://hub.docker.com/r/hernan1305/jenkins-ansible)

---

## 🔐 Configuración de Seguridad

- **Claves SSH Ed25519** — mayor seguridad que RSA para conexión Jenkins ↔ VMs
- **Jenkins Credential Store** — credenciales nunca hardcodeadas en el código
- **ssh-agent en pipeline** — manejo seguro de claves en tiempo de ejecución
- **Permisos estrictos** — `chmod 700 ~/.ssh` y `chmod 600 ~/.ssh/authorized_keys`
- **Ngrok con HTTPS** — comunicación cifrada GitHub ↔ Jenkins

### Credenciales configuradas en Jenkins

| Nombre | Tipo | Uso |
|--------|------|-----|
| `github-token-hernan` | Secret Text | Acceso al repositorio GitHub |
| `web1-key` | SSH Private Key | Conexión a VMs de Multipass |

---

## 🚀 Cómo Reproducir el Proyecto

### Prerrequisitos
- Ubuntu 24.04 LTS
- Minikube + kubectl + Helm instalados
- Multipass (`sudo snap install multipass`)
- Cuenta gratuita en Ngrok

### Paso 1 — Crear las VMs
```bash
multipass launch --name ansible-dev ubuntu
multipass launch --name ansible-staging ubuntu
multipass launch --name ansible-prod ubuntu
multipass list  # verificar que estén Running
```

### Paso 2 — Instalar Jenkins en Minikube
```bash
kubectl create namespace jenkins
helm repo add jenkins https://charts.jenkins.io && helm repo update
helm install jenkins -n jenkins jenkins/jenkins
kubectl port-forward svc/jenkins -n jenkins 8081:8080
# Obtener password inicial:
kubectl exec -it -n jenkins jenkins-0 -- cat /var/jenkins_home/secrets/initialAdminPassword
```

### Paso 3 — Configurar imagen personalizada
Usar la imagen `hernan1305/jenkins-ansible:latest` en el Pod Template
de Jenkins para que tenga Ansible disponible en los agentes.

### Paso 4 — Generar y distribuir claves SSH
```bash
ssh-keygen -t ed25519 -C "jenkins-ansible"
# Copiar clave pública a cada VM:
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@[IP_VM_DEV]
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@[IP_VM_STAGING]
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@[IP_VM_PROD]
```

### Paso 5 — Exponer Jenkins con Ngrok
```bash
# En terminal 1:
kubectl port-forward svc/jenkins -n jenkins 8081:8080
# En terminal 2:
ngrok http 8081
# Copiar URL pública → configurar como webhook en GitHub:
# https://[subdominio].ngrok-free.app/github-webhook/
```

### Paso 6 — Crear Job Multibranch Pipeline en Jenkins
1. Nuevo Item → **Multibranch Pipeline** → nombre: `desafio-7`
2. Branch Sources → Git → URL: `https://github.com/hernan130/desafio-7.git`
3. Agregar credencial `github-token-hernan`
4. Guardar → Jenkins escanea y crea jobs por cada rama con `Jenkinsfile`

---

## 📈 Mejoras Identificadas (roadmap personal)

- [ ] Migrar creación de VMs a **Terraform** (IaC completo)
- [ ] Implementar **HashiCorp Vault** para gestión de secrets
- [ ] Agregar **health checks** post-despliegue en el pipeline
- [ ] Implementar **rollback automático** ante fallo en producción
- [ ] Monitoreo con **Prometheus + Grafana**
- [ ] Reemplazar Multipass por instancias **EC2 en AWS**

---

## 🧠 Tecnologías aprendidas en este proyecto

Este desafío integra los contenidos de todo el bootcamp:

| Módulo del Bootcamp | Aplicado en |
|---------------------|-------------|
| Linux y Bash | VMs Ubuntu, scripting en pipeline |
| Redes y SSH | Inventarios Ansible, autenticación Ed25519 |
| Git y GitFlow | Multibranch, separación de entornos por rama |
| Docker | Imagen personalizada Jenkins |
| Kubernetes | Jenkins corriendo en Minikube |
| Ansible | Playbooks, inventarios, templates Jinja2 |
| Jenkins / CI-CD | Multibranch Pipeline, Webhooks |
| Scrum | Planificación y entrega iterativa del proyecto |

---

## 👤 Autor

**Hernán Andrés Acosta**  
DevOps Engineer en formación | Corrientes, Argentina

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
