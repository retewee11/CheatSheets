# Cybersecurity Cheat Sheets

Repositorio de referencia técnica y comandos rápidos para auditoría de sistemas, pentesting e investigación forense.

---

## Índice de Guías Metodológicas

| Categoría | Recurso | Descripción |
| :--- | :--- | :--- |
| **Active Directory** | [Pentesting AD](CheatSheets/Pentesting_AD.md) | Enumeración, explotación y movimiento lateral en dominios. |
| **Linux** | [Pentesting Linux](CheatSheets/Pentesting_Linux.md) | Reconocimiento local, escalada de privilegios y persistencia. |
| **Windows** | [Pentesting Windows](CheatSheets/Pentesting_Windows.md) | Técnicas locales, evasión y escalada de privilegios. |
| **Forense** | [Volatility 3 para Windows](CheatSheets/volatility3_windows.md) | Análisis de memoria RAM, plugins de registro y forense digital. |
| **Defensivo / SIEM** | [Wazuh](CheatSheets/wazuh.md) | Despliegue de servidores, agentes y análisis de vulnerabilidades. |

---

## Índice de Herramientas

| Herramienta / Técnica | Recurso | Descripción |
| :--- | :--- | :--- |
| **Nmap** | [Nmap Cheat Sheet](CheatSheets/Tools/nmap.md) | Escaneo de puertos y descubrimiento de servicios. |
| **Fuzzing Web** | [Web Fuzzing (ffuf/Gobuster)](CheatSheets/Tools/fuzzing.md) | Fuerza bruta de directorios, subdominios y parámetros. |
| **NetExec** | [NetExec](CheatSheets/Tools/netexec.md) | Auditoría de servicios SMB, LDAP, WinRM, SSH y MSSQL. |
| **Impacket** | [Impacket Suite](CheatSheets/Tools/impacket.md) | Movimiento lateral, extracción de hashes y operaciones Kerberos. |
| **Chisel** | [Chisel](CheatSheets/Tools/chisel.md) | Configuración de túneles TCP y proxy SOCKS para pivoting. |
| **Pivoting Avanzado** | [Redirección de Puertos](CheatSheets/Tools/port_redirection.md) | Enrutamiento y redirecciones con Ligolo-ng y Socat. |
| **Cracking** | [Password Cracking](CheatSheets/Tools/cracking.md) | Descifrado de hashes con Hashcat y John the Ripper. |
| **Sqlmap** | [Sqlmap Cheat Sheet](CheatSheets/Tools/sqlmap.md) | Detección y explotación de inyecciones SQL (SQLi). |
| **Mimikatz** | [Mimikatz Cheat Sheet](CheatSheets/Tools/mimikatz.md) | Extracción de credenciales de memoria y tokens de seguridad. |
| **Msfvenom** | [Msfvenom & Metasploit](CheatSheets/Tools/msfvenom.md) | Generación de payloads y configuración de listeners. |

---

## Descripción General

Espacio de referencia diseñado para centralizar comandos, payloads y flujos de trabajo en auditorías de seguridad y entornos de laboratorio (Hack The Box, TryHackMe).

* **Enfoque Práctico**: Comandos directos listos para su uso.
* **Formato Limpio**: Sin texto superfluo para agilizar la copia en consola.
* **Anotaciones OPSEC**: Notas sobre nivel de ruido y firmas de detección.
