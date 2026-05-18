# 🔗 Desafío N°2 — Integración GitHub Webhook + Jenkins
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT

---

## 📌 Descripción

Configuración de un **webhook en GitHub** para disparar automáticamente un pipeline en Jenkins cada vez que se realiza un `push` al repositorio. Se utilizó **Ngrok** para exponer el Jenkins local a internet y recibir los eventos de GitHub.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **Jenkins** | Servidor CI que recibe y procesa el webhook |
| **GitHub Webhooks** | Disparador automático en cada `push` |
| **Ngrok** | Túnel seguro para exponer Jenkins local a internet |
| **Plugin GitHub (Jenkins)** | Build trigger: `GitHub hook trigger for GITScm polling` |
| **Node.js** | Runtime configurado en Jenkins (plugin NodeJS) |

---

## 🏗️ Flujo de Integración

```
Dev → git push
         ↓
GitHub detecta el push → envía HTTP POST al webhook
         ↓
Ngrok recibe la solicitud → redirige a Jenkins local
         ↓
Jenkins (plugin GitHub) → dispara el pipeline automáticamente
         ↓
Pipeline ejecutado: Finished: SUCCESS ✅
```

---

## ⚙️ Configuración Paso a Paso

### 1. Exponer Jenkins con Ngrok
```bash
# Exponer Jenkins local al puerto 8080
ngrok http 8080

# URL pública generada (ejemplo):
# https://084c-190-7-36-128.ngrok-free.app
```

### 2. Configurar Webhook en GitHub
- Repositorio → **Settings** → **Webhooks** → **Add webhook**
- **Payload URL:** `https://[subdominio].ngrok-free.app/github-webhook/`
- **Content type:** `application/x-www-form-urlencoded`
- **SSL verification:** habilitada
- **Events:** `Just the push event`
- **Active:** ✅

### 3. Configurar Build Trigger en Jenkins
En la configuración del Job:
- **Build Triggers** → ✅ `GitHub hook trigger for GITScm polling`

### 4. Configurar Pipeline desde SCM
```
Definition: Pipeline script from SCM
SCM: Git
Repository URL: https://github.com/hernan130/nodejs-helloworld-api.git
Credentials: (none — repo público)
```

### 5. Habilitar Node.js en Jenkins
- **Manage Jenkins** → **Tools** → **NodeJS installations**
- Nombre: `nodejs21` — Versión: `NodeJS 21.7.3`
- ✅ Instalar automáticamente

---

## ✅ Verificación

El webhook funcionó correctamente — respuesta **HTTP 200** confirmada en GitHub:

```
Request URL: https://084c-190-7-36-128.ngrok-free.app/github-webhook/
Request method: POST
X-GitHub-Event: push
Response: 200 OK ✅
Last delivery was successful.
```

Al hacer `git commit` en el README y push, Jenkins disparó el pipeline automáticamente sin intervención manual — `Finished: SUCCESS`.

---

## 🧠 Conceptos Aplicados

- Configuración de **webhooks HTTP** en GitHub
- Exposición segura de servicios locales con **Ngrok**
- **Build triggers** automáticos en Jenkins via plugin GitHub
- Diferencia entre URL estática (`$JENKINS_BASE_URL/github-webhook/`) y URL dinámica (Ngrok)
- Configuración de credenciales para repos públicos vs privados en Jenkins

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
