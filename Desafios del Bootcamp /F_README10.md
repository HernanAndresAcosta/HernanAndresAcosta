# 🐳 Desafío N°10 — Docker Multi-Stage + Docker Compose: NestJS + MongoDB
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT

---

## 📌 Descripción

Contenerización completa de una aplicación **NestJS con MongoDB** usando **Docker multi-stage build** para optimizar la imagen de producción, y **Docker Compose** para levantar el entorno completo de desarrollo local con un solo comando.

> Este desafío es la base del Desafío N°11 (migración a Kubernetes).

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **NestJS** | Framework Node.js para la aplicación |
| **MongoDB** | Base de datos NoSQL |
| **Docker** | Contenerización con multi-stage build |
| **Docker Compose** | Orquestación local de servicios |
| **Node.js 20 Alpine** | Imagen base ligera |

---

## 🏗️ Arquitectura

```
docker-compose up --build
        ↓
┌─────────────────────────────────────┐
│         Docker Network               │
│                                     │
│  ┌──────────────┐  ┌─────────────┐  │
│  │  app         │  │   mongo     │  │
│  │  NestJS      │→ │  MongoDB    │  │
│  │  :3000       │  │  :27017     │  │
│  └──────────────┘  └─────────────┘  │
│    host:3000          host:27018    │
└─────────────────────────────────────┘
         ↓
  http://localhost:3000 → "Hello World!" ✅
```

---

## 🐳 Dockerfile — Multi-Stage Build

```dockerfile
# ── Stage 1: Builder ──────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app

# Instalar dependencias primero (aprovecha cache de Docker)
COPY package*.json ./
RUN npm install

# Copiar código y compilar TypeScript → JavaScript
COPY . .
RUN npm run build

# ── Stage 2: Producción ───────────────────────────────
FROM node:20-alpine

WORKDIR /app

# Solo copiar lo necesario del builder (imagen más liviana)
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package*.json ./

EXPOSE 3000
CMD ["node", "dist/main"]
```

**Ventaja del multi-stage:** la imagen final no incluye el compilador TypeScript ni las devDependencies, resultando en una imagen significativamente más pequeña y segura para producción.

---

## 🐙 docker-compose.yaml

```yaml
version: "3.8"
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    depends_on:
      - mongo
    environment:
      - PORT=3000
      - MONGO_DB_URI=mongodb://mongo:27017
      - MONGO_DB_NAME=<db_name_here>
      - MONGO_DB_USER=<mongo_user_here>
      - MONGO_DB_PASS=<mongo_pass_here>
    restart: always

  mongo:
    image: mongo:latest
    ports:
      - "27018:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: <mongo_user_here>
      MONGO_INITDB_ROOT_PASSWORD: <mongo_pass_here>
    volumes:
      - mongo_data:/data/db
    restart: always

volumes:
  mongo_data:
```

---

## 📁 Estructura del Proyecto

```
nestjs-docker/
├── src/
│   ├── app.controller.ts    ← Controlador principal
│   ├── app.service.ts       ← Servicio principal
│   ├── app.module.ts        ← Módulo principal
│   └── main.ts              ← Punto de entrada
├── test/                    ← Pruebas unitarias e integración
├── Dockerfile               ← Multi-stage build
├── docker-compose.yaml      ← Orquestación local
└── package.json
```

---

## 🚀 Comandos de Uso

### Levantar el entorno completo
```bash
docker-compose up --build
```

### Verificar contenedores activos
```bash
docker ps
# CONTAINER ID   IMAGE              PORTS
# bd8484b1fb5c   nestjs-docker-app  0.0.0.0:3000->3000/tcp
# 282fad4c5478   mongo:latest       0.0.0.0:27018->27017/tcp
```

### Acceder a la aplicación
```bash
curl -X GET http://localhost:3000/
# "Hello World!" ✅
```

### Verificar conexión a MongoDB
```bash
mongosh "mongodb://localhost:27018"
# Current Mongosh Log ID: 67cd0154038a5f0a0f6b140a
# Using MongoDB: 8.0.5 ✅
```

### Detener el entorno
```bash
docker-compose down        # detiene y elimina contenedores
docker-compose stop        # solo detiene (conserva contenedores)
```

---

## ✅ Resultado

- Aplicación NestJS corriendo en `http://localhost:3000` → responde `Hello World!`
- MongoDB activo y conectado automáticamente via Docker network interna
- Entorno **100% reproducible** en cualquier máquina con Docker instalado

---

## 🧠 Conceptos Aplicados

- **Docker multi-stage build** — separación builder/producción para imagen optimizada
- **Docker Compose** — orquestación multi-contenedor con dependencias (`depends_on`)
- **Docker volumes** — persistencia de datos MongoDB entre reinicios
- **Docker networking** — comunicación entre contenedores por nombre de servicio
- **Variables de entorno** — configuración externalizada del contenedor
- **restart: always** — resiliencia ante fallos del contenedor

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
