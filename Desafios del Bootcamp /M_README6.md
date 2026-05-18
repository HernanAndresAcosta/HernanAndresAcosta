# 📦 Desafío N°6 — Ansible: Modularización y Despliegue de Sitio Web
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT  
**Fecha:** 21/05/2025

---

## 📌 Descripción

Reorganización y modularización de un proyecto Ansible para automatizar el despliegue de un sitio web estático con **Apache2** en una VM Ubuntu (Multipass), aplicando buenas prácticas de **Infraestructura como Código**: estructura modular, variables centralizadas, templates Jinja2 y handlers para recarga de servicios.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **Ansible** | Automatización de configuración |
| **Apache2** | Servidor web desplegado |
| **Multipass** | VM Ubuntu local (`web1`) |
| **SSH Ed25519** | Autenticación segura |
| **Jinja2** | Templates de configuración Apache |

---

## 🏗️ Arquitectura

```
Máquina local (controlador Ansible)
         ↓  SSH Ed25519
VM Multipass: web1 (10.170.126.79)
         └── Apache2
               └── /var/www/html/ansible_site/
                     └── index.html → "mi sitio ansible"
```

---

## 📁 Estructura Modularizada del Proyecto

```
desafio6/ansible/
├── main.yml                    ← Playbook principal
├── inventory.ini               ← Inventario adaptado a Multipass
├── Jenkinsfile                 ← Pipeline para CI/CD
├── Jenkinsfile-validate        ← Pipeline de validación
├── files/
│   └── index.html              ← Contenido del sitio web
├── includes/
│   └── install-apache2.yml     ← Tarea modular: instalar Apache
├── templates/
│   └── ansible_site.conf.j2    ← Template VirtualHost Apache
└── vars/
    └── vars-site.yml           ← Variables centralizadas
```

> **Principio aplicado:** cada responsabilidad en su propio archivo — instalación, configuración, variables y contenido separados y reutilizables.

---

## ⚙️ Archivos Clave

### `inventory.ini`
```ini
[all:vars]
ansible_python_interpreter=/usr/bin/python3

[webservers]
web1 ansible_host=10.170.126.79

[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/hernan/.ssh/id_ed25519
ansible_python_interpreter=/usr/bin/python3
```

### `vars/vars-site.yml` — Variables centralizadas
```yaml
site_dir: /var/www/html/ansible_site   # Ruta del sitio
apache_conf: ansible_site.conf         # Nombre del archivo de configuración
```

### `includes/install-apache2.yml` — Módulo reutilizable
```yaml
- name: Actualizar el cache de paquetes
  apt:
    update_cache: yes
  when: is_ubuntu

- name: Instalar Apache2
  apt:
    name: apache2
    state: present
  when: is_ubuntu

- name: Verificar que apache esté corriendo
  service:
    name: apache2
    state: started
    enabled: yes
  when: is_ubuntu
```

### `templates/ansible_site.conf.j2` — VirtualHost Apache
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/ansible_site

    <Directory /var/www/html/ansible_site>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### `main.yml` — Playbook principal
```yaml
- name: Deployment de un sitio estático
  hosts: all
  become: yes
  pre_tasks:
    - name: Verificar si el OS es Ubuntu
      set_fact:
        is_ubuntu: "{{ ansible_distribution == 'Ubuntu' }}"
  vars_files:
    - vars/vars-site.yml
  tasks:
    - name: Instalar servicio Apache
      include_tasks: includes/install-apache2.yml

    - name: Crear directorio para el sitio
      file:
        path: "{{ site_dir }}"
        state: directory
        owner: www-data
        group: www-data
      when: is_ubuntu

    - name: Copiar index.html al directorio del sitio
      copy:
        src: files/index.html
        dest: "{{ site_dir }}"
        owner: www-data
        group: www-data
        mode: '0644'
      when: is_ubuntu

    - name: Configuración del sitio apache
      template:
        src: templates/ansible_site.conf.j2
        dest: /etc/apache2/sites-available/ansible_site.conf
      notify: Reload Apache
      when: is_ubuntu

    - name: Activar el nuevo sitio
      command: a2ensite ansible_site.conf
      notify: Reload Apache
      when: is_ubuntu

    - name: Deshabilitar el sitio default
      command: a2dissite 000-default.conf
      notify: Reload Apache
      when: is_ubuntu

    - name: Cambiar el e-mail del webmaster
      lineinfile:
        path: /etc/apache2/sites-available/000-default.conf
        regexp: 'ServerAdmin webmaster@localhost'
        line: 'ServerAdmin webmaster@educacionit.com'
      notify: Reload Apache
      when: is_ubuntu

  handlers:
    - name: Reload Apache
      service:
        name: apache2
        state: reloaded
      when: is_ubuntu
```

---

## 🚀 Proceso de Despliegue

### 1. Preparar el entorno
```bash
# Crear VM con Multipass
multipass launch --name web1 --cpus 1 --mem 1G --disk 5G

# Generar clave SSH Ed25519
ssh-keygen -t ed25519 -C "ansible-key"

# Copiar clave pública a la VM
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@10.170.126.79
```

### 2. Verificar conectividad
```bash
ansible all -i inventory.ini -m ping

# web1 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# } ✅
```

### 3. Ejecutar el playbook
```bash
ansible-playbook -i inventory.ini main.yml
```

### 4. Verificar el sitio desplegado
```bash
curl http://10.170.126.79
# → <html><p>mi sitio ansible</p></html> ✅
```

---

## ✅ Resultado del Playbook

```
TASK [Actualizar el cache de paquetes]    changed ✅
TASK [Instalar Apache2]                   ok ✅
TASK [Verificar que apache esté corriendo] ok ✅
TASK [Crear directorio para el sitio]     ok ✅
TASK [Copiar index.html]                  ok ✅
TASK [Configuración del sitio apache]     ok ✅
TASK [Activar el nuevo sitio]             changed ✅
TASK [Deshabilitar el sitio default]      changed ✅
TASK [Cambiar el e-mail del webmaster]    ok ✅
RUNNING HANDLER [Reload Apache]           changed ✅

PLAY RECAP
web1: ok=13  changed=4  unreachable=0  failed=0 ✅
```

---

## ☁️ Anexo — Adaptación a AWS EC2

El proyecto está preparado para escalar a AWS con mínimos cambios:

```bash
# Crear instancias EC2
aws ec2 run-instances \
  --image-id ami-08c40ec9ead489470 \   # Ubuntu 22.04 LTS
  --count 2 \
  --instance-type t2.micro \
  --key-name ansible-aws-key \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=webserver-ansible}]'

# Ejecutar playbook con inventario dinámico
ansible-playbook -i aws_ec2.yml main.yml
```

**Inventario dinámico para AWS (`aws_ec2.yml`):**
```yaml
plugin: aws_ec2
regions:
  - us-east-1
filters:
  tag:Name: webserver-ansible
compose:
  ansible_host: public_ip_address
```

---

## 🧠 Conceptos Aplicados

- **Modularización Ansible** — separación en `includes/`, `templates/`, `vars/`, `files/`
- **`include_tasks`** — reutilización de tareas entre playbooks
- **`vars_files`** — variables centralizadas, única fuente de verdad
- **Templates Jinja2** — configuración dinámica de Apache VirtualHost
- **`handlers`** — recarga de Apache solo cuando hay cambios reales
- **`set_fact`** — detección dinámica del OS para condicionales
- **`when: is_ubuntu`** — condicionales para portabilidad multi-OS
- **`lineinfile`** — modificación quirúrgica de archivos de configuración
- **SSH Ed25519** — autenticación segura sin contraseña
- **Idempotencia** — el playbook puede ejecutarse múltiples veces sin efectos no deseados

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
