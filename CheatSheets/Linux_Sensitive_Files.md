# Linux Sensitive Files Cheat Sheet

Guía de referencia rápida sobre archivos críticamente sensibles, ficheros de configuración, logs de auditoría y directorios clave en sistemas Linux para tareas de seguridad, pentesting y análisis forense.

---

## Referencias y Recursos

| Recurso | Descripción |
| :--- | :--- |
| [HackTricks: Linux Privilege Escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation) | Guía de escalada de privilegios y archivos de interés. |
| [PayloadsAllTheThings: Linux - Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation) | Métodos y ubicaciones sensibles en Linux. |

---

## 1. Identidades, Grupos y Credenciales

### `/etc/passwd`
* Información general de las cuentas de usuario locales (nombre de usuario, UID/GID, directorio home, shell).
* Históricamente contenía hashes de contraseñas. Si el campo de contraseña (segundo campo) tiene un hash en lugar de una `x`, es vulnerable a descifrado offline (ver [Password Cracking](Tools/cracking.md)).
  ```bash
  cat /etc/passwd
  ```

### `/etc/shadow`
* Contiene los hashes de contraseñas cifradas y la información de expiración de las cuentas de usuario.
* **OPSEC / Privilegios**: Solo lectura para `root` o miembros del grupo `shadow`. Su lectura directa suele implicar privilegios elevados.
  ```bash
  cat /etc/shadow
  ```

### `/etc/group` y `/etc/gshadow`
* `/etc/group`: Define los grupos del sistema y qué usuarios pertenecen a ellos.
* `/etc/gshadow`: Versión segura con contraseñas de grupos cifradas.
  ```bash
  cat /etc/group
  ```

### `/etc/sudoers` y `/etc/sudoers.d/*`
* Configuración de permisos de `sudo`. Define qué usuarios/grupos pueden ejecutar comandos como otros usuarios (incluyendo `root`), con o sin contraseña.
* Buscar directivas como `NOPASSWD:` o `env_keep+=LD_PRELOAD`.
  ```bash
  cat /etc/sudoers
  ls -la /etc/sudoers.d/
  ```

### `/etc/opasswd`
* Contiene contraseñas antiguas de usuarios en sistemas que usan PAM (`pam_cracklib` / `pam_unix`) para evitar que se reutilicen contraseñas recientes.

---

## 2. Archivos y Directorios de Acceso SSH e Identidades

### Directorio `~/.ssh/` (Por Usuario)
Contiene las credenciales para la autenticación SSH del usuario. Es uno de los objetivos más comunes para el movimiento lateral.
* `id_rsa`, `id_dsa`, `id_ecdsa`, `id_ed25519`: Claves privadas SSH. **NUNCA** deben compartirse.
* `authorized_keys`: Lista de claves públicas autorizadas para iniciar sesión como este usuario sin contraseña.
* `known_hosts`: Direcciones IP y claves públicas de hosts a los que el usuario se ha conectado previamente (útil para mapear la red).
* `config`: Configuraciones de cliente SSH personalizadas (puede contener alias y puertos alternativos).
  ```bash
  ls -la ~/.ssh/
  cat ~/.ssh/id_rsa 2>/dev/null
  ```

### Claves y Directorios GPG/PGP
* Ubicado en `~/.gnupg/`. Contiene claves privadas y públicas para cifrado de correo y archivos.
  ```bash
  gpg --list-secret-keys
  ```

---

## 3. Logs de Sistema, Autenticación y Actividad

### Historiales de Consola (Shell History)
Ficheros donde se registran los comandos ejecutados por los usuarios. Pueden contener contraseñas introducidas por error como argumentos o rutas secretas.
* `~/.bash_history` (Bash)
* `~/.zsh_history` (Zsh)
* `~/.sh_history` o `~/.history`
* `~/.mysql_history` (Historial de comandos de MySQL/MariaDB)
  ```bash
  cat ~/.bash_history
  cat ~/.zsh_history
  cat ~/.mysql_history
  ```

### Directorio de Logs `/var/log/`
* `/var/log/auth.log` (Debian/Ubuntu) o `/var/log/secure` (Rhel/CentOS): Intentos de inicio de sesión, uso de `sudo`, SSH, autenticaciones fallidas.
* `/var/log/syslog` o `/var/log/messages`: Log principal del sistema.
* `/var/log/cron` o `/var/log/cron.log`: Registro de tareas cron ejecutadas.
* `/var/log/apache2/` o `/var/log/nginx/`: Logs de acceso y errores web (posible inyección de código / Log Poisoning).
* `/var/log/tallylog`: Registro de intentos de inicio de sesión fallidos.
* `/var/log/wtmp` y `/var/log/btmp`: Archivos binarios que registran inicios de sesión correctos y fallidos. Se leen con los comandos `last` y `lastb`.
  ```bash
  tail -n 50 /var/log/auth.log
  last
  lastb
  ```

---

## 4. Configuraciones de Servicios y Credenciales Web

### Archivos de Configuración de Bases de Datos
A menudo contienen contraseñas de administración o credenciales de conexión en texto plano:
* **MySQL/MariaDB**: `/etc/mysql/my.cnf`, `/etc/my.cnf`, `~/.my.cnf` (credenciales automáticas del cliente).
* **PostgreSQL**: `/etc/postgresql/[version]/main/postgresql.conf`, `~/.pgpass` (contraseñas almacenadas de PostgreSQL).
* **Redis**: `/etc/redis/redis.conf` (buscar `requirepass` o `bind` sin autenticación).
  ```bash
  grep -i "password" /etc/mysql/my.cnf 2>/dev/null
  cat ~/.pgpass 2>/dev/null
  ```

### Archivos de Configuración de Aplicaciones Web
Archivos comunes en los directorios de publicación web (`/var/www/html/`, `/srv/http/`, `/opt/`) que suelen contener secretos:
* `.env` y `.env.production` (Común en Node.js, Laravel, Django).
* `wp-config.php` (WordPress).
* `config.php`, `database.php` o `settings.py`.
* Archivos `.git` (repositorios expuestos con historiales completos de código).
  ```bash
  find /var/www/ -name ".env" -o -name "wp-config.php" 2>/dev/null
  ```

### Archivos de Servidor Web y Proxy
* `/etc/apache2/apache2.conf`, `/etc/apache2/sites-enabled/*` (Apache)
* `/etc/nginx/nginx.conf`, `/etc/nginx/sites-enabled/*` (Nginx)
* `/etc/haproxy/haproxy.cfg` (HAProxy)

### VPN y Enrutamiento
* `/etc/openvpn/` o archivos `.ovpn` (Configuraciones de OpenVPN, pueden incluir certificados incrustados o rutas a archivos de credenciales).

---

## 5. Montajes, Tareas Programadas e Información del Kernel

### `/etc/fstab` y `/etc/exports`
* `/etc/fstab`: Montajes automáticos de discos y carpetas de red. Puede incluir credenciales en texto plano para sistemas Samba/CIFS o NFS.
* `/etc/exports`: Configuración de directorios compartidos vía NFS. Si contiene la opción `no_root_squash`, es explotable para escalada de privilegios local.
  ```bash
  cat /etc/fstab
  cat /etc/exports
  ```

### Tareas Programadas (Cron Jobs)
Las tareas programadas pueden ejecutarse como root. Modificar o secuestrar los scripts a los que apuntan permite comprometer el sistema.
* `/etc/crontab` (Cron del sistema principal)
* `/etc/cron.d/*`, `/etc/cron.daily/*`, `/etc/cron.hourly/*`, `/etc/cron.monthly/*`, `/etc/cron.weekly/*`
* `/var/spool/cron/crontabs/*` (Crontabs individuales de usuarios)
  ```bash
  cat /etc/crontab
  ls -la /var/spool/cron/crontabs/
  ```

### El Directorio Dinámico `/proc/`
El sistema de archivos `/proc` contiene información en tiempo real sobre procesos y hardware.
* `/proc/version`: Versión del kernel y del compilador (útil para buscar exploits de kernel locales).
* `/proc/cmdline`: Parámetros pasados al kernel durante el arranque (puede contener secretos o modos de recuperación de root).
* `/proc/self/environ` o `/proc/[PID]/environ`: Variables de entorno asociadas a un proceso. Pueden contener API keys, tokens de sesión o contraseñas en memoria de la sesión activa del usuario.
* `/proc/net/tcp` y `/proc/net/udp`: Conexiones activas en formato hexadecimal (se mapean con `/proc/[PID]/fd/`).
  ```bash
  cat /proc/version
  cat /proc/cmdline
  strings /proc/self/environ
  ```

### Archivos del Sistema / Información de la Distribución
* `/etc/issue` y `/etc/issue.net`: Banners pre-autenticación que revelan la distribución exacta del SO.
* `/etc/os-release` y `/etc/lsb-release`: Información formal de la versión del sistema operativo.
  ```bash
  cat /etc/os-release
  ```

---

## 6. Comandos de Búsqueda Rápida (Reconocimiento / Post-Explotación)

### Buscar Archivos de Configuración y Claves Privadas
* Buscar claves privadas SSH en todo el sistema:
  ```bash
  find / -name "id_rsa" -o -name "id_dsa" -o -name "id_ed25519" -o -name "id_ecdsa" 2>/dev/null
  ```
* Buscar archivos con extensión `.env` o `.conf`:
  ```bash
  find / -type f -name "*.env" -o -name "*.config" -o -name "*.conf" 2>/dev/null
  ```
* Buscar backups olvidados (`.bak`, `.old`, `.tar.gz`, `.zip`):
  ```bash
  find / -type f -name "*.bak" -o -name "*.old" -o -name "*.tar.gz" -o -name "*.zip" 2>/dev/null | grep -v "/usr/"
  ```

### Buscar Patrones de Contraseña y Secretos en Texto Plano
* Buscar la palabra "password" en el directorio web de forma recursiva:
  ```bash
  grep -rnw '/var/www' -e 'password' -e 'passwd' -e 'key' -e 'db_password' 2>/dev/null
  ```
* Buscar credenciales en los archivos de configuración en `/etc/`:
  ```bash
  grep -rnw '/etc/' -e 'password' -e 'passwd' -e 'admin' 2>/dev/null
  ```

### Buscar Archivos con Permisos de Escritura del Usuario Actual
* Archivos fuera del directorio home que el usuario actual puede modificar (potencial secuestro de binarios o librerías):
  ```bash
  find / -writable ! -user $(whoami) -type f 2>/dev/null | grep -v -E "/proc|/sys|/dev|/run|/tmp"
  ```
