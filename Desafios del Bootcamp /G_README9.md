# ⚙️ Desafío N°9 — GitHub Actions: CI/CD automático hacia Docker Hub
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT

---

## 📌 Descripción

Implementación de un pipeline **CI/CD automatizado con GitHub Actions** que construye una imagen Docker de una aplicación Node.js y la publica automáticamente en **Docker Hub** con versionado por etiquetas, en cada `push` a la rama `main`. Soporte multi-plataforma (`linux/amd64` y `linux/arm64`).

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **GitHub Actions** | Orquestación del pipeline CI/CD |
| **Docker Buildx** | Build multi-plataforma |
| **Docker Hub** | Registro de imágenes |
| **Node.js 16** | Runtime de la aplicación |
| **GitHub Secrets** | Credenciales seguras |

---

## 🏗️ Flujo de Trabajo

```
Desarrollador
     ↓  git push → rama main
GitHub (repositorio)
     ↓  trigger automático
GitHub Actions
     ↓  ejecuta docker-build-push.yml
Docker Buildx
     ├── Build linux/amd64
     └── Build linux/arm64
          ↓  push automático
Docker Hub
     ├── hernan1305/nodejs-helloworld-api:1.0.0
     └── hernan1305/nodejs-helloworld-api:latest ✅
```

---

## 🔐 Configuración de Secrets en GitHub

Se configuraron en `Settings → Secrets → Actions`:

| Secret | Descripción |
|--------|-------------|
| `DOCKERHUB_USERNAME` | Usuario de Docker Hub |
| `DOCKERHUB_TOKEN` | Personal Access Token (PAT) con permisos Read/Write/Delete |

> El token se generó en Docker Hub → Account Settings → Personal access tokens.
> **Nunca hardcodear credenciales en el código.**

---

## 🐳 Dockerfile

```dockerfile
FROM node:16

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

---

## ⚙️ Workflow: `.github/workflows/docker-build-push.yml`

```yaml
name: Docker Build and Push

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          platforms: linux/amd64, linux/arm64
          push: true
          tags: |
            hernan1305/nodejs-helloworld-api:1.0.0
            hernan1305/nodejs-helloworld-api:latest
```

---

## ✅ Verificación

### Workflow ejecutado exitosamente
- Status: **Success** — duración: 4m 47s
- Trigger: `push` en rama `main`
- Job `build-and-push` completado ✅

### Imagen publicada en Docker Hub
```
hernan1305/nodejs-helloworld-api
├── Tag: latest  (pushed hace 32 min)
└── Tag: 1.0.0   (pushed hace 32 min)
Repository size: 679 MB
```

### Prueba local de la imagen
```bash
docker build -t hernan1305/nodejs-helloworld-api:1.0.0 .
docker run -p 3000:3000 hernan1305/nodejs-helloworld-api:1.0.0

# Server is listening on port 3000
# http://localhost:3000 → {"message":"Hello, Educacionit!"} ✅
```

---

## 🧠 Conceptos Aplicados

- **GitHub Actions** — pipelines declarativos con triggers por eventos (`push`)
- **Docker Buildx** — build multi-arquitectura (`amd64` + `arm64`) en un solo paso
- **GitHub Secrets** — manejo seguro de credenciales sensibles
- **Versionado de imágenes** — etiquetas `latest` + versión semántica (`1.0.0`)
- **Personal Access Token (PAT)** — autenticación segura sin contraseña en CI/CD
- **docker/login-action**, **docker/build-push-action** — actions oficiales de Docker

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
