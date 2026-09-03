# SMB & RPC Enumeration Cheat Sheet

Comandos esenciales para el reconocimiento y enumeración de servicios SMB (puertos 139/445) y RPC (puerto 135) en auditorías de seguridad y examen eJPT.

---

## 1. enum4linux / enum4linux-ng

Herramienta automatizada para la extracción de información en sistemas Windows/Samba.

```bash
# Enumeración completa (usuarios, shares, políticas de contraseñas, grupos)
enum4linux -a <IP_OBJETIVO>

# Enumeración con credenciales conocidas
enum4linux -a -u "<USUARIO>" -p "<PASSWORD>" <IP_OBJETIVO>

# enum4linux-ng (versión moderna y más rápida)
enum4linux-ng -A <IP_OBJETIVO>
```

---

## 2. smbclient

Cliente de línea de comandos para listar e interactuar con recursos compartidos (shares) SMB.

### Listar Recursos Compartidos (Shares)
```bash
# Conexión anónima / Null Session
smbclient -L //<IP_OBJETIVO> -N

# Listar recursos con usuario conocido
smbclient -L //<IP_OBJETIVO> -U "<USUARIO>"
```

### Conectarse a un recurso compartido
```bash
# Conexión anónima a un share específico (ej. SharedData)
smbclient //<IP_OBJETIVO>/SharedData -N

# Conexión con credenciales
smbclient //<IP_OBJETIVO>/SharedData -U "<USUARIO>"%"<PASSWORD>"
```

### Comandos útiles dentro del prompt de `smbclient`
* `ls` : Listar archivos del directorio actual.
* `get <archivo>` : Descargar un archivo al equipo local.
* `put <archivo>` : Subir un archivo local al recurso compartido.
* `prompt off` : Desactivar la confirmación interactiva para descargas múltiples.
* `mget *` : Descargar todos los archivos del directorio actual.

---

## 3. smbmap

Herramienta rápida para auditar permisos de lectura/escritura en recursos compartidos SMB.

```bash
# Auditar permisos con sesión nula / anónima
smbmap -H <IP_OBJETIVO>

# Auditar permisos con credenciales
smbmap -H <IP_OBJETIVO> -u "<USUARIO>" -p "<PASSWORD>"

# Listar recursivamente el contenido de un recurso compartido
smbmap -H <IP_OBJETIVO> -u "<USUARIO>" -p "<PASSWORD>" -r "SharedData"

# Buscar un patrón de nombre de archivo específico
smbmap -H <IP_OBJETIVO> -u "<USUARIO>" -p "<PASSWORD>" -F "config"

# Subir / Descargar archivos
smbmap -H <IP_OBJETIVO> -u "<USUARIO>" -p "<PASSWORD>" --download "SharedData/passwords.txt"
```

---

## 4. rpcclient

Cliente para interactuar con la interfaz MS-RPC a través de sesiones nulas o autenticadas.

```bash
# Iniciar sesión nula interactiva
rpcclient -U "" -N <IP_OBJETIVO>

# Iniciar sesión autenticada
rpcclient -U "<USUARIO>" <IP_OBJETIVO>
```

### Comandos dentro de `rpcclient`
```text
rpcclient $> srvinfo                 # Información del SO y versión del servidor
rpcclient $> enumdomusers            # Listar usuarios del dominio/sistema local
rpcclient $> enumdomgroups           # Listar grupos del dominio/sistema local
rpcclient $> querydispinfo           # Mostrar información detallada de cuentas de usuario
rpcclient $> queryuser <RID/USERNAME> # Consultar información específica de un usuario (ej. SID, grupos)
rpcclient $> netshareenumall         # Listar todos los recursos compartidos RPC
```

---

## 5. Nmap SMB Scripts

Nmap incluye scripts NSE específicos para auditar vulnerabilidades SMB comunes.

```bash
# Enumeración de usuarios y recursos compartidos
nmap --script smb-enum-shares,smb-enum-users -p 445 <IP_OBJETIVO>

# Chequeo de vulnerabilidades SMB críticas (ej. MS17-010, EternalBlue)
nmap --script smb-vuln-ms17-010,smb-vuln-ms08-067 -p 445 <IP_OBJETIVO>

# Chequeo de sesión nula / Guest
nmap --script smb-security-mode,smb2-security-mode -p 445 <IP_OBJETIVO>
```
