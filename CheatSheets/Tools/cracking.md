# Password Cracking Cheat Sheet

Comandos y configuración para descifrado de hashes offline utilizando Hashcat y John the Ripper.

---

## 1. Hashcat

### Sintaxis General
```bash
hashcat -m <MODO> -a 0 <HASH_FILE> <WORDLIST>
```

### Opciones Comunes
* `-a 0`: Ataque basado en diccionario (Straight).
* `-a 3`: Ataque basado en máscara (Fuerza bruta).
* `-r <REGLAS>`: Aplicación de reglas (e.g., `/usr/share/hashcat/rules/best64.rule`).

### Modos de Hash Frecuentes (`-m`)

| Modo | Tipo de Hash | Descripción / Contexto |
| :--- | :--- | :--- |
| **0** | MD5 | Hashes web heredados. |
| **1000** | NTLM | Contraseñas locales de Windows y Active Directory. |
| **1800** | sha512crypt | Contraseñas del archivo `/etc/shadow` en Linux moderno. |
| **13100** | Kerberoast (TGS-REP) | Tickets de cuentas de servicio de Active Directory. |
| **18200** | AS-REP Roasting | Tickets TGT de usuarios AD sin pre-autenticación. |
| **5600** | NetNTLMv2 | Hashes capturados en red (e.g., Responder). |
| **22000** | WPA-PBKDF2 | Capturas WPA2 de redes inalámbricas (PMKID / EAPOL). |

---

## 2. John the Ripper

### Sintaxis General
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### Especificar Formato
```bash
john --format=NT --wordlist=rockyou.txt hash.txt
```

### Mostrar Contraseñas Descifradas
```bash
john --show hash.txt
```

---

## 3. Extractores de Hashes (Herramientas auxiliares de John)

Obtención del hash criptográfico a partir de archivos cifrados para su descifrado posterior.

### Archivos Comprimidos
* **ZIP**:
  ```bash
  zip2john archivo.zip > zip.hash
  john --wordlist=rockyou.txt zip.hash
  ```
* **RAR**:
  ```bash
  rar2john archivo.rar > rar.hash
  john --wordlist=rockyou.txt rar.hash
  ```

### Claves Privadas e Identidades
* **SSH (id_rsa)**:
  ```bash
  ssh2john id_rsa > ssh.hash
  john --wordlist=rockyou.txt ssh.hash
  ```
* **Bases de datos KeePass (.kdbx)**:
  ```bash
  keepass2john base.kdbx > keepass.hash
  john --wordlist=rockyou.txt keepass.hash
  ```

### Documentos
* **PDF**:
  ```bash
  pdf2john documento.pdf > pdf.hash
  john --wordlist=rockyou.txt pdf.hash
  ```
