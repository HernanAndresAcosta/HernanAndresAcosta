# CI/CD Multi-Environment DevOps Pipeline

## 📌 Descripción
Este proyecto implementa un **pipeline CI/CD multi-entorno** que automatiza
el despliegue de una aplicación web y la configuración de servidores utilizando
**Jenkins, Ansible y Kubernetes**.

El flujo simula un escenario real donde los cambios pasan por los entornos
**dev → staging → producción**, según la rama de Git utilizada.

---

## 🏗️ Arquitectura del Proyecto
- Jenkins ejecutándose en **Kubernetes (Minikube)**
- Repositorio GitHub conectado mediante **Webhooks (Ngrok)**
- Pipeline **Multibranch** en Jenkins
- **Ansible** para la configuración y despliegue
- **VMs Linux (Multipass)** como servidores destino

*(Acá se puede agregar un diagrama de arquitectura)*

---

## 🔄 Flujo CI/CD
1. Push a la rama `dev`  
   → Despliegue automático en entorno **desarrollo**

2. Merge a la rama `staging`  
   → Despliegue automático en entorno **pre-producción**

3. Merge a la rama `main`  
   → Despliegue en **producción**

Cada entorno se ejecuta de forma independiente.

---

## ⚙️ Tecnologías Utilizadas
- Jenkins (Multibranch Pipeline)
- Ansible
- Kubernetes (Minikube)
- Git / GitHub
- Ngrok
- Linux / SSH

---

## 📂 Estructura de Directorios

```text
.
├── dev/
│   └── desafio-7/
│       ├── files/
│       │   └── index.html
│       ├── includes/
│       │   └── install-apache2.yml
│       ├── templates/
│       │   └── ansible_site.conf.j2
│       ├── vars/
│       │   └── vars-site.yml
│       └── main.yml
│
├── staging/
│   └── desafio-7/
│       ├── files/
│       │   └── index.html
│       ├── includes/
│       │   └── install-apache2.yml
│       ├── templates/
│       │   └── ansible_site.conf.j2
│       ├── vars/
│       │   └── vars-site.yml
│       ├── inventory.ini
│       └── READMEstaging.md
│
├── main/
│   └── desafio-7/
│       ├── files/
│       │   └── index.html
│       ├── includes/
│       │   └── install-apache2.yml
│       ├── templates/
│       │   └── ansible_site.conf.j2
│       ├── vars/
│       │   └── vars-site.yml
│       ├── inventory.ini
│       ├── Jenkinsfile
│       └── main.yml
│
└── README.md

## 🧠 ¿Por qué separé los entornos físicamente?

### Motivos

- **Aislar configuraciones**  
  Cada entorno tiene sus propios archivos de variables, templates e inventarios.

- **Evitar errores**  
  Trabajar en un entorno no afecta a los otros, reduciendo riesgos en producción.

- **Claridad**  
  Es fácil identificar qué archivos pertenecen a cada entorno.

- **Simular un flujo real**  
  Los cambios se validan primero en desarrollo, luego en staging y finalmente en producción.
