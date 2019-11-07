# Ansible para la instalacion de nodos FORWARD y MASTER para Security Onion.

## Tabla de contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Instrucciones para el despliegue de un nodo Master](#instrucciones-para-el-despliegue-de-un-nodo-master)
3. [Instrucciones para el despliegue de un nodo Forward](#instrucciones-para-el-despliegue-de-un-nodo-forward)
4. [Referencias](#referencias)


## Pre-requisitos

1. Contar con un servidor de Security Onion con la interfaz de red pre-configurada: [Repositorio con instrucciones para la instalacion de Security Onion](https://gitlab.unc.edu.ar/csirt/csirt-docs)

2. Agregar clave SSH publica del dispositivo desde el cual se realiza el despliegue en el servidor con Security Onion
   (No agregar claves SSH  sobre el usuario ROOT del servidor con SECURITY ONION ).

3. Contar con un servidor con InfluxDB y Grafana. Los servidores Master y Forwards configurados con Ansible seran integrados con Grafana. 
   Comprobar configuracion de archivo: `roles/telegraf_install/files/telegraf.conf`
 
   Este paso es opcional, en caso de no contar con el servidor con InfluxDB y Grafana instalados setear a la variable
   `INSTALL_TELEGRAF: 'no'` en lugar de  `INSTALL_TELEGRAF: 'yes'`

4. Mantener actualizado el archivo `/roles/securityonion_setup_master/files/clasiffication_rules` con la clasificacion de reglas del Forward node.

5. Contar con un servidor con TheHive instalado y mantener actualizadas las reglas de TheHive que seran copiadas en el Master node. 
   Las reglas actualizadas se encuentran en el [Repositorio con The Hive Rules](https://gitlab.unc.edu.ar/csirt/elastalert-thehive) 
   y deben guardarse en `/roles/securityonion_setup_master/files/clasiffication_rules`, es necesario definir dentro del archivo de variables
   del Master (en carpeta host_vars) las variables:
    *  hive_host
    *  hive_port
    *  hive_apikey
   
   Este paso es opcional, en caso de no contar con el servidor con TheHive instalado setear a la variable
   `COPY_THEHIVE_RULES: 'no'` en lugar de `COPY_THEHIVE_RULES: 'yes'`, en el archivo de variables de la carpeta `host_vars`.


## Instrucciones para el despliegue de un nodo Master

 Seguir las instrucciones del documento: [Instrucciones para el despligue de un Master Node](documentacion/instructions-master-node.md)

## Instrucciones para el despliegue de un nodo Forward

 Seguir las instrucciones del documento: [Instrucciones para el despligue de un Forward Node](documentacion/instructions-forward-node.md)

## Referencias

* https://adamdehaven.com/blog/how-to-generate-an-ssh-key-and-add-your-public-key-to-the-server-for-authentication/
* https://docs.ansible.com/ansible/latest/modules/expect_module.html
* https://unix.stackexchange.com/questions/453859/when-i-use-ansible-module-expect-i-get-this-msg-the-pexpect-python-module-is-r
* https://stackoverflow.com/questions/35095481/how-to-find-list-membership-in-a-list-in-ansible

















