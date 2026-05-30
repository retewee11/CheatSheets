# Nmap Cheat Sheet
> **Guía rápida de comandos para el escaneo de puertos, detección de servicios, uso de scripts (NSE) y evasión de firewalls.**

---

## 🚀 Escaneos Esenciales

* **Escaneo rápido TCP (todos los puertos)**:
  Identifica puertos abiertos a alta velocidad exportando los resultados en formato grepable.
  ```bash
  nmap -p- --open -sS --min-rate 5000 -n -Pn <IP> -oG allPorts
  ```
* **Escaneo exhaustivo (Detección de versiones, S.O. y scripts básicos)**:
  Se ejecuta sobre los puertos específicos encontrados en el paso anterior.
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

## 🛠️ Scripts del Motor de Nmap (NSE)

Nmap incluye una amplia colección de scripts para enumeración avanzada y detección de vulnerabilidades.

* **Ejecutar una categoría completa de scripts (safe, vuln, discovery)**:
  ```bash
  nmap -p<PUERTOS> --script=vuln <IP>
  ```
* **Buscar scripts específicos por palabra clave**:
  ```bash
  locate .nse | grep "smb"
  ```
* **Scripts populares por protocolo**:
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

## 🛡️ Evasión de Firewalls y EDRs

Técnicas útiles en auditorías internas donde existen sistemas de detección de intrusos.

* **Fragmentación de paquetes (`-f`)**:
  Divide las cabeceras TCP en fragmentos pequeños para evadir filtros.
  ```bash
  nmap -f <IP>
  ```
* **Especificar MTU personalizado (debe ser múltiplo de 8)**:
  ```bash
  nmap --mtu 24 <IP>
  ```
* **Simular origen usando señuelos (Decoy - `-D`)**:
  Mezcla tu dirección IP real con IPs falsas en los registros del IDS.
  ```bash
  nmap -D 192.168.1.5,192.168.1.9,ME <IP>
  ```
* **Cambiar el puerto de origen (Source Port - `--source-port` o `-g`)**:
  Fuerza a Nmap a enviar paquetes desde un puerto comúnmente permitido (ej. DNS/53 o HTTP/80).
  ```bash
  nmap -g 53 <IP>
  ```
* **Establecer un retraso entre peticiones (Evitar picos de tráfico)**:
  ```bash
  nmap --scan-delay 5s <IP>
  ```
