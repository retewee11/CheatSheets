# Volatility 3 Cheat Sheet para Windows
> **Guía de comandos y referencia de plugins para el análisis forense de memoria RAM en sistemas operativos Windows utilizando Volatility 3.**

---

## 🚀 Uso Básico de Volatility 3

* **Comando general para ejecutar un plugin**:
  ```bash
  python3 vol.py -f <memoria.raw> <plugin>
  ```
* **Obtener información básica de la imagen de memoria (S.O., versión, arquitectura)**:
  ```bash
  python3 vol.py -f <memoria.raw> windows.info
  ```

---

## 🛠️ Tabla de Referencia de Plugins para Windows

| Categoría | Plugin | Descripción |
| :--- | :--- | :--- |
| **Información General** | `windows.info` | Información básica de la imagen de memoria analizada. |
| **Procesos** | `windows.pslist` | Lista de procesos activos (según las estructuras activas). |
| | `windows.pstree` | Muestra la jerarquía de procesos en formato de árbol. |
| | `windows.psscan` | Escaneo en busca de procesos ocultos o ya terminados. |
| | `windows.cmdline` | Muestra los argumentos y línea de comandos de inicio de cada proceso. |
| | `windows.getsid` | Muestra los identificadores de seguridad (SID) asociados a cada proceso. |
| | `windows.privs` | Enumera los privilegios activos e inactivos por proceso. |
| **Módulos y DLLs** | `windows.modules` | Módulos cargados en el espacio de memoria del Kernel. |
| | `windows.dlllist` | Lista de DLLs cargadas en el espacio de memoria de cada proceso. |
| | `windows.ldrmodules` | Compara DLLs en PEB y listas de carga para detectar inyecciones/desalineaciones. |
| **Red** | `windows.netscan` | Escanea y lista conexiones de red activas, puertos escuchando y sockets. |
| | `windows.sockets` | Listado específico de sockets abiertos. |
| **Objetos e Interactivos** | `windows.handles` | Lista los descriptores (handles) a archivos, claves de registro y mutex por proceso. |
| | `windows.filescan` | Identifica objetos de tipo archivo abiertos en memoria. |
| | `windows.consoles` | Recupera el historial y comandos ejecutados en consolas interactivas (`cmd.exe`). |
| **Malware y Memoria** | `windows.malfind` | Detecta código inyectado y páginas de memoria sospechosas (permisos RWX). |
| | `windows.ssdt` | Comprueba si la SSDT (System Service Descriptor Table) ha sido alterada. |
| | `windows.callbacks` | Muestra las rutinas de callbacks de kernel registradas (común en EDR/Rootkits). |
| | `windows.driverscan` | Escanea la memoria física buscando objetos controlador (drivers). |
| | `windows.yarascan` | Ejecuta reglas YARA sobre todo el espacio de memoria física. |
| | `windows.vadyara` | Ejecuta reglas YARA sobre las regiones VAD (Virtual Address Descriptor). |
| | `windows.vadinfo` | Detalla las asignaciones VAD de un proceso para análisis de inyecciones. |
| **Registro de Windows** | `windows.registry.hivelist` | Muestra la dirección virtual de las colmenas (hives) del registro cargadas. |
| | `windows.registry.printkey` | Imprime el contenido de una clave de registro específica. |
| | `windows.registry.userassist` | Muestra los programas ejecutados por el usuario y su frecuencia desde el registro. |
| | `windows.registry.shellbags` | Revela el historial de carpetas exploradas por los usuarios. |
| | `windows.registry.shimcache` | Lista binarios ejecutados registrados en la base de datos ShimCache. |
| | `windows.registry.amcache` | Analiza el archivo AmCache para extraer información de instalaciones y ejecuciones. |
| **Forense de Archivos** | `windows.hashdump` | Extrae hashes de contraseñas de las cuentas de usuario locales (SAM/SYSTEM). |
| | `windows.dumpfiles` | Extrae archivos cacheados en memoria directamente a disco. |
| | `windows.mftscan` | Escanea registros de la Master File Table (MFT). |
| | `windows.timeliner` | Construye una línea de tiempo agregando marcas de tiempo de múltiples eventos. |

---

## 💻 Comandos Esenciales por Categoría

### 👥 Análisis de Procesos
* **Listar procesos activos de forma jerárquica**:
  ```bash
  python3 vol.py -f memoria.raw windows.pstree
  ```
* **Buscar procesos ocultos mediante escaneo físico**:
  ```bash
  python3 vol.py -f memoria.raw windows.psscan
  ```
* **Obtener la línea de comando exacta de inicio de un proceso**:
  ```bash
  python3 vol.py -f memoria.raw windows.cmdline --pid <PID>
  ```
* **Revisar privilegios específicos asignados a un PID**:
  ```bash
  python3 vol.py -f memoria.raw windows.privs --pid <PID>
  ```

### 🕸️ Conexiones y Tráfico de Red
* **Listar sockets y conexiones activa**:
  ```bash
  python3 vol.py -f memoria.raw windows.netscan
  ```

### 🦠 Detección de Malware y Volcado de Memoria
* **Escanear inyecciones de código (procesos con páginas de memoria inusuales)**:
  ```bash
  python3 vol.py -f memoria.raw windows.malfind --pid <PID>
  ```
* **Escanear memoria RAM con reglas YARA**:
  ```bash
  python3 vol.py -f memoria.raw windows.yarascan --yara-rules myrules.yar
  ```
* **Realizar un volcado completo de la memoria direccionable de un proceso**:
  ```bash
  python3 vol.py -f memoria.raw windows.memdump --pid <PID> --dump-dir ./dumps
  ```

### 📁 Recuperación de Archivos y Credenciales
* **Volcar hashes NTLM locales**:
  ```bash
  python3 vol.py -f memoria.raw windows.hashdump
  ```
* **Extraer un archivo específico cargado en memoria**:
  ```bash
  python3 vol.py -f memoria.raw windows.dumpfiles --pid <PID> --dump-dir ./dumps
  ```

### 🔑 Análisis del Registro de Windows
* **Listar colmenas cargadas en memoria**:
  ```bash
  python3 vol.py -f memoria.raw windows.registry.hivelist
  ```
* **Consultar el historial de ejecución UserAssist**:
  ```bash
  python3 vol.py -f memoria.raw windows.registry.userassist
  ```
* **Consultar ejecuciones registradas en ShimCache**:
  ```bash
  python3 vol.py -f memoria.raw windows.registry.shimcache
  ```

### 🕒 Reconstrucción de Eventos e Historial
* **Generar línea de tiempo (timeline) forense**:
  ```bash
  python3 vol.py -f memoria.raw windows.timeliner
  ```
* **Extraer logs de eventos del sistema (Event Logs)**:
  ```bash
  python3 vol.py -f memoria.raw windows.eventlogs
  ```

---

## ⚙️ Opciones y Parámetros Comunes

* `-f <archivo>`: Ruta al volcado de memoria RAM a analizar (Requerido).
* `-p <PID>`: Filtrar la ejecución del plugin para un identificador de proceso específico.
* `--dump-dir <ruta>`: Directorio destino para exportar archivos o volcados de memoria.
* `--yara-rules <archivo>`: Ruta al archivo conteniendo firmas/reglas YARA.
* `-v` / `--verbose`: Habilitar salida detallada con información de depuración.

---

## ⏰ Determinación del Huso Horario del Sistema

Para correlacionar eventos correctamente durante una investigación forense, es fundamental ajustar la hora local analizando la configuración de zona horaria en el Registro.

### 1. Consultar la clave de Zona Horaria en el Registro
```bash
python3 vol.py -f memoria.raw windows.registry.printkey --key "ControlSet001\\Control\\TimeZoneInformation"
```

### 2. Calcular el desfase horario con `ActiveTimeBias`
El valor de `ActiveTimeBias` en el registro indica la diferencia en minutos con respecto a UTC. Si se representa como un entero de 32 bits con signo y el resultado es muy grande (debido a la representación de números negativos en complemento a dos):

* **Si el número devuelto es negativo** (o interpretado como un valor muy alto tipo `42949672xx` debido al desbordamiento de $2^{32} = 4294967296$):
  * **Cálculo**: Resta $4294967296$ al valor devuelto para obtener el número negativo real.
  * **Fórmula**: $\text{Desfase en horas} = \frac{|\text{Valor negativo}|}{60}$
  * **Resultado**: Representa un huso horario **positivo** (Ej: $+X$ horas con respecto a UTC).
* **Si el número devuelto es positivo y pequeño**:
  * **Cálculo**: Divide directamente el valor entre $60$.
  * **Resultado**: Representa un huso horario **negativo** (Ej: $-X$ horas con respecto a UTC).

---

## 📋 Flujo de Trabajo Forense Recomendado

1. **Reconocimiento inicial**: Ejecutar `windows.info` para determinar la versión exacta de Windows.
2. **Análisis de Procesos**: Listar con `windows.pstree` y buscar anomalías o procesos huérfanos.
3. **Escaneo de Evasiones**: Ejecutar `windows.psscan` para contrastar procesos ocultos eliminados de las listas de doble enlace.
4. **Comprobación de Red**: Ejecutar `windows.netscan` para mapear conexiones remotas maliciosas.
5. **Detección de Inyecciones**: Utilizar `windows.malfind` en busca de secciones de memoria cargadas dinámicamente.
6. **Análisis Forense Avanzado**: Volcar credenciales con `windows.hashdump` y extraer artefactos/archivos con `windows.dumpfiles`.
