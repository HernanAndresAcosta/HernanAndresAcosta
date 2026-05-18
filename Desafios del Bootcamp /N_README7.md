# 🚀 Pipeline CI/CD Automatizado — Jenkins + Ansible + Kubernetes

> **Proyecto:** Desafío N°7 — Bootcamp DevOps Engineer  
> **Autor:** Hernán Andrés Acosta  
> **Stack:** Jenkins · Ansible · Minikube · Docker · Ngrok · GitHub Webhooks

---

## 📋 Descripción

Pipeline CI/CD completo que automatiza el despliegue de infraestructura en tres entornos
(**dev**, **staging** y **producción**) usando ramas de Git como disparador.

Cada push a una rama activa automáticamente un webhook en GitHub, que notifica a Jenkins
(corriendo en Minikube) a través de Ngrok, y Jenkins ejecuta los playbooks de Ansible
correspondientes sobre las máquinas virtuales de destino.

---

## 🏗️ Arquitectura

```
Usuario
   ↓
GitHub (push a rama dev / staging / main)
   ↓  Webhook
Ngrok (túnel seguro hacia Jenkins local)
   ↓
Jenkins en Minikube (Multibranch Pipeline)
   ↓
Ansible (playbooks por entorno)
   ↓
VMs en Multipass
   ├── ansible-dev     → 10.x.x.x  (rama: dev)
   ├── ansible-staging → 10.x.x.x  (rama: staging)
   └── ansible-prod    → 10.x.x.x  (rama: main)
```

---

## 🛠️ Stack Tecnológico

| Herramienta | Uso |
|-------------|-----|
| **Jenkins** | Orquestación del pipeline (Multibranch Pipeline) |
| **Ansible** | Configuración e instalación en VMs (Apache2, sitio web) |
| **Minikube** | Kubernetes local donde corre Jenkins |
| **Multipass** | Máquinas virtuales Ubuntu 24.04 LTS |
| **Ngrok** | Exposición segura de Jenkins local a internet |
| **GitHub Webhooks** | Disparador automático del pipeline en cada push |
| **Docker** | Imagen personalizada de Jenkins con Ansible + Git + SSH |
| **SSH Ed25519** | Autenticación segura entre Jenkins y las VMs |

---

## 📁 Estructura del Proyecto

```
.
├── dev/
│   └── desafio-7/
│       ├── files/index.html
│       ├── includes/install-apache2.yml
│       ├── Jenkinsfile
│       ├── main.yml
│       ├── dev/inventory.ini
│       ├── templates/ansible_site.conf.j2
│       └── vars/vars-site.yml
├── staging/
│   └── desafio-7/
│       ├── files/index.html
│       ├── includes/install-apache2.yml
│       ├── Jenkinsfile
│       ├── main.yml
│       ├── staging/inventory.ini
│       ├── templates/ansible_site.conf.j2
│       └── vars/vars-site.yml
└── main/
    └── desafio-7/
        ├── files/index.html
        ├── includes/install-apache2.yml
        ├── Jenkinsfile
        ├── main.yml
        ├── production/inventory.ini
        ├── templates/ansible_site.conf.j2
        └── vars/vars-site.yml
```

---

## ⚙️ Pipeline Jenkins — Lógica por Entorno

El `Jenkinsfile` detecta la rama activa y selecciona automáticamente el inventario y la IP correspondiente:

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
          } else if (env.BRANCH_NAME == 'staging') {
            env.INVENTORY = 'staging/inventory.ini'
          } else if (env.BRANCH_NAME == 'main') {
            env.INVENTORY = 'production/inventory.ini'
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

## 🔐 Seguridad

- Claves SSH **Ed25519** para conexión entre Jenkins y VMs
- Credenciales gestionadas desde el **Credential Store de Jenkins** (nunca hardcodeadas)
- `ssh-agent` en el pipeline para manejo seguro de claves en tiempo de ejecución
- Permisos estrictos en `~/.ssh` (`700` directorio, `600` authorized_keys)
- Comunicación Jenkins ↔ GitHub a través de **Ngrok con HTTPS**

---

## 🐳 Imagen Docker Personalizada de Jenkins

Se construyó una imagen custom basada en `jenkins/inbound-agent:latest` con Ansible, Git y SSH preinstalados:

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

> 📦 Publicada en Docker Hub: `hernan1305/jenkins-ansible:latest`

---

## 🚦 Flujo de Trabajo

| Rama | Entorno | Resultado |
|------|---------|-----------|
| `dev` | Desarrollo | Deploy en VM ansible-dev |
| `staging` | Testing | Deploy en VM ansible-staging |
| `main` | Producción | Deploy en VM ansible-prod |

Cada push dispara automáticamente:
1. GitHub detecta el push y envía el webhook
2. Ngrok recibe y redirige al Jenkins local
3. Jenkins determina el entorno según la rama
4. Ansible valida conexión (`ping`) y ejecuta el playbook
5. Apache2 queda configurado y el sitio desplegado

---

## 🚀 Cómo Reproducir el Proyecto

### Prerrequisitos
- Ubuntu 24.04 LTS
- Minikube instalado
- Multipass instalado (`sudo snap install multipass`)
- Ngrok cuenta gratuita

### 1. Crear las VMs
```bash
multipass launch --name ansible-dev ubuntu
multipass launch --name ansible-staging ubuntu
multipass launch --name ansible-prod ubuntu
```

### 2. Instalar Jenkins en Minikube
```bash
kubectl create namespace jenkins
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install jenkins -n jenkins jenkins/jenkins
kubectl port-forward svc/jenkins -n jenkins 8081:8080
```

### 3. Exponer Jenkins con Ngrok
```bash
ngrok http 8081
# Copiar la URL pública y configurarla como webhook en GitHub:
# https://[subdominio].ngrok-free.app/github-webhook/
```

### 4. Generar claves SSH y distribuir en VMs
```bash
ssh-keygen -t ed25519 -C "jenkins-ansible"
# Copiar clave pública a cada VM:
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@[IP_VM]
```

### 5. Configurar credenciales en Jenkins
| Nombre | Tipo | Uso |
|--------|------|-----|
| `github-token-hernan` | Secret Text | Acceso al repositorio GitHub |
| `web1-key` | SSH Private Key | Conexión a VMs de Multipass |

### 6. Crear Job Multibranch Pipeline
- Nuevo Item → Multibranch Pipeline
- Branch Sources → Git → URL del repo
- Jenkins detecta automáticamente las ramas con `Jenkinsfile`

---

## 📈 Mejoras Identificadas

- [ ] Implementar **HashiCorp Vault** para gestión de secrets
- [ ] Migrar creación de VMs a **Terraform** (IaC completo)
- [ ] Agregar **health checks** post-despliegue
- [ ] Implementar **rollback automático** ante fallo en producción
- [ ] Monitoreo con **Prometheus + Grafana**

---

## 👤 Autor

**Hernán Andrés Acosta**  
DevOps Engineer en formación | Corrientes, Argentina  
Bootcamp DevOps Engineer — EducaciónIT / Manhattan University (150 hs)  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://linkedin.com/in/TU-PERFIL)
[![GitHub](https://img.shields.io/badge/GitHub-hernan130-black?logo=github)](https://github.com/hernan130)
