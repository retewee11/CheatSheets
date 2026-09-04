# Guía Detallada de Enumeración SMB & RPC

Explicación completa y detallada de **enum4linux**, **smbclient** y **smbmap** (junto con **rpcclient**) para auditorías de seguridad en entornos Windows / Samba y examen eJPT v2.

---

## 1. Flujo de Trabajo Recomendado en Auditorías SMB

Cuando descubres el puerto **445/TCP** o **139/TCP** abierto en una máquina objetivo:

```
[1. Auditar Permisos Rápidos] ----> smbmap -H <IP> (Comprobar permisos READ/WRITE)
                                          |
[2. Extraer Usuarios y Datos] ----> enum4linux -a <IP> (Extraer usuarios, grupos y política de pass)
                                          |
[3. Inspeccionar y Descargar] ----> smbclient //<IP>/<SHARE> -N (Conectarse y descargar archivos)
                                          |
[4. Consultas RPC Avanzadas] ----> rpcclient -U "" -N <IP> (Consultar SIDs, RIDs y detalles)
```

---

## 2. enum4linux / enum4linux-ng

### ¿Qué hace enum4linux?
**enum4linux** es una herramienta automatizada de reconocimientos que ejecuta múltiples consultas de red (vía NetBIOS, RPC, SMB y LDAP) para extraer información crítica de un servidor Windows o Samba sin necesidad de explotar ninguna vulnerabilidad.

### Explicación de Flags y Parámetros

| Flag | Descripción Explicada | Ejemplo de uso |
| :--- | :--- | :--- |
| `-a` | **Hacer TODO** (All checks). Equivale a `-U -S -G -P -o -i -r`. | `enum4linux -a 10.10.10.50` |
| `-U` | Listar **usuarios** del dominio o sistema local (`enumdomusers`). | `enum4linux -U 10.10.10.50` |
| `-S` | Listar **recursos compartidos (shares)** (`netshareenumall`). | `enum4linux -S 10.10.10.50` |
| `-G` | Listar **grupos** y sus miembros (`enumdomgroups`). | `enum4linux -G 10.10.10.50` |
| `-P` | Mostrar la **política de contraseñas** (longitud mínima, bloqueo de cuenta). | `enum4linux -P 10.10.10.50` |
| `-o` | Información del **Sistema Operativo** y versión de Samba (`srvinfo`). | `enum4linux -o 10.10.10.50` |
| `-i` | Información de adaptadores de red y nombre del Dominio/Workgroup. | `enum4linux -i 10.10.10.50` |
| `-r` | Enumerar usuarios mediante **RID Cycling** (fuerza bruta de identificadores de 500 a 1000). | `enum4linux -r 10.10.10.50` |
| `-u <USER>` | Especificar un **usuario** para autenticación. | `enum4linux -u "admin" -p "pass" -a 10.10.10.50` |
| `-p <PASS>` | Especificar la **contraseña** del usuario. | `enum4linux -u "guest" -p "" -a 10.10.10.50` |

### Ejemplo Práctico:
```bash
# Enumeración completa anónima (Null Session)
enum4linux -a 10.10.10.50

# enum4linux-ng (Versión moderna en Python 3, más rápida y limpia)
enum4linux-ng -A 10.10.10.50 -oA enum_results
```
* **Explicación**: `-oA enum_results` en `enum4linux-ng` guardará los hallazgos automáticamente en formatos JSON y YAML para análisis posterior.

---

## 3. smbclient

### ¿Qué hace smbclient?
**smbclient** es la herramienta nativa en Linux que funciona exactamente como un **cliente FTP pero para el protocolo SMB**. Te permite listar directorios compartidos en servidores remotos y conectarte a ellos de forma interactiva para descargar o subir archivos.

### Explicación de Flags y Parámetros

| Flag | Descripción Explicada | Ejemplo de uso |
| :--- | :--- | :--- |
| `-L <IP>` | **Listar** todos los recursos compartidos disponibles en el servidor objetivo. | `smbclient -L //10.10.10.50 -N` |
| `-N` | **No pedir contraseña** (Fuerza el intento de sesión nula o anónima). | `smbclient -L //10.10.10.50 -N` |
| `-U <USUARIO>` | Especificar el nombre de **usuario** para conectarse. | `smbclient //10.10.10.50/Shared -U "admin"` |
| `-U <USER>%<PASS>` | Especificar usuario y contraseña en una sola línea. | `smbclient //10.10.10.50/Shared -U "admin"%"Pass123"` |
| `-W <DOMINIO>` | Especificar el nombre del **Workgroup o Dominio** de Active Directory. | `smbclient -L //10.10.10.50 -U "user" -W "CORP"` |
| `--option="..."` | Pasar opciones de configuración de Samba (útil para servidores viejos). | `--option="client min protocol=NT1"` |

### Conexión Interactiva y Comandos Internos
Una vez conectado a un recurso compartido (ej. `smbclient //10.10.10.50/SharedDocs -N`), entras en la consola `smb: \>`.

#### Comandos Clave dentro de `smbclient`:
* `ls` / `dir` : Listar los archivos y carpetas del recurso compartido actual.
* `cd <directorio>` : Cambiar de directorio dentro del recurso compartido.
* `get <archivo>` : **Descargar** un archivo remoto a tu máquina local actual.
* `put <archivo_local>` : **Subir** un archivo local al recurso compartido.
* `prompt off` : Desactiva la confirmación interactiva `(y/n)` al descargar múltiples archivos.
* `recurse ON` : Activa la descarga recursiva en subcarpetas.
* `mget *` : **Descargar TODOS los archivos** del directorio actual (y subcarpetas si `recurse ON` está activo).

#### Ejemplo de Descarga Masiva (Exfiltración rápida de archivos):
```text
smb: \> prompt off
smb: \> recurse ON
smb: \> mget *
```
*(Esto descargará todo el contenido del recurso compartido a tu carpeta actual en Kali)*.

---

## 4. smbmap

### ¿Qué hace smbmap?
**smbmap** es una de las herramientas más eficientes en auditorías SMB. Su objetivo principal es **mapear visualmente los permisos de acceso** (`READ ONLY`, `READ, WRITE`, `NO ACCESS`) en todos los recursos compartidos del servidor y buscar archivos sensibles de forma automática.

### Explicación de Flags y Parámetros

| Flag | Descripción Explicada | Ejemplo de uso |
| :--- | :--- | :--- |
| `-H <IP>` | Especifica la **IP objetivo** o segmento de red CIDR (ej. `192.168.1.0/24`). | `smbmap -H 10.10.10.50` |
| `-u <USER>` | Especifica el **usuario** (usar `""` o `'guest'` para anónimo). | `smbmap -H 10.10.10.50 -u ""` |
| `-p <PASS>` | Especifica la **contraseña** (usar `""` para contraseña vacía). | `smbmap -H 10.10.10.50 -u "" -p ""` |
| `-d <DOMINIO>` | Especificar **Dominio** o Workgroup. | `smbmap -H 10.10.10.50 -u "user" -p "pass" -d "WORKGROUP"` |
| `-r <SHARE>` | **Listar recursivamente** el contenido de un recurso compartido. | `smbmap -H 10.10.10.50 -u "" -p "" -r "SharedDocs"` |
| `-F <PATRÓN>` | **Buscar un archivo** por su nombre o extensión en todos los shares. | `smbmap -H 10.10.10.50 -u "" -p "" -F "pass"` |
| `--download <RUTA>` | **Descargar un archivo remoto** directamente. | `smbmap -H 10.10.10.50 -u "admin" -p "pass" --download "SharedDocs/config.txt"` |
| `--upload <LOCAL> <REMOTO>` | **Subir un archivo local** a un recurso con permiso `READ, WRITE`. | `smbmap -H 10.10.10.50 -u "admin" -p "pass" --upload "nc.exe" "SharedDocs/nc.exe"` |
| `-x <COMANDO>` | **Ejecutar comandos remotos** (requiere privilegios administrativos en SMB). | `smbmap -H 10.10.10.50 -u "Admin" -p "Pass" -x "whoami"` |

---

## 5. rpcclient (Consultas de Bajo Nivel MS-RPC)

Cliente interactivo para comunicarse con la interfaz RPC de Windows.

### Conexión:
```bash
# Sesión nula
rpcclient -U "" -N 10.10.10.50

# Con usuario
rpcclient -U "usuario" 10.10.10.50
```

### Comandos más usados dentro de `rpcclient $>`:
* `srvinfo` : Muestra información del servidor y versión del SO.
* `enumdomusers` : Lista todos los usuarios del dominio/sistema con su RID.
* `enumdomgroups` : Lista los grupos de usuarios.
* `querydispinfo` : Muestra información extendida de cuentas (comentarios, descripciones).
* `queryuser <RID>` : Muestra información detallada del usuario con ese RID (ej. `queryuser 0x1f4` para el usuario 500/Administrator).
* `lookupnames <USERNAME>` : Devuelve el SID/RID exacto de un nombre de usuario.

---

## 6. Solución a Errores Comunes en SMB

### Error `NT_STATUS_CONNECTION_RESET` o `NT_STATUS_REVISION_MISMATCH`
Ocurre cuando el servidor SMB es muy antiguo (Windows Server 2003 / XP / Samba viejo) y utiliza **SMBv1**, el cual está desactivado por defecto en clientes Linux modernos.

* **Solución en `smbclient`**:
  ```bash
  smbclient -L //10.10.10.50 -N --option="client min protocol=NT1"
  ```
* **Solución en `smbmap`**:
  ```bash
  smbmap -H 10.10.10.50 --option="client min protocol=NT1"
  ```
