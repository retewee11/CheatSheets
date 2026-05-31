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

* Fuzzing de cabeceras Host (VHosts):
  > [!NOTE]
  > Asocie previamente la IP al nombre base en `/etc/hosts` (e.g., `<IP> dominio.local`).
  ```bash
  ffuf -u http://dominio.local -H "Host: FUZZ.dominio.local" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -fs <TAMAÑO_RESPUESTA>
  ```
  *(Filtrar con `-fs` el tamaño de respuesta por defecto para descartar falsos positivos, e.g., `-fs 1234`)*

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

## 4. Filtros Comunes de ffuf

* `-mc`: Códigos de estado HTTP a mostrar (e.g., `-mc 200,301,302,403`).
* `-fc`: Códigos de estado HTTP a ocultar (e.g., `-fc 404,500`).
* `-fs`: Ocultar respuestas según tamaño exacto en bytes (esencial en VHosts).
* `-fl`: Ocultar respuestas por número de líneas.
* `-fr`: Ocultar respuestas según coincidencia con expresión regular.
