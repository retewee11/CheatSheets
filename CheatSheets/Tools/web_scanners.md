# Web Scanners Cheat Sheet (Nikto & WPScan)

Herramientas esenciales para el escaneo automático de vulnerabilidades en aplicaciones web y CMS (WordPress).

---

## 1. Nikto

Nikto es un escáner de servidores web que examina archivos peligrosos, software desactualizado y malas configuraciones.

### Escaneo Básico
```bash
nikto -h http://<IP_OBJETIVO>
nikto -h https://<IP_OBJETIVO> -p 8443
```

### Escaneo a través de Proxy (Pivoting)
```bash
nikto -h http://192.168.1.50 -useproxy http://127.0.0.1:8080
```

### Guardar Resultados
```bash
nikto -h http://<IP_OBJETIVO> -output nikto_report.txt
nikto -h http://<IP_OBJETIVO> -output nikto_report.html -Format html
```

---

## 2. WPScan

Escáner especializado en sitios web construidos sobre WordPress para detectar vulnerabilidades en plugins, temas y enumerar usuarios.

### Enumeración Completa de WordPress
```bash
# Enumeración agresiva de plugins vulnerables, temas y usuarios
wpscan --url http://<IP_OBJETIVO> --enumerate vp,vt,u
```

### Opciones de Enumeración Específicas:
* `--enumerate u` : Enumerar nombres de usuarios de WordPress.
* `--enumerate vp` : Enumerar plugins vulnerables.
* `--enumerate ap` : Enumerar todos los plugins instalados.
* `--enumerate vt` : Enumerar temas vulnerables.

### Ataque de Fuerza Bruta a Contraseñas de Usuarios en WordPress
```bash
# Fuerza bruta a un usuario específico mediante la API de login
wpscan --url http://<IP_OBJETIVO> -U admin -P /usr/share/wordlists/rockyou.txt

# Fuerza bruta usando XML-RPC (más rápido que la interfaz web convencional)
wpscan --url http://<IP_OBJETIVO> -U admin -P /usr/share/wordlists/rockyou.txt --stealthy
```

### Uso con WAF Bypass / User-Agent personalizado
```bash
wpscan --url http://<IP_OBJETIVO> --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```
