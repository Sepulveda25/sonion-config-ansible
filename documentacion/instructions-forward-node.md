## Instrucciones para el despliegue de un nodo Forward

## Tabla de contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Template Forward Node](#template-forward-node)
3. [Despliegue con Ansible](#despliegue-con-ansible)

## Pre-requisitos

1. Contar con un servidor de Security Onion con la interfaz de red pre-configurada: [Repositorio con instrucciones para la instalacion de Security Onion](https://gitlab.unc.edu.ar/csirt/csirt-docs)
   Se debe contar con al menos dos interfaces una para administracion y otra para realizar el monitoreo. 

2. Agregar clave SSH publica del dispositivo desde el cual se realiza el despliegue en el servidor con Security Onion
   (No agregar claves SSH  sobre el usuario ROOT del servidor con SECURITY ONION ).

3. Contar con un servidor con InfluxDB y Grafana. Los servidores Master y Forwards configurados con Ansible seran integrados con Grafana. 
   Comprobar configuracion de archivo: `roles/telegraf_install/files/telegraf.conf`
 
   Este paso es opcional, en caso de no contar con el servidor con InfluxDB y Grafana instalados setear a la variable
   `INSTALL_TELEGRAF: 'no'` en lugar de  `INSTALL_TELEGRAF: 'yes'`

4. Mantener actualizado el archivo `/roles/securityonion_setup_master/files/clasiffication_rules` con la clasificacion de reglas del Forward node.

5. Contar con un servidor con TheHive instalado y mantener actualizadas las reglas de TheHive que seran copiadas en el Master Node desde el Forward Node. 
   Las reglas actualizadas se encuentran en el [Repositorio con The Hive Rules](https://gitlab.unc.edu.ar/csirt/elastalert-thehive) 
   y deben guardarse en `/roles/copy_hive_rules_to_master/files/thehive_rules`, es necesario definir dentro del archivo de variables
   del Foward Node (en carpeta host_vars) las variables:
    *  hive_host
    *  hive_port
    *  hive_apikey
   
   Este paso es opcional, en caso de no contar con el servidor con TheHive instalado setear a la variable
   `COPY_THEHIVE_RULES: 'no'` en lugar de `COPY_THEHIVE_RULES: 'yes'`, en el archivo de variables de la carpeta `host_vars`.

## Template Forward Node
    
    
*   En la carpeta `host_vars` agregar (copiar) un archivo .yml que tiene la forma del archivo `template_forward.yml`, renombrar de la forma `nombre_usuario.yml` 
 (Ej. sonionforward.yml) y modificar las variables de configuracion para el despliguete del Foward Node  (`nombre_usuario` es utilizado en el paso siguiente y en la variable `ansible_user`).

Dentro del archivo `template_forward.yml` tenemos las siguientes variables:

- `ansible_host` y `ansible_user` corresponden a la IP y Username del host objetivo (el Forward Node).

    ```yaml
        ansible_host: '172.16.81.126'
        ansible_user: 'sonionforwar
    ```

- La seccion `Variables for file sosetup_forward.conf` incluye todos los campos necesarios para ejecutar el setup de Security Onion:

Se selecciona la interfaz de administracion, se coloca el nombre de la misma en el campo `MGMT_INTERFACE`
(es necesario un conocimiento previo de las interfaces), se configura tambien el uso de una direccion IP estatica o DHCP. 
En caso de ser una direccion IP estatica se configuran la direccion IP, mascara de red, gateway, servidores DNS y un nombre de dominio.
 
    ```yaml
        # MGMT_INTERFACE
        # Which network interface should be the management interface?
        MGMT_INTERFACE: 'ens160'
        # MGMT_CONFIG_TYPE
        # Should the management interface be configured using DHCP or static IP?
        MGMT_CONFIG_TYPE: 'static'
        # If MGMT_CONFIG_TYPE=static, then provide the details here.
        # If you have multiple nameservers, please separate them with spaces like this: NAMESERVER='192.168.204.2 192.168.204.3'
        ADDRESS: '172.16.81.126'
        NETMASK: '255.255.255.0'
        GATEWAY: '172.16.81.1'
        NAMESERVER: '200.16.16.1 200.16.16.2 8.8.8.8'
        DOMAIN: 'soniontest.local.psi'
    ```

