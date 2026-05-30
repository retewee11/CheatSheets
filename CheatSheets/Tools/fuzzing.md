# Web Fuzzing Cheat Sheet (ffuf & Gobuster)
> **Guía rápida de comandos para el descubrimiento de directorios, archivos, hosts virtuales (VHosts) y parámetros mediante fuerza bruta.**

---

## 📂 Descubrimiento de Directorios y Archivos

### 🚀 Uso con Gobuster (Escritura rápida en Go)
* **Búsqueda básica de directorios**:
  ```bash
  gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
  ```
* **Búsqueda de archivos con extensiones específicas**:
  ```bash
  gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,txt,html,json
  ```
* **Omitir la validación de certificados SSL (`-k`)**:
  ```bash
  gobuster dir -u https://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k
  ```

### ⚡ Uso con ffuf (Altamente configurable e interactivo)
* **Fuzzing básico de directorios y extensiones**:
  ```bash
  ffuf -u http://<IP>/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e .php,.txt,.html -t 50
  ```
* **Fuzzing recursivo (sigue descubriendo subcarpetas de forma automática)**:
  ```bash
  ffuf -u http://<IP>/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -recursion -recursion-depth 2 -v
  ```

---

## 🪪 Descubrimiento de Subdominios y Hosts Virtuales (VHosts)

Útil para identificar aplicaciones web secundarias alojadas bajo la misma dirección IP.

* **Fuzzing de VHosts (cuando los subdominios no resuelven públicamente por DNS)**:
  > [!NOTE]
  > Agrega la IP al archivo `/etc/hosts` de tu máquina atacante antes de escanear (ej. `<IP> dominio.local`).
  ```bash
  ffuf -u http://dominio.local -H "Host: FUZZ.dominio.local" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -fs <TAMAÑO_RESPUESTA_A_FILTRAR>
  ```
  *(Reemplaza `<TAMAÑO_RESPUESTA_A_FILTRAR>` por el tamaño en bytes de la respuesta por defecto para descartar falsos positivos, ej: `-fs 1234`)*.

---

## 🔑 Fuzzing de Parámetros y Valores

### 1. Fuzzing de Parámetros GET
Encuentra variables ocultas en el código de la web (ej. `?page=`, `?file=`, `?cmd=`).
```bash
ffuf -u 'http://<IP>/index.php?FUZZ=test' -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt -fs <TAMAÑO_FILTRAR>
```

### 2. Fuerza Bruta en Formularios POST (Login, campos ocultos)
```bash
ffuf -u http://<IP>/login.php -X POST -d "username=admin&password=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -fr "Login failed"
```
*(El parámetro `-fr` filtra respuestas que contengan la expresión regular indicada, en este caso "Login failed")*.

---

## ⚙️ Filtros Comunes de ffuf

ffuf responde por defecto con todos los códigos HTTP. Es crucial filtrar la salida para evitar ruido.

* **`-mc` (Match Codes)**: Mostrar solo ciertos códigos de respuesta (ej. `-mc 200,301,302,403`).
* **`-fc` (Filter Codes)**: Ocultar ciertos códigos de respuesta (ej. `-fc 404,500`).
* **`-fs` (Filter Size)**: Ocultar respuestas de un tamaño específico en bytes. Muy útil en fuzzing de VHosts.
* **`-fl` (Filter Lines)**: Ocultar respuestas según el número de líneas.
* **`-fr` (Filter Regexp)**: Ocultar respuestas que contengan un texto específico.
