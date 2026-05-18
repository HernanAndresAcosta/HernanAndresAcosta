# 🖥️ Desafío N°3 — Script Bash: Informe del Sistema con Funciones y Manejo de Errores
**Autor:** Hernán Andrés Acosta &nbsp;|&nbsp; **Bootcamp:** DevOps Engineer — EducaciónIT

---

## 📌 Descripción

Mejora y refactorización de un script Bash de informe del sistema (`informe_del_sistema.sh`), aplicando **buenas prácticas de programación**: organización en funciones, función `main()` como punto de entrada, y manejo robusto de errores con `exit 1`.

---

## 🛠️ Stack Utilizado

| Herramienta | Rol |
|-------------|-----|
| **Bash** | Lenguaje del script |
| **Linux** | Sistema operativo donde se ejecuta |
| **Comandos del sistema** | `df`, `free`, `who`, `ps`, `hostname`, `date` |

---

## 🎯 Mejoras Implementadas

| Mejora | Descripción |
|--------|-------------|
| **Funciones modulares** | Cada funcionalidad separada en su propia función |
| **Función `main()`** | Punto de entrada único que orquesta la ejecución |
| **Manejo de errores** | Verificación de resultados con `exit 1` ante fallos |
| **Comentarios mejorados** | Documentación clara de propósito de cada función |

---

## 📋 Funciones del Script

```bash
#!/bin/bash

# Encabezado del informe con nombre del host
imprimir_encabezado()

# Fecha y hora actual formateada
mostrar_fecha_hora()

# Uso del disco — partición raíz (/)
mostrar_uso_disco()

# Lista de usuarios logueados actualmente
mostrar_usuarios_logueados()

# Uso de memoria RAM en Megabytes
mostrar_uso_memoria()

# Búsqueda interactiva de procesos por nombre
buscar_proceso()

# Función principal — ejecuta todo en orden
main()
```

---

## 💻 Script Completo

```bash
#!/bin/bash

imprimir_encabezado() {
    echo "Informe del Sistema para: $(hostname)"
    echo ""
}

mostrar_fecha_hora() {
    echo "Fecha y Hora: $(date +'%d/%m/%Y %H:%M:%S')"
    echo ""
}

mostrar_uso_disco() {
    echo "Uso del disco del sistema (partición raíz /):"
    resultado=$(df -h | grep '^/')
    if [ -z "$resultado" ]; then
        echo "Error: No se pudo obtener el uso del disco."
        exit 1
    fi
    echo "$resultado"
    echo ""
}

mostrar_usuarios_logueados() {
    echo "Usuarios actualmente logueados:"
    resultado=$(who)
    if [ -z "$resultado" ]; then
        echo "Error: No se pudo obtener la lista de usuarios."
        exit 1
    fi
    echo "$resultado"
    echo ""
}

mostrar_uso_memoria() {
    echo "Uso de memoria (en Megabytes):"
    resultado=$(free -m)
    if [ $? -ne 0 ]; then
        echo "Error: No se pudo obtener el uso de memoria."
        exit 1
    fi
    echo "$resultado"
    echo ""
}

buscar_proceso() {
    read -p "Ingresa el nombre o parte del nombre de un proceso: " proceso
    echo "Procesos que coinciden con '$proceso':"
    resultados=$(ps -aux | grep "$proceso")
    if [ -z "$resultados" ]; then
        echo "Error: No se encontraron procesos que coincidan con '$proceso'."
        exit 1
    fi
    echo "$resultados"
}

main() {
    imprimir_encabezado
    mostrar_fecha_hora
    mostrar_uso_disco
    mostrar_usuarios_logueados
    mostrar_uso_memoria
    buscar_proceso
}

main
```

---

## ✅ Ejemplo de Salida

```
Informe del Sistema para: AhA

Fecha y Hora: 11/12/2024 13:53:06

Uso del disco del sistema (partición raíz /):
/dev/sdb2   176G  25G  143G  15% /
/dev/sdd1    15G  40K   15G   1% /media/hernan/NUEVO VOL

Usuarios actualmente logueados:
hernan   seat0    2024-12-11 10:20 (login screen)
hernan   tty2     2024-12-11 10:20 (tty2)

Uso de memoria (en Megabytes):
        total   usado   libre   compartido   búf/caché   disponible
Mem:     7835    5829     467          381        2181         2006
Inter:   4095      51    4044

Ingresa el nombre o parte del nombre de un proceso: █
```

---

## 🧠 Conceptos Aplicados

- **Modularización** en Bash con funciones reutilizables
- Patrón **función `main()`** como punto de entrada del script
- **Manejo de errores** con `exit 1` y verificación de variables vacías (`[ -z ]`)
- Verificación de código de salida con `$?`
- Comandos de administración del sistema: `df`, `free -m`, `who`, `ps -aux`, `hostname`, `date`
- Entrada interactiva del usuario con `read -p`
- Filtrado de salidas con `grep`

---

## 👤 Autor

**Hernán Andrés Acosta** — DevOps Engineer en formación  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hernán_Acosta-blue?logo=linkedin)](https://www.linkedin.com/in/hernan-a-acosta)
[![GitHub](https://img.shields.io/badge/GitHub-HernanAndresAcosta-black?logo=github)](https://github.com/HernanAndresAcosta)
