# Volatility 3 Cheat Sheet (Windows)

Guía de comandos y referencia de plugins para el análisis forense digital de memoria RAM en sistemas operativos Windows utilizando Volatility 3.

---

## Uso Básico

* Comando general para ejecutar plugins:
  ```bash
  python3 vol.py -f <memoria.raw> <plugin>
  ```
* Obtener información básica del sistema (S.O., versión, arquitectura):
  ```bash
  python3 vol.py -f <memoria.raw> windows.info
  ```

---

## Referencia de Plugins para Windows

| Categoría | Plugin | Descripción |
| :--- | :--- | :--- |
| **Información General** | `windows.info` | Información básica de la imagen de memoria analizada. |
| **Procesos** | `windows.pslist` | Lista de procesos activos (estructuras del kernel activas). |
| | `windows.pstree` | Jerarquía de procesos en formato de árbol. |
| | `windows.psscan` | Escaneo físico en memoria para identificar procesos ocultos o terminados. |
| | `windows.cmdline` | Argumentos y líneas de comandos de inicio de los procesos. |
| | `windows.getsid` | Identificadores de seguridad (SID) asociados a cada proceso. |
| | `windows.privs` | Privilegios asignados a nivel de proceso. |
| **Módulos y DLLs** | `windows.modules` | Módulos cargados en el espacio del Kernel. |
| | `windows.dlllist` | Bibliotecas dinámicas (DLLs) cargadas por cada proceso. |
| | `windows.ldrmodules` | Contraste de listas de carga (PEB) para detectar inyecciones o desalineaciones. |
| **Red** | `windows.netscan` | Conexiones de red activas, puertos en escucha y sockets abiertos. |
| | `windows.sockets` | Listado específico de sockets abiertos. |
| **Objetos** | `windows.handles` | Descriptores abiertos (archivos, llaves de registro, mutexes) por proceso. |
| | `windows.filescan` | Objetos de tipo archivo cargados en memoria. |
| | `windows.consoles` | Historial de comandos de consola interactiva (`cmd.exe`). |
| **Malware** | `windows.malfind` | Detección de código inyectado y páginas de memoria sospechosas (RWX). |
| | `windows.ssdt` | Comprobación de integridad de la tabla SSDT (System Service Descriptor Table). |
| | `windows.callbacks` | Rutinas de callback registradas en el kernel (comunes en EDRs/Rootkits). |
| | `windows.driverscan` | Escaneo en busca de objetos de controladores de dispositivo (drivers). |
| | `windows.yarascan` | Búsqueda de firmas YARA en la memoria física global. |
| | `windows.vadyara` | Búsqueda de firmas YARA en las regiones VAD (Virtual Address Descriptor). |
| | `windows.vadinfo` | Detalles sobre asignaciones VAD para análisis de inyecciones de memoria. |
| **Registro** | `windows.registry.hivelist` | Identificación y direcciones de las colmenas (hives) cargadas en memoria. |
| | `windows.registry.printkey` | Impresión del contenido de llaves de registro específicas. |
| | `windows.registry.userassist` | Programas ejecutados por el usuario y estadísticas de ejecución. |
| | `windows.registry.shellbags` | Historial de navegación y directorios accedidos por el usuario. |
| | `windows.registry.shimcache` | Base de datos de compatibilidad de aplicaciones (ShimCache) para ejecuciones. |
| | `windows.registry.amcache` | Análisis de Amcache para identificar metadatos de binarios ejecutados. |
| **Forense de Archivos**| `windows.hashdump` | Extracción de hashes de contraseñas de las cuentas de usuario locales (SAM/SYSTEM). |
| | `windows.dumpfiles` | Volcado a disco de archivos cacheados en memoria. |
| | `windows.mftscan` | Escaneo de registros de la MFT (Master File Table). |
| | `windows.timeliner` | Línea de tiempo unificada generada a partir de múltiples artefactos de memoria. |

---

## Comandos por Categoría

### Análisis de Procesos
* Listar procesos activos en estructura de árbol:
  ```bash
  python3 vol.py -f memoria.raw windows.pstree
  ```
* Buscar procesos ocultos mediante escaneo físico de estructuras:
  ```bash
  python3 vol.py -f memoria.raw windows.psscan
  ```
* Obtener los argumentos de línea de comandos de un proceso específico:
  ```bash
  python3 vol.py -f memoria.raw windows.cmdline --pid <PID>
  ```
* Consultar privilegios asignados a un proceso específico:
  ```bash
  python3 vol.py -f memoria.raw windows.privs --pid <PID>
  ```

### Análisis de Conexiones de Red
* Listar sockets activos y puertos abiertos:
  ```bash
  python3 vol.py -f memoria.raw windows.netscan
  ```

### Detección de Malware y Volcado de Datos
* Identificar inyecciones de código en memoria:
  ```bash
  python3 vol.py -f memoria.raw windows.malfind --pid <PID>
  ```
* Escanear con firmas YARA en la memoria:
  ```bash
  python3 vol.py -f memoria.raw windows.yarascan --yara-rules reglas.yar
  ```
* Volcar el espacio de direccionamiento de un proceso específico:
  ```bash
  python3 vol.py -f memoria.raw windows.memdump --pid <PID> --dump-dir ./salida
  ```

### Extracción de Credenciales y Archivos
* Volcar hashes NTLM locales desde el registro en memoria:
  ```bash
  python3 vol.py -f memoria.raw windows.hashdump
  ```
* Volcar un archivo específico guardado en la caché de memoria:
  ```bash
  python3 vol.py -f memoria.raw windows.dumpfiles --pid <PID> --dump-dir ./salida
  ```

### Análisis del Registro de Windows
* Listar colmenas (hives) cargadas en memoria:
  ```bash
  python3 vol.py -f memoria.raw windows.registry.hivelist
  ```
* Extraer el registro de ejecuciones UserAssist:
  ```bash
  python3 vol.py -f memoria.raw windows.registry.userassist
  ```
* Extraer el historial de ShimCache:
  ```bash
  python3 vol.py -f memoria.raw windows.registry.shimcache
  ```

---

## Parámetros Comunes

* `-f <ruta>`: Ruta del volcado de memoria (requerido).
* `-p <PID>`: Filtrar por identificador de proceso (opcional).
* `--dump-dir <ruta>`: Directorio de salida para exportar archivos o volcados.
* `--yara-rules <ruta>`: Archivo con firmas/reglas YARA para escaneo en memoria.
* `-v` / `--verbose`: Salida detallada y de depuración.

---

## Determinación de Zona Horaria (ActiveTimeBias)

El valor de `ActiveTimeBias` en el registro indica el desfase en minutos respecto a UTC.

### 1. Consultar la clave en el registro
```bash
python3 vol.py -f memoria.raw windows.registry.printkey --key "ControlSet001\\Control\\TimeZoneInformation"
```

### 2. Cálculo matemático
* **Número devuelto negativo** (o valor tipo `42949672xx` debido a representación de 32 bits con signo):
  * Restar `4294967296` al valor devuelto para obtener el número real.
  * Desfase (horas) = `|Valor real| / 60`.
  * Resultado: Representa un huso horario **positivo** (UTC+X).
* **Número devuelto positivo y pequeño**:
  * Dividir directamente entre `60`.
  * Resultado: Representa un huso horario **negativo** (UTC-X).

---

## Flujo de Trabajo Recomendado

1. **Reconocimiento**: Ejecutar `windows.info` para identificar el sistema operativo y arquitectura.
2. **Procesos**: Listar con `windows.pstree` y correlacionar con `windows.psscan` para detectar procesos huérfanos u ocultos.
3. **Red**: Correlacionar procesos sospechosos con conexiones externas activas utilizando `windows.netscan`.
4. **Memoria**: Analizar inyecciones de código con `windows.malfind`.
5. **Artefactos**: Volcar credenciales locales (`windows.hashdump`) y extraer archivos sospechosos (`windows.dumpfiles`).
