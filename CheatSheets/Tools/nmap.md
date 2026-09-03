# Nmap Cheat Sheet

Comandos esenciales de Nmap para el escaneo de puertos, descubrimiento de servicios, automatización con scripts (NSE) y evasión de firewalls.

---

## 1. Descubrimiento de Hosts (Host Discovery / Ping Sweep)

Identificar qué equipos están encendidos en una subred sin escanear puertos:

* **Ping Sweep básico de subred (`-sn`)**:
  Envía sondas ICMP Echo y peticiones TCP/ARP para encontrar hosts activos:
  ```bash
  nmap -sn 192.168.1.0/24
  ```
* **ARP Ping Sweep (Ultra rápido en red local LAN / Capa 2)**:
  Infallible en redes locales aunque el firewall del objetivo bloquee ICMP/Ping:
  ```bash
  nmap -sn -PR 192.168.1.0/24
  ```
* **Descubrimiento por SYN/ACK a puertos comunes (`-PS` / `-PA`)**:
  Útil cuando ICMP está bloqueado pero queremos probar puertos clave (22, 80, 445):
  ```bash
  nmap -sn -PS22,80,443,445 192.168.1.0/24
  ```
* **Herramientas de descubrimiento rápido en red local**:
  ```bash
  arp-scan --interface=eth0 192.168.1.0/24
  netdiscover -r 192.168.1.0/24
  ```

---

## 2. Escaneos Esenciales de Puertos

* **Escaneo rápido TCP (todos los puertos)**:
  Identificación rápida de puertos abiertos y guardado en formato grepable (`.gnmap`):
  ```bash
  nmap -p- --open -sS --min-rate 5000 -n -Pn <IP> -oG allPorts
  ```
* **Escaneo de servicios y scripts de puertos específicos**:
  Ejecución de versiones y scripts por defecto en los puertos detectados:
  ```bash
  nmap -p<PUERTOS> -sCV <IP> -oN targeted
  ```
* **Escaneo rápido de puertos UDP populares**:
  ```bash
  nmap -sU --top-ports 100 <IP>
  ```
* **Escaneo completo UDP (los 65535 puertos - Muy lento)**:
  ```bash
  nmap -sU -p- --min-rate 2000 -Pn <IP> -oN targeted_udp
  ```

---

## 2. Scripts del Motor de Nmap (NSE)

* **Ejecutar una categoría de scripts (vuln, discovery, safe)**:
  ```bash
  nmap -p<PUERTOS> --script=vuln <IP>
  ```
* **Buscar scripts locales**:
  ```bash
  locate .nse | grep "smb"
  ```
* **Scripts comunes por servicio**:
  * **SMB (445)**:
    ```bash
    nmap -p 445 --script="smb-vuln-* or smb-enum-*" <IP>
    ```
  * **HTTP (80, 443)**:
    ```bash
    nmap -p 80 --script="http-enum or http-vuln-*" <IP>
    ```
  * **FTP (21)**:
    ```bash
    nmap -p 21 --script="ftp-anon or ftp-syst" <IP>
    ```
  * **LDAP (389)**:
    ```bash
    nmap -p 389 --script="ldap-*" <IP>
    ```

---

## 3. Evasión de Firewalls y EDRs

* **Fragmentación de paquetes (`-f`)**:
  Fragmentar cabeceras TCP para dificultar la reconstrucción del flujo de escaneo:
  ```bash
  nmap -f <IP>
  ```
* **MTU personalizado (debe ser múltiplo de 8)**:
  ```bash
  nmap --mtu 24 <IP>
  ```
* **Simulación de direcciones origen (Decoy - `-D`)**:
  Ocultar la IP del escáner mezclándola con direcciones origen señuelo:
  ```bash
  nmap -D 192.168.1.5,192.168.1.9,ME <IP>
  ```
* **Cambiar el puerto origen (`--source-port` / `-g`)**:
  Simular tráfico desde puertos permitidos por firewalls (e.g. DNS/53 o HTTP/80):
  ```bash
  nmap -g 53 <IP>
  ```
* **Retraso de escaneo (Scan Delay)**:
  ```bash
  nmap --scan-delay 5s <IP>
  ```
