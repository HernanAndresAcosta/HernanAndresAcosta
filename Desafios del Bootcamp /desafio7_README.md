# CI/CD Multi-Environment DevOps Pipeline

## 📌 Descripción
Pipeline CI/CD multi-entorno (dev, staging, producción) que automatiza
despliegues de infraestructura usando Jenkins, Ansible y Kubernetes.

## 🏗️ Arquitectura
- Jenkins desplegado en Kubernetes (Minikube)
- GitHub Webhooks expuestos vía Ngrok
- Jenkins Multibranch Pipeline
- Ansible para configuración de servidores
- VMs Linux (Multipass) como targets

(diagrama o imagen acá)

## 🔄 Flujo CI/CD
1. Push a rama `dev` → despliegue en entorno dev
2. Merge a `staging` → despliegue en staging
3. Merge a `main` → despliegue en producción

## ⚙️ Tecnologías
- Jenkins (Multibranch Pipeline)
- Ansible
- Kubernetes (Minikube)
- Git / GitHub
- Ngrok
- Linux / SSH

## 🔐 Seguridad
- Autenticación SSH por clave
- Gestión de credenciales en Jenkins
- Hardening básico de accesos

## 📂 Estructura del Proyecto
(tree resumido)

## 🚀 Cómo ejecutar el proyecto
- Instalación de dependencias
- Configuración de Jenkins
- Ejecución del pipeline

## 🧠 Qué demuestra este proyecto
- CI/CD real multi-entorno
- Automatización y orquestación
- Buenas prácticas DevOps
