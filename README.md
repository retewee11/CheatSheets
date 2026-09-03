# Cybersecurity Cheat Sheets

Repositorio de referencia técnica y comandos rápidos para auditoría de sistemas, pentesting e investigación forense.

---

## Índice de Guías Metodológicas

| Categoría | Recurso | Descripción |
| :--- | :--- | :--- |
| **Active Directory** | [Pentesting AD](CheatSheets/Pentesting_AD.md) | Enumeración, explotación y movimiento lateral en dominios. |
| **Linux** | [Pentesting Linux](CheatSheets/Pentesting_Linux.md) | Reconocimiento local, escalada de privilegios y persistencia. |
| **Linux (PrivEsc eJPT)** | [Escalada Linux eJPT](CheatSheets/privesc_linux_ejpt.md) | SUID, Sudo, permisos de archivos y LinPEAS (eJPT v2). |
| **Linux (Archivos)** | [Archivos Sensibles Linux](CheatSheets/Linux_Sensitive_Files.md) | Catálogo de archivos críticamente sensibles y rutas de interés en auditorías. |
| **Windows** | [Pentesting Windows](CheatSheets/Pentesting_Windows.md) | Técnicas locales, evasión y escalada de privilegios. |
| **Windows (PrivEsc eJPT)** | [Escalada Windows eJPT](CheatSheets/privesc_windows_ejpt.md) | Token Impersonation, Unquoted Paths, AlwaysInstallElevated y WinPEAS (eJPT v2). |
| **Web (Vulnerabilidades eJPT)** | [Vulnerabilidades Web eJPT](CheatSheets/web_vulnerabilities_ejpt.md) | Explotación manual de LFI, RFI, Command Injection y bypasses de login. |
| **Web (Payloads y Pruebas)** | [Web Testing & Payloads](CheatSheets/Web_Testing_Payloads.md) | Catálogo de payloads para SQLi, XSS, Command Injection, LFI, Subida de archivos, SSRF e IDOR. |
| **Forense** | [Volatility 3 para Windows](CheatSheets/volatility3_windows.md) | Análisis de memoria RAM, plugins de registro y forense digital. |
| **Defensivo / SIEM** | [Wazuh](CheatSheets/wazuh.md) | Despliegue de servidores, agentes y análisis de vulnerabilidades. |

---

## Índice de Herramientas

| Herramienta / Técnica | Recurso | Descripción |
| :--- | :--- | :--- |
| **Nmap** | [Nmap Cheat Sheet](CheatSheets/Tools/nmap.md) | Escaneo de puertos y descubrimiento de servicios. |
| **Fuzzing Web** | [Web Fuzzing (ffuf/Gobuster)](CheatSheets/Tools/fuzzing.md) | Fuerza bruta de directorios, subdominios y parámetros. |
| **Metasploit Framework** | [Metasploit & Meterpreter](CheatSheets/Tools/metasploit.md) | Uso de msfconsole, comandos de Meterpreter y pivoting (autoroute/socks_proxy). |
| **Msfvenom** | [Msfvenom Payloads](CheatSheets/Tools/msfvenom.md) | Generación de payloads binarios y web por plataforma. |
| **Transferencia de Archivos** | [Transferencia de Archivos](CheatSheets/Tools/file_transfers.md) | Métodos de descarga en Windows (certutil, PS, SMB) y Linux (wget, curl, nc). |
| **Reverse Shells** | [Reverse Shells & Payloads](CheatSheets/Tools/reverse_shells.md) | One-liners en Bash, Python, PHP, PowerShell y bypasses en Base64. |
| **NetExec** | [NetExec](CheatSheets/Tools/netexec.md) | Auditoría de servicios SMB, LDAP, WinRM, SSH y MSSQL. |
| **Enumeración SMB / RPC** | [SMB & RPC Enumeration](CheatSheets/Tools/enum_smb.md) | Auditoría SMB/RPC con enum4linux, smbclient, smbmap y rpcclient. |
| **Impacket** | [Impacket Suite](CheatSheets/Tools/impacket.md) | Movimiento lateral, extracción de hashes y operaciones Kerberos. |
| **Hydra** | [Hydra Brute Force](CheatSheets/Tools/hydra.md) | Ataques de fuerza bruta a servicios de red (SSH, FTP, SMB, RDP, HTTP-FORM). |
| **Searchsploit** | [Searchsploit](CheatSheets/Tools/searchsploit.md) | Búsqueda e inspección offline de vulnerabilidades y exploits de ExploitDB. |
| **Escáneres Web** | [Nikto & WPScan](CheatSheets/Tools/web_scanners.md) | Escaneo automático de servidores web y sitios WordPress. |
| **Netcat & Ncat** | [Netcat / Ncat](CheatSheets/Tools/netcat.md) | Reverse shells, listeners, transferencias de archivos y estabilización de TTY. |
| **Chisel** | [Chisel](CheatSheets/Tools/chisel.md) | Configuración de túneles TCP y proxy SOCKS para pivoting. |
| **Pivoting Avanzado** | [Redirección de Puertos](CheatSheets/Tools/port_redirection.md) | Enrutamiento y redirecciones con Ligolo-ng y Socat. |
| **Cracking** | [Password Cracking](CheatSheets/Tools/cracking.md) | Descifrado de hashes con Hashcat y John the Ripper. |
| **Sqlmap** | [Sqlmap Cheat Sheet](CheatSheets/Tools/sqlmap.md) | Detección y explotación de inyecciones SQL (SQLi). |
| **Mimikatz** | [Mimikatz Cheat Sheet](CheatSheets/Tools/mimikatz.md) | Extracción de credenciales de memoria y tokens de seguridad. |

---

## Descripción General

Espacio de referencia diseñado para centralizar comandos, payloads y flujos de trabajo en auditorías de seguridad y entornos de laboratorio (Hack The Box, TryHackMe).

* **Enfoque Práctico**: Comandos directos listos para su uso.
* **Formato Limpio**: Sin texto superfluo para agilizar la copia en consola.
* **Anotaciones OPSEC**: Notas sobre nivel de ruido y firmas de detección.
