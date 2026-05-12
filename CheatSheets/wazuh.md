#Wazuh

El primer paso es instalar wazuh en el servidor con este comando:

```bash
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv

curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

Luego accederemos a la interfaz web de wazuh con la siguiente URL:

```
https://<IP_ADDRESS>
```
El usuario y contraseña nos lo dará el script de instalación al final de ella.

Una vez dentro de la interfaz web, podemos agregar agentes a nuestro servidor wazuh para monitorear nuestros sistemas. Para agregar un agente, debemos seguir los siguientes pasos:
1. En la interfaz web de wazuh, ir a la sección "Agents management" y hacer clic en "summary" y ahí en "Add new agent".

2. Aquí tendrás que introducir el sistema operativo, en nuestro caso deb amd64, el nombre del agente, la IP del servidor wazuh y el grupo al que quieres que pertenezca el agente.

3. En esa misma ventana tendremos un paso que nos dará el comando para copiar y pegar en el agente que queremos monitorear.

4. Luego una vez instalado el agente, nos dará en la propia web otro comando que es para reiniciar los servicios del agente y así empezar a monitorear el sistema.

Una vez que el agente esté instalado y configurado, podremos ver que está activo en "summary", ahora en el servidor haremos lo siguiente:

```bash
sudo su
nano /var/ossec/etc/internal_options.conf
```
Aquí buscaremos la línea que habla de vulnerability detector y scan manager y cambiaremos el 1 por un 0 para activar la funcion de escaneo de vulnerabilidades.

```bash
# Vulnerability detector - Enable or disable the scan manager
# 0. Enabled
# 1. Disabled
vulnerability-detection.disable_scan_manager=1
```
Luego reiniciamos el servicio de wazuh para que los cambios tengan efecto:

```bash
sudo systemctl restart wazuh-manager
```
Ahora ya tendremos el escaneo de vulnerabilidades activado y podremos ver los resultados en la interfaz web de wazuh. Para ver los resultados, debemos ir a la sección "Vulnerability detection" luego iremos a endpoint security ahí a configuration assesment y ahi vemos el resultado de los test que se han pasado y los que no se han pasado. cada test que no se haya pasado te dirá que comando usar y que hay que modificar para pasar el test.

Luego de seguir todos los pasos, tendremos que reiniciar en el cliente el servicio de wazuh para que los cambios tengan efecto:

```bash
sudo systemctl restart Wazuh-agent.service
```

Y al darle a F5 en la interfaz web de wazuh, podremos ver que el agente ha pasado todos los test y ya tenemos nuestro sistema monitoreado y protegido contra vulnerabilidades.