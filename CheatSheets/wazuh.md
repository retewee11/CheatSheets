# Wazuh Deployment and Configuration Guide

Guía de comandos y configuración para la instalación del servidor Wazuh, el despliegue de agentes y la habilitación del detector de vulnerabilidades.

---

## 1. Instalación del Servidor Wazuh

### Extender volumen lógico (Ubuntu - si aplica)
```bash
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

### Ejecutar script de instalación todo en uno
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

> [!IMPORTANT]
> El instalador imprimirá en la consola las credenciales por defecto (usuario `admin` y contraseña generada). Registrar estos datos antes de continuar.

### Acceso a la interfaz web
```http
https://<IP_SERVIDOR_WAZUH>
```

---

## 2. Despliegue de Agentes

1. **Generación del comando en la consola de Wazuh**:
   * Acceder a **Agents management** > **Summary** > **Add new agent**.
   * Seleccionar los parámetros del host (S.O., arquitectura, IP del servidor y grupo).
2. **Instalación en el host cliente**:
   * Ejecutar en el cliente el comando de instalación generado.
3. **Inicio del servicio en el cliente**:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable wazuh-agent
   sudo systemctl start wazuh-agent
   ```
4. **Verificación**:
   * Comprobar que el agente figure como activo en la consola de administración.

---

## 3. Configuración del Detector de Vulnerabilidades

Este módulo requiere ser habilitado en el **servidor de Wazuh**.

1. **Editar archivo de configuración**:
   ```bash
   sudo nano /var/ossec/etc/internal_options.conf
   ```
2. **Habilitar el Gestor de Escaneo**:
   Modificar el valor de la directiva `vulnerability-detection.disable_scan_manager` a `0`:
   ```ini
   # Vulnerability detector - Enable or disable the scan manager
   # 0. Enabled
   # 1. Disabled
   vulnerability-detection.disable_scan_manager=0
   ```
3. **Reiniciar el servicio del gestor**:
   ```bash
   sudo systemctl restart wazuh-manager
   ```

---

## 4. Gestión y Auditoría de Vulnerabilidades

* **Visualizar vulnerabilidades**:
  Consultar el panel **Vulnerability detection** > **Endpoint security** en la interfaz de administración.
* **Evaluación de configuración**:
  Consultar **Configuration assessment** para revisar tests de seguridad aplicados en el host.
* **Forzar re-escaneo posterior a remediaciones**:
  Tras aplicar las remediaciones sugeridas en el cliente, reiniciar el agente para enviar un reporte inmediato al servidor:
  ```bash
  sudo systemctl restart wazuh-agent
  ```