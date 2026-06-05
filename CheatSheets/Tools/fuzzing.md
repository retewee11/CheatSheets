# Web Fuzzing Cheat Sheet (ffuf & Gobuster)

Comandos para el descubrimiento de directorios, archivos, subdominios, hosts virtuales (VHosts) y parámetros web mediante fuerza bruta.

---

## 1. Descubrimiento de Directorios y Archivos

### Gobuster
* Búsqueda estándar de directorios:
  ```bash
  gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
  ```
* Búsqueda filtrando extensiones de archivo:
  ```bash
  gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,txt,html,json
  ```
* Omitir validación de certificados SSL/TLS:
  ```bash
  gobuster dir -u https://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k
  ```

### ffuf
* Fuzzing básico de directorios y extensiones:
  ```bash
  ffuf -u http://<IP>/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e .php,.txt,.html -t 50
  ```
* Fuzzing recursivo:
  ```bash
  ffuf -u http://<IP>/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -recursion -recursion-depth 2 -v
  ```

---

## 2. Descubrimiento de Subdominios y Hosts Virtuales (VHosts)

### ffuf
* Fuzzing de cabeceras Host (VHosts):
  > [!NOTE]
  > Asocie previamente la IP al nombre base en `/etc/hosts` (e.g., `<IP> dominio.local`).
  ```bash
  ffuf -u http://dominio.local -H "Host: FUZZ.dominio.local" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -fs <TAMAÑO_RESPUESTA>
  ```
  *(Filtrar con `-fs` el tamaño de respuesta por defecto para descartar falsos positivos, e.g., `-fs 1234`)*

### wfuzz
* Enumeración de subdominios DNS (directa):
  ```bash
  wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u "http://FUZZ.dominio.local" --hc 400,404
  ```
* Fuzzing de cabeceras Host (VHosts):
  ```bash
  wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.dominio.local" -u http://dominio.local --hc 404
  ```

---

## 3. Fuzzing de Parámetros y Valores

### Parámetros GET
Búsqueda de variables en la URL:
```bash
ffuf -u 'http://<IP>/index.php?FUZZ=test' -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt -fs <TAMAÑO_RESPUESTA>
```

### Fuerza Bruta en Formularios POST (Campos y Login)
```bash
ffuf -u http://<IP>/login.php -X POST -d "username=admin&password=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -fr "Login failed"
```
*(Filtra respuestas que coincidan con la expresión regular del parámetro `-fr`, e.g., "Login failed")*

---

## 4. Ignorar Respuestas y Filtrado (Ocultar Falsos Positivos)

### ffuf
* `-mc`: Mostrar solo códigos de estado específicos (e.g., `-mc 200,301,403`).
* `-fc`: Ocultar códigos de estado específicos (e.g., `-fc 404,500,302`).
* `-fs`: Ocultar respuestas por tamaño en bytes (e.g., `-fs 1234`).
* `-fl`: Ocultar respuestas por número de líneas (e.g., `-fl 10`).
* `-fw`: Ocultar respuestas por número de palabras (e.g., `-fw 15`).
* `-fr`: Ocultar respuestas que coincidan con una expresión regular (e.g., `-fr "Error de conexión"`).

### Gobuster
* `-b` o `--status-codes-blacklist`: Ocultar códigos de estado HTTP (e.g., `-b "404,403,302"`).
* `--exclude-length`: Ocultar respuestas de un tamaño en bytes exacto (e.g., `--exclude-length 1234,0`).
* `--exclude-pattern`: Ocultar respuestas que contengan un patrón de texto en la respuesta.

### wfuzz
* `--hc`: Ocultar códigos de estado HTTP (e.g., `--hc 404,500,403`).
* `--hs`: Ocultar respuestas por expresión regular/texto en el contenido (e.g., `--hs "Page not found"`).
* `--hl`: Ocultar respuestas por número de líneas (e.g., `--hl 24`).
* `--hw`: Ocultar respuestas por número de palabras (e.g., `--hw 105`).
* `--hh`: Ocultar respuestas por número de caracteres / tamaño en bytes (e.g., `--hh 1234`).
