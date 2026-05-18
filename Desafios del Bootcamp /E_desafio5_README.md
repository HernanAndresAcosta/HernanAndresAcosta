# 🔐 Desafío N°5 — AWS IAM: Usuario Admin + VPC + RDS MariaDB
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT

---

## 📌 Descripción

Configuración completa de infraestructura AWS: creación de un **usuario IAM administrador** con MFA, diseño y despliegue de una **VPC con subredes públicas**, configuración de **Security Groups**, y despliegue de una **instancia RDS MariaDB** accesible desde la consola local.

---

## 🛠️ Stack Utilizado

| Servicio | Uso |
|----------|-----|
| **AWS IAM** | Usuario admin con MFA + política AdministratorAccess |
| **Amazon VPC** | Red virtual privada con subredes públicas |
| **Security Groups** | Control de tráfico entrante/saliente |
| **Amazon RDS** | Instancia MariaDB en la VPC |
| **AWS CLI / MariaDB client** | Verificación de conectividad |

---

## 🏗️ Arquitectura Implementada

```
Amazon VPC (10.0.0.0/16)
─────────────────────────────────────────
  Public Subnet 1     Public Subnet 2
  (10.0.1.0/24)       (10.0.2.0/24)
  us-east-1a          us-east-1b
       │                   │
       └───────┬───────────┘
               │
        Internet Gateway
               │
           Internet
─────────────────────────────────────────
  RDS MariaDB (db.t4g.micro)
  Endpoint: database-1.c728cuoy6dkg.us-west-2.rds.amazonaws.com
  Port: 3306 | Engine: MariaDB 11.4.4
```

---

## 👤 Parte 1 — Creación de Usuario IAM Administrador

### Pasos realizados

```
1. Inicio de sesión con usuario raíz (única vez)
2. IAM → Usuarios → Agregar usuario → "Admin_Hernan"
3. Tipo de credenciales:
   - Acceso a Consola de AWS ✅
   - Contraseña personalizada ✅
4. MFA habilitado ✅
5. Permisos → Adjuntar política: AdministratorAccess ✅
6. Descarga de CSV con credenciales
7. Verificación: login con nuevo usuario IAM ✅
```

> **Buena práctica:** Nunca operar con el usuario raíz en el día a día. El usuario IAM con `AdministratorAccess` es suficiente para todas las tareas administrativas, con la ventaja de poder auditar y revocar accesos.

---

## 🌐 Parte 2 — Creación de VPC

### Configuración aplicada

| Parámetro | Valor |
|-----------|-------|
| VPC ID | `vpc-030c985080ad4edc7` |
| CIDR Block | `10.0.0.0/16` |
| DNS Resolution | Enabled |
| DNS Hostnames | Enabled |

### Subredes Públicas Creadas

| Subred | CIDR Block | Availability Zone | Route Table | Internet Gateway |
|--------|------------|-------------------|-------------|-----------------|
| Subnet 1 | 10.0.1.0/24 | us-east-1a | Public-Route-Table | igw-id |
| Subnet 2 | 10.0.2.0/24 | us-east-1b | Public-Route-Table | igw-id |

### Configuración Route Table
```
Destino: 0.0.0.0/0 → Internet Gateway
Auto-assign public IPv4: habilitado en subredes públicas
```

---

## 🔒 Parte 3 — Security Groups

### Regla configurada para RDS

| Type | Protocol | Port Range | Source |
|------|----------|------------|--------|
| MYSQL/Aurora | TCP | 3306 | IP específica /32 |

> **Nota de seguridad:** Se evitó `0.0.0.0/0` (acceso universal) y se configuró acceso solo desde la IP específica del equipo de trabajo. En producción siempre usar IPs específicas o rangos controlados.

---

## 🗄️ Parte 4 — RDS MariaDB en la VPC

### Configuración de la instancia

| Parámetro | Valor |
|-----------|-------|
| DB Identifier | `database-1` |
| Engine | MariaDB 11.4.4 |
| Instance Class | `db.t4g.micro` (Free Tier) |
| Availability Zone | us-west-2b |
| VPC | tutorial-vpc |
| Subnet Group | tutorial-db-subnet-group |
| Public Access | Yes |

### Conexión verificada desde consola local

```bash
mariadb -h database-1.c728cuoy6dkg.us-west-2.rds.amazonaws.com -uadmin -p

# Welcome to the MariaDB monitor.
# Your MariaDB connection id is 72
# Server version: 11.4.4-MariaDB-log managed by https://aws.amazon.com/rds/

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| innodb             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0,217 sec) ✅
```

---

## 🧠 Conceptos Aplicados

- **IAM Best Practices** — nunca usar root, MFA obligatorio, política de mínimo privilegio
- **VPC Design** — subredes públicas/privadas, Internet Gateway, Route Tables
- **Security Groups** — firewall stateful a nivel de instancia, reglas por IP específica
- **DB Subnet Group** — agrupa subredes para alta disponibilidad de RDS
- **RDS Free Tier** — instancia `db.t4g.micro` para entornos de desarrollo/prueba
- **Conectividad RDS** — acceso externo controlado via endpoint + puerto 3306

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
