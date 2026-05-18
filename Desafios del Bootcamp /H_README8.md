# 🏗️ Desafío N°8 — IaC Automatizado: Terraform Cloud + GitHub Actions + AWS
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT  
**Fecha:** 31/03/2025

---

## 📌 Descripción

Implementación de un pipeline de **Infraestructura como Código (IaC)** completamente automatizado, integrando **HCP Terraform** para gestión centralizada del estado, **GitHub Actions** para CI/CD, y **AWS** como proveedor cloud. El flujo incluye revisión de cambios via Pull Request antes del despliegue en producción.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **Terraform / HCP Terraform** | IaC + gestión centralizada de estado |
| **GitHub Actions** | CI/CD automatizado (plan + apply) |
| **AWS** | Proveedor cloud — instancia EC2 desplegada |
| **GitHub Secrets** | Credenciales seguras para AWS y Terraform |

---

## 🏗️ Flujo de Trabajo Implementado

```
[Desarrollador]
      ↓  git push (rama feature)
[Repositorio GitHub]
      ↓  Pull Request abierto
[GitHub Actions — terraform-plan.yml]
      ↓  ejecuta Terraform Plan (especulativo)
[HCP Terraform]
      ↓  muestra cambios en comentario del PR
[Revisión y aprobación del PR]
      ↓  merge a main
[GitHub Actions — terraform-apply.yml]
      ↓  ejecuta Terraform Apply
[HCP Terraform]
      ↓  gestiona estado y aplica cambios
[AWS — Recursos creados]
      └── EC2 t2.micro con acceso HTTP ✅
```

---

## 🔐 Configuración de Secrets

### En GitHub (`Settings → Secrets → Actions`)
| Secret | Descripción |
|--------|-------------|
| `TF_API_TOKEN` | Token de API de HCP Terraform |
| `AWS_ACCESS_KEY_ID` | Credencial AWS (write-only en HCP Terraform) |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS (write-only en HCP Terraform) |

### En HCP Terraform (variables de entorno sensibles)
| Variable | Categoría |
|----------|-----------|
| `AWS_ACCESS_KEY_ID` | Environment — Sensible |
| `AWS_SECRET_ACCESS_KEY` | Environment — Sensible |

---

## ⚙️ Workflows de GitHub Actions

### `terraform-plan.yml` — Se ejecuta en Pull Requests
```yaml
name: "Terraform Plan"
on:
  pull_request:

env:
  TF_CLOUD_ORGANIZATION: "mi-org-tf"
  TF_API_TOKEN: "${{ secrets.TF_API_TOKEN }}"
  TF_WORKSPACE: "learn-terraform-github-actions"
  CONFIG_DIRECTORY: "./"

jobs:
  terraform:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v3
      - uses: hashicorp/tfc-workflows-github/actions/upload-configuration@v1.0.0
        with:
          workspace: ${{ env.TF_WORKSPACE }}
          directory: ${{ env.CONFIG_DIRECTORY }}
          speculative: true   # Plan especulativo — no aplica cambios
      - uses: hashicorp/tfc-workflows-github/actions/create-run@v1.0.0
        with:
          plan_only: true
      # → Publica resultado del plan como comentario en el PR
```

### `terraform-apply.yml` — Se ejecuta en push a main
```yaml
name: "Terraform Apply"
on:
  push:
    branches:
      - main

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: hashicorp/tfc-workflows-github/actions/upload-configuration@v1.0.0
      - uses: hashicorp/tfc-workflows-github/actions/create-run@v1.0.0
      - uses: hashicorp/tfc-workflows-github/actions/apply-run@v1.0.0
        if: fromJSON(steps.apply-run.outputs.payload).data.attributes.actions.IsConfirmable
        with:
          comment: "Apply Run from GitHub Actions CI ${{ github.sha }}"
```

---

## 📁 Estructura del Proyecto

```
learn-terraform-github-actions/
├── .github/
│   └── workflows/
│       ├── terraform-plan.yml    ← Ejecuta en PR
│       └── terraform-apply.yml  ← Ejecuta en merge a main
├── .terraform/
├── main.tf                       ← Configuración principal de Terraform
├── .terraform.lock.hcl
└── README.md
```

---

## ✅ Recursos Desplegados en AWS

**Instancia EC2 creada automáticamente:**

| Parámetro | Valor |
|-----------|-------|
| Instance ID | `i-0beda5f6445428d30` |
| Instance Type | `t2.micro` |
| Public IP | `34.218.209.185` |
| Region | us-west-2 |
| Estado | Running ✅ |

---

## 📈 Mejoras Propuestas

Estructura modular sugerida (documentada en el desafío):

```
learn-terraform-github-actions/
├── .github/workflows/
├── modules/                    ← Módulos reutilizables
│   └── example-module/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── environments/               ← Configuración por entorno
│   ├── dev/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   └── prod/
│       ├── main.tf
│       ├── terraform.tfvars
│       └── backend.tf
├── main.tf
├── providers.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars.example    ← Sin valores sensibles
```

---

## 🧠 Conceptos Aplicados

- **IaC con Terraform** — infraestructura declarativa y versionada
- **HCP Terraform** — estado remoto centralizado y colaborativo
- **GitOps workflow** — cambios de infra revisados via PR antes de aplicar
- **Terraform Plan especulativo** — preview de cambios sin aplicarlos
- **GitHub Actions + Terraform Cloud integration** — pipeline IaC end-to-end
- **Secrets management** — credenciales nunca en código, siempre en Secrets
- **Separación plan/apply** — revisión obligatoria antes del despliegue en producción

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
