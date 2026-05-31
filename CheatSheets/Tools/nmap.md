# Nmap Cheat Sheet

Comandos esenciales de Nmap para el escaneo de puertos, descubrimiento de servicios, automatización con scripts (NSE) y evasión de firewalls.

---

## 1. Escaneos Esenciales

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
