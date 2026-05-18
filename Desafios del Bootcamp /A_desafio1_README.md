# 🔧 Desafío N°1 — Gestión de Usuarios Linux con Jenkins Pipeline Declarativo
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT

---

## 📌 Descripción

Pipeline declarativo en Jenkins que automatiza la **creación y eliminación de usuarios** en un sistema Linux, integrando scripts Bash almacenados en GitHub y ejecutándolos remotamente desde Jenkins.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **Jenkins** | Orquestación del pipeline declarativo |
| **Bash** | Scripts de creación y eliminación de usuarios |
| **GitHub** | Repositorio de scripts y Jenkinsfile |
| **Linux** | Sistema operativo destino |
| **Plugin GitHub (Jenkins)** | Integración Jenkins ↔ GitHub |
| **Plugin Email Extension** | Notificación de contraseñas temporales |

---

## 🏗️ Arquitectura

```
GitHub (scripts + Jenkinsfile)
        ↓  clone
Jenkins (Pipeline Declarativo)
        ↓  ejecuta
Bash Scripts en Linux
   ├── create_user.sh  → crea usuario, asigna grupo, genera contraseña temporal
   └── delete_user.sh  → elimina usuario, grupo y directorio home
```

---

## 📁 Archivos del Proyecto

```
desafios/
├── create_user.sh          ← Crea usuario con contraseña temporal segura
├── delete_user.sh          ← Elimina usuario y su directorio home
├── pipeline.jenkinsfile    ← Pipeline para crear usuario (parametrizado)
└── eliminar.jenkinsfile    ← Pipeline para eliminar usuario (parametrizado)
```

---

## ⚙️ Scripts Bash

### `create_user.sh`
```bash
#!/bin/bash
LOGIN=$1
FULL_NAME=$2
DEPARTMENT=$3
PASSWORD=$(openssl rand -base64 12)
sudo useradd -m -c "$FULL_NAME" -s /bin/bash -g "$DEPARTMENT" "$LOGIN"
echo "$LOGIN:$PASSWORD" | sudo chpasswd
sudo passwd -e "$LOGIN"
echo "Usuario: $LOGIN"
echo "Contraseña Temporal: $PASSWORD"
```

### `delete_user.sh`
```bash
#!/bin/bash
LOGIN=$1
sudo userdel -r "$LOGIN"
echo "usuario fue eliminado"
```

---

## 🔁 Pipelines Jenkins

### Pipeline Crear Usuario (parametrizado)
```groovy
pipeline {
  agent any
  parameters {
    string(name: 'LOGIN', description: 'Login del usuario (nombre.apellido)')
    string(name: 'FULL_NAME', description: 'Nombre y Apellido del usuario')
    choice(name: 'DEPARTMENT',
      choices: 'contabilidad\nfinanzas\ntecnologia',
      description: 'Departamento del usuario')
  }
  stages {
    stage('Crear Usuario') {
      steps {
        sh 'bash create_user.sh "${LOGIN}" "${FULL_NAME}" "${DEPARTMENT}"'
      }
    }
  }
}
```

### Pipeline Eliminar Usuario (parametrizado)
```groovy
pipeline {
  agent any
  parameters {
    string(name: 'LOGIN', description: 'Login del usuario a eliminar (nombre.apellido)')
  }
  stages {
    stage('Eliminar Usuario') {
      steps {
        sh 'bash delete_user.sh "${LOGIN}" "${FULL_NAME}" "${DEPARTMENT}"'
      }
    }
  }
}
```

---

## ✅ Resultado

Pipeline ejecutado con éxito — `Finished: SUCCESS` — creando y eliminando usuarios en Linux de forma automática desde Jenkins, con generación de contraseña temporal segura via `openssl rand`.

---

## 🧠 Conceptos Aplicados

- Pipeline **declarativo** con parámetros de entrada en Jenkins
- Scripts Bash para administración de usuarios en Linux (`useradd`, `userdel`, `chpasswd`)
- Generación de contraseñas seguras con `openssl rand -base64`
- Integración Jenkins ↔ GitHub via plugin y credenciales (token)
- Manejo de errores en Bash con `exit 1` y verificación de resultados

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
