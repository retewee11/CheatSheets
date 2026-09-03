# Searchsploit Cheat Sheet

Guía de comandos para la búsqueda offline de vulnerabilidades y exploits públicos con la herramienta `searchsploit` (Exploit-DB CLI).

---

## 1. Búsquedas Básicas

```bash
# Búsqueda general por nombre de software o servicio
searchsploit wordpress 5.0

# Búsqueda exacta por versión de software
searchsploit "Apache 2.4.41"

# Búsqueda por CVE
searchsploit cve:2021-44228
```

---

## 2. Búsqueda con Filtros Avanzados

```bash
# Filtrar por título (evita coincidir con rutas de archivo)
searchsploit -t vsftpd

# Búsqueda estricta haciendo coincidir todos los términos
searchsploit --www -t "Drupal 7"

# Excluir términos de la búsqueda (usar filtro pipe o grep)
searchsploit linux privilege escalation | grep -v "Kernel"
```

---

## 3. Inspección y Copia de Exploits

### Inspeccionar el código fuente del exploit
```bash
# Examinar el contenido de un exploit sin abrir un editor
searchsploit -x exploits/php/webapps/49898.py
```

### Copiar (Mirror) el exploit al directorio actual
```bash
# Copia el exploit directamente a la ruta actual para poder editarlo o ejecutarlo
searchsploit -m exploits/php/webapps/49898.py
searchsploit -m 49898
```

---

## 4. Actualización del Índice de Exploits

```bash
# Actualizar la base de datos de searchsploit
sudo searchsploit -u
```
