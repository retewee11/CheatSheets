# Volatility 3 Cheat Sheet para Windows

## Introducción
Volatility 3 es una poderosa herramienta de análisis forense de memoria RAM, utilizada para investigar sistemas Windows, Linux y Mac. Esta cheat sheet está enfocada en el análisis de memoria de sistemas Windows.

---

## Uso Básico

```bash
python3 vol.py -f <memoria.raw> windows.info
```

- `-f <memoria.raw>`: Archivo de memoria a analizar.
- `windows.info`: Plugin a ejecutar.

---

## Plugins Esenciales para Windows

### Información General
- `windows.info` — Información básica del sistema.
- `windows.pslist` — Lista de procesos activos.
- `windows.pstree` — Árbol de procesos.
- `windows.psscan` — Escaneo de procesos ocultos.
- `windows.modules` — Módulos del kernel cargados.
- `windows.ldrmodules` — Módulos cargados por proceso.
- `windows.dlllist` — DLLs cargadas por proceso.
- `windows.handles` — Handles abiertos por proceso.
- `windows.filescan` — Archivos abiertos en memoria.
- `windows.netscan` — Conexiones de red y sockets.
- `windows.sockets` — Sockets abiertos.
- `windows.netscan` — Escaneo de conexiones de red.
- `windows.cmdline` — Comandos ejecutados por procesos.
- `windows.getservicesids` — Servicios y sus SIDs.
- `windows.getserviceinfo` — Información de servicios.
- `windows.getsid` — SIDs de procesos.
- `windows.malfind` — Detección de código malicioso en memoria.
- `windows.ssdt` — Tabla de Descriptores de Servicios del Sistema.
- `windows.callbacks` — Callbacks del kernel.
- `windows.driverscan` — Escaneo de drivers.
- `windows.registry.hivelist` — Listado de hives del registro.
- `windows.registry.printkey` — Visualización de claves del registro.
- `windows.registry.userassist` — Programas ejecutados por el usuario.
- `windows.registry.shellbags` — Carpetas accedidas por el usuario.
- `windows.registry.userassist` — Programas ejecutados por el usuario.
- `windows.registry.shimcache` — Programas ejecutados (ShimCache).
- `windows.registry.amcache` — Programas instalados (AmCache).
- `windows.hashdump` — Hashes de contraseñas.
- `windows.dumpfiles` — Extraer archivos de la memoria.
- `windows.memmap` — Mapeo de memoria de procesos.
- `windows.vadinfo` — Información de VADs (Virtual Address Descriptors).
- `windows.vadwalk` — Recorrido de VADs.
- `windows.vadyara` — YARA sobre VADs.
- `windows.yarascan` — YARA sobre memoria.
- `windows.bigpools` — Pooles de memoria grandes.
- `windows.poolscanner` — Escaneo de pools de memoria.
- `windows.privs` — Privilegios de procesos.
- `windows.sids` — SIDs de procesos.
- `windows.cmdscan` — Comandos ejecutados en consola.
- `windows.consoles` — Consolas abiertas.
- `windows.eventlogs` — Logs de eventos.
- `windows.mftscan` — Escaneo de la MFT.
- `windows.timeliner` — Línea de tiempo de eventos.

---



## Comandos Esenciales Adicionales

### Análisis de registro
python3 vol.py -f memoria.raw windows.registry.userassist
python3 vol.py -f memoria.raw windows.registry.shellbags
python3 vol.py -f memoria.raw windows.registry.shimcache
python3 vol.py -f memoria.raw windows.registry.amcache

### Timeline de eventos
python3 vol.py -f memoria.raw windows.timeliner

### Logs de eventos
python3 vol.py -f memoria.raw windows.eventlogs

### Escaneo de la MFT
python3 vol.py -f memoria.raw windows.mftscan

### Privilegios y SIDs
python3 vol.py -f memoria.raw windows.privs --pid <PID>
python3 vol.py -f memoria.raw windows.sids --pid <PID>

### Buscar rootkits
python3 vol.py -f memoria.raw windows.ssdt
python3 vol.py -f memoria.raw windows.callbacks

### Análisis de red
python3 vol.py -f memoria.raw windows.netscan
python3 vol.py -f memoria.raw windows.sockets

### Listar procesos y obtener PID
python3 vol.py -f memoria.raw windows.pslist
python3 vol.py -f memoria.raw windows.pstree

### Dump de memoria de un proceso específico
python3 vol.py -f memoria.raw windows.memdump --pid <PID> --dump-dir ./dumps

### Dump de archivos mapeados en un proceso
python3 vol.py -f memoria.raw windows.dumpfiles --pid <PID> --dump-dir ./dumps

### Dump de DLLs de un proceso
python3 vol.py -f memoria.raw windows.dlllist --pid <PID>

### Buscar regiones sospechosas en un proceso
python3 vol.py -f memoria.raw windows.malfind --pid <PID>

### Buscar comandos ejecutados en un proceso
python3 vol.py -f memoria.raw windows.cmdline --pid <PID>

### Mapeo de memoria de un proceso
python3 vol.py -f memoria.raw windows.memmap --pid <PID>

### Información de VADs de un proceso
python3 vol.py -f memoria.raw windows.vadinfo --pid <PID>

### Buscar con YARA en VADs de un proceso
python3 vol.py -f memoria.raw windows.vadyara --pid <PID> --yara-rules myrules.yar

### Extraer handles de un proceso
python3 vol.py -f memoria.raw windows.handles --pid <PID>


### Listar procesos
```bash
python3 vol.py -f memoria.raw windows.pslist
```

### Buscar procesos ocultos
```bash
python3 vol.py -f memoria.raw windows.psscan
```

### Listar conexiones de red
```bash
python3 vol.py -f memoria.raw windows.netscan
```

### Buscar código malicioso
```bash
python3 vol.py -f memoria.raw windows.malfind
```

### Extraer archivos
```bash
python3 vol.py -f memoria.raw windows.dumpfiles --dump-dir ./dumped_files
```

### Buscar con YARA
```bash
python3 vol.py -f memoria.raw windows.yarascan --yara-rules myrules.yar
```

---

## Opciones Comunes

- `-f <archivo>`: Archivo de memoria.
- `-o <offset>`: Offset de la partición.
- `-p <PID>`: ID de proceso.
- `--dump-dir <dir>`: Carpeta de salida para archivos extraídos.
- `--yara-rules <archivo>`: Archivo de reglas YARA.
- `-v` o `--verbose`: Salida detallada.
- `-h` o `--help`: Ayuda del plugin.

---

## Flujo de Trabajo Recomendado

1. `windows.info` — Confirmar perfil y versión de Windows.
2. `windows.pslist` / `windows.pstree` — Analizar procesos.
3. `windows.psscan` — Buscar procesos ocultos.
4. `windows.netscan` — Revisar conexiones de red.
5. `windows.malfind` — Buscar inyecciones de código.
6. `windows.dlllist` / `windows.ldrmodules` — Revisar DLLs sospechosas.
7. `windows.handles` — Buscar handles sospechosos.
8. `windows.registry.*` — Analizar el registro.
9. `windows.hashdump` — Extraer hashes.
10. `windows.dumpfiles` — Extraer archivos sospechosos.

---

## Consejos Avanzados

- Usa `windows.vadyara` para buscar patrones en los VADs de procesos.
- Combina `windows.psscan` y `windows.pslist` para detectar rootkits.
- Usa `windows.registry.printkey` para explorar claves específicas del registro.
- Extrae y analiza archivos con `windows.dumpfiles`.
- Usa `windows.timeliner` para reconstruir la línea de tiempo de eventos.
- Aplica reglas YARA personalizadas para detectar malware.





## Problema con la hora del sistema.

mirar la hora local del sistema

vol -f memorydump windows.registry.printkey --key "ControlSet001\\Control\\TimeZoneInformation" 


activeTimeBias - 2 elevado a 32 ( 4294967296) si el numero que devuelve es negativo  entre 60 y nos da el + cuanto es tu hora y si da positivo dividimos entre 60 y es el - cuanto es la hora
