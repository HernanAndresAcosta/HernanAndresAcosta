# ☁️ Desafío N°4 — AWS S3 + IAM: Roles, Políticas y AWS CLI
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT

---

## 📌 Descripción

Implementación de una arquitectura de **acceso seguro a S3 mediante IAM** en AWS. Se creó un usuario con credenciales programáticas que asume un rol con permisos específicos sobre un bucket S3, aplicando el principio de **mínimo privilegio** y el patrón **AssumeRole** via AWS CLI.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **AWS IAM** | Gestión de usuarios, roles y políticas |
| **AWS S3** | Servicio de almacenamiento objetivo |
| **AWS STS** | AssumeRole — credenciales temporales |
| **AWS CLI** | Interacción programática con AWS |

---

## 🏗️ Arquitectura de Seguridad

```
s3-support (Usuario IAM)
   └── Política inline: s3OperatorAssumerol
         └── Permite: sts:AssumeRole → S3OperatorRole
                              ↓
                    S3OperatorRole (Rol IAM)
                         └── Políticas adjuntas:
                               ├── AmazonS3OutpostsReadO...
                               └── AmazonS3ReadOnlyAccess
                                         ↓
                               s3-hernan-eduit (Bucket S3)
```

---

## 📋 Orden de Creación de Objetos

```
1. Crear bucket S3          → s3-hernan-eduit (us-east-1)
2. Crear usuario IAM        → s3-support (solo credenciales programáticas)
3. Crear rol IAM            → S3OperatorRole
4. Adjuntar políticas al rol → AmazonS3ReadOnlyAccess + AmazonS3OutpostsReadOnly
5. Crear política inline    → s3OperatorAssumerol (en el usuario s3-support)
6. Configurar AWS CLI       → perfil s3operator con credenciales del usuario
7. Asumir el rol via STS    → obtener credenciales temporales
8. Verificar acceso a S3    → listar buckets con el rol asumido
```

---

## ⚙️ Comandos AWS CLI Utilizados

### Configurar perfil del usuario
```bash
aws configure --profile s3operator
# AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key: ***
# Default region name: us-east-1
```

### Asumir el rol con STS
```bash
aws --profile s3operator sts assume-role \
  --role-arn arn:aws:iam::207567791252:role/s3OperatorRole \
  --role-session-name s3OperatorRole-session
```

> Esto retorna credenciales temporales: `AccessKeyId`, `SecretAccessKey` y `SessionToken` con fecha de expiración.

### Configurar credenciales temporales del rol
```bash
# Editar ~/.aws/credentials con las credenciales obtenidas del AssumeRole
[s3operator]
aws_access_key_id = ASIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
aws_session_token = AQoXnyc4lcK4w4OIAAAAAAAXXbbbbbbb...
```

### Verificar identidad del rol asumido
```bash
aws --profile s3operator sts get-caller-identity
# {
#   "UserId": "AROATAVABESKMDOW2FG4U:s3OperatorRole-session",
#   "Account": "207567791252",
#   "Arn": "arn:aws:sts::207567791252:assumed-role/s3OperatorRole/s3OperatorRole-session"
# }
```

### Listar buckets S3
```bash
aws --profile s3operator s3 ls
# 2025-01-24 15:05:37 s3-hernan-eduit ✅
```

---

## 🔐 Política Inline del Usuario (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Statement1",
      "Effect": "Allow",
      "Action": ["sts:AssumeRole"],
      "Resource": ["arn:aws:iam::207567791252:role/s3OperatorRole"]
    }
  ]
}
```

---

## 🔐 Trust Policy del Rol (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Principal": {
        "AWS": "207567791252"
      },
      "Condition": {}
    }
  ]
}
```

---

## ✅ Resultado Verificado

```bash
# Sin asumir el rol → AccessDenied
aws --profile s3operator s3 ls
# An error occurred (AccessDenied): not authorized to perform s3:ListAllMyBuckets

# Con el rol asumido → Acceso concedido ✅
aws --profile s3operator s3 ls
# 2025-01-24 15:05:37 s3-hernan-eduit
```

---

## 🧠 Conceptos Aplicados

- **Principio de mínimo privilegio** — el usuario no tiene permisos directos, solo puede asumir un rol
- **IAM AssumeRole** — patrón de delegación de permisos temporales via STS
- **Credenciales temporales** con `SessionToken` y fecha de expiración
- **Política inline** vs **política administrada** por AWS
- **Trust Policy** — define quién puede asumir el rol
- Configuración de **múltiples perfiles** en AWS CLI (`~/.aws/credentials`)
- Verificación de identidad con `sts get-caller-identity`

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
