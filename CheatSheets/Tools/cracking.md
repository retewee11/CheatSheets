# Password Cracking Cheat Sheet (Hashcat & John)
> **Guía rápida para descifrar hashes de contraseñas, recuperar claves de archivos protegidos y configurar ataques con Hashcat y John the Ripper.**

---

## ⚡ 1. Uso de Hashcat (Acelerado por GPU)

Hashcat es idóneo para descifrar hashes de forma masiva aprovechando la potencia de tarjetas gráficas.

* **Comando general**:
  ```bash
  hashcat -m <MODO> -a 0 <HASH_FILE> <WORDLIST>
  ```
* **Opciones y reglas comunes**:
  * `-a 0`: Ataque de diccionario (Straight).
  * `-a 3`: Ataque de fuerza bruta de máscara (Mask).
  * `-r <REGLAS>`: Aplicar variaciones a la wordlist (ej. `/usr/share/hashcat/rules/best64.rule`).

### 🛠️ Modos de Hash Frecuentes (`-m`)

| Modo | Tipo de Hash | Descripción / Origen |
| :--- | :--- | :--- |
| **0** | MD5 | Hashes web heredados. |
| **1000** | NTLM | Contraseñas de usuario locales de Windows e identidades AD. |
| **1800** | sha512crypt | Contraseñas de usuarios locales de Linux modernos (`/etc/shadow`). |
| **13100** | Kerberoast (TGS-REP) | Cuentas de servicio de Active Directory. |
| **18200** | AS-REP Roasting | Usuarios AD con pre-autenticación desactivada. |
| **5600** | NetNTLMv2 | Hashes capturados de red mediante Responder o NTLM Relay. |
| **22000** | WPA-PBKDF2 | Capturas de redes WiFi (WPA2 PMKID / EAPOL). |

---

## 🔑 2. Uso de John the Ripper (Versatilidad por CPU)

John es excelente para auditar rápidamente archivos hash individuales e identificar formatos de forma automática.

* **Comando básico**:
  ```bash
  john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
  ```
* **Especificar un formato exacto**:
  ```bash
  john --format=NT --wordlist=rockyou.txt hash.txt
  ```
* **Visualizar las contraseñas descifradas**:
  ```bash
  john --show hash.txt
  ```

---

## 🗄️ 3. Extractores de Hashes (John's Suite)

Antes de crackear un contenedor de datos protegido (ZIP, PDF, SSH), debes extraer su firma criptográfica en formato de hash usando los scripts auxiliares de John.

* **Archivos comprimidos ZIP**:
  ```bash
  zip2john archivo.zip > zip.hash
  john --wordlist=rockyou.txt zip.hash
  ```
* **Archivos comprimidos RAR**:
  ```bash
  rar2john archivo.rar > rar.hash
  john --wordlist=rockyou.txt rar.hash
  ```
* **Claves privadas SSH (`id_rsa` protegidas con clave)**:
  ```bash
  ssh2john id_rsa > ssh.hash
  john --wordlist=rockyou.txt ssh.hash
  ```
* **Bases de datos KeePass (`.kdbx`)**:
  ```bash
  keepass2john base.kdbx > keepass.hash
  john --wordlist=rockyou.txt keepass.hash
  ```
* **Documentos PDF**:
  ```bash
  pdf2john documento.pdf > pdf.hash
  john --wordlist=rockyou.txt pdf.hash
  ```
