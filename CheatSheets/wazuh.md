# Wazuh Setup & Configuration Guide
> **Guía práctica para la instalación del servidor Wazuh, despliegue de agentes e integración del detector de vulnerabilidades.**

---

## 1. Instalación del Servidor Wazuh

Antes de realizar la instalación, asegúrate de expandir el volumen de almacenamiento lógico en tu sistema (si es necesario) y descargar el instalador oficial.

* **Extender volumen lógico y redimensionar el sistema de archivos (Ubuntu)**:
  ```bash
  sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
  sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
  ```
* **Descargar y ejecutar el script de instalación todo en uno**:
  ```bash
  curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
  ```
  > [!IMPORTANT]
  > Al finalizar la instalación, el script imprimirá en consola las credenciales de acceso por defecto (usuario `admin` y una contraseña aleatoria). Asegúrate de guardarlas.

* **Acceso a la interfaz web de administración**:
  Abre tu navegador e ingresa a la siguiente URL (reemplazando por la dirección IP de tu servidor):
  ```http
  https://<IP_SERVIDOR_WAZUH>
  ```

---

## 2. Despliegue e Incorporación de Agentes

Para comenzar a monitorear un host (cliente), debes registrarlo como agente en el panel web.

1. **Generar el comando de registro**:
   * En la interfaz web de Wazuh, dirígete a **Agents management** > **Summary** > **Add new agent**.
   * Selecciona los parámetros correspondientes de tu máquina cliente:
     * Sistema operativo (ej. `Debian/Ubuntu` - arquitectura `amd64`).
     * Dirección IP del servidor Wazuh.
     * Nombre descriptivo para identificar al agente.
     * Grupo al que pertenecerá el agente.
2. **Ejecutar el comando de instalación en la máquina cliente**:
   Copia el comando generado por la web y ejecútalo en la terminal del host cliente para instalar el paquete del agente.
3. **Iniciar el agente**:
   Ejecuta el comando indicado en la misma pantalla del asistente para iniciar y habilitar el servicio en la máquina cliente:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable wazuh-agent
   sudo systemctl start wazuh-agent
   ```
4. **Verificación**:
   Una vez completados los pasos, confirma que el agente figura en estado activo (Active) en la sección **Summary** de la web de Wazuh.

---

## 3. Activación del Detector de Vulnerabilidades

Por defecto, el módulo de detección de vulnerabilidades puede estar desactivado en el gestor. Sigue estos pasos en el **Servidor** para habilitarlo.

1. **Editar el archivo de configuración interna**:
   ```bash
   sudo nano /var/ossec/etc/internal_options.conf
   ```
2. **Habilitar el Gestor de Escaneo**:
   Busca la directiva del detector de vulnerabilidades y cambia el valor a `0` para activarla (cambiar de `1` a `0`):
   ```ini
   # Vulnerability detector - Enable or disable the scan manager
   # 0. Enabled
   # 1. Disabled
   vulnerability-detection.disable_scan_manager=0
   ```
3. **Reiniciar el servicio del Administrador de Wazuh** para aplicar los cambios:
   ```bash
   sudo systemctl restart wazuh-manager
   ```

---

## 4. Auditoría y Verificación de Vulnerabilidades

Una vez activado el escaneo, puedes visualizar el estado de seguridad en la consola web:

1. Dirígete a la sección **Vulnerability detection** > **Endpoint security**.
2. Accede a **Configuration assessment** para inspeccionar las pruebas de cumplimiento de seguridad realizadas en el agente (tests aprobados y fallidos).
3. **Solución de debilidades**: Cada test que no haya sido superado indicará detalladamente el comando recomendado y la configuración que debes modificar en el cliente para mitigar el riesgo.
4. **Actualizar el agente**: Tras aplicar las correcciones recomendadas en el cliente, reinicia su servicio para forzar un nuevo reporte:
   ```bash
   sudo systemctl restart wazuh-agent
   ```
5. Recarga la consola web (F5) para verificar que las correcciones han surtido efecto y el sistema figura como protegido.