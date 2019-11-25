## Instrucciones para el despliegue de un servidor Master

## Tabla de contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Template Forward Node](#template-forward-node)
3. [Despliegue con Ansible](#despliegue-con-ansible)


## Pre-requisitos

1. Contar con un servidor de Security Onion con la interfaz de red pre-configurada: [Repositorio con instrucciones para la instalacion de Security Onion](https://gitlab.unc.edu.ar/csirt/csirt-docs)

2. Agregar clave SSH publica del dispositivo desde el cual se realiza el despliegue en el servidor con Security Onion
   (No agregar claves SSH  sobre el usuario ROOT del servidor con SECURITY ONION ).

3. Contar con un servidor con InfluxDB y Grafana. Los servidores Master y Forwards configurados con Ansible seran integrados con Grafana. 
   Comprobar configuracion de archivo: `roles/telegraf_install/files/telegraf.conf`
 
   Este paso es opcional, en caso de no contar con el servidor con InfluxDB y Grafana instalados setear a la variable
   `INSTALL_TELEGRAF: 'no'` en lugar de  `INSTALL_TELEGRAF: 'yes'`


## Template Master Node

*  En la carpeta `host_vars` agregar un archivo .yml que tiene la forma del archivo `template_master.yml`, renombrar de la forma `nombre_usuario.yml` 
 (Ej. sonionmaster.yml) y modificar las variables de configuracion para el despliguete del Master Node  (`nombre_usuario` es utilizado en el paso siguiente y en la variable `ansible_user`).
    
   
    ```yaml

	ansible_host: '172.16.81.127'
	ansible_user: 'soniontest2'

	##########################################
	#Variables for file sosetup_forward.conf
	##########################################

	# MGMT_INTERFACE
	# Which network interface should be the management interface?
	MGMT_INTERFACE: 'ens160'
	# MGMT_CONFIG_TYPE
	# Should the management interface be configured using DHCP or static IP?
	MGMT_CONFIG_TYPE: 'static'
	# If MGMT_CONFIG_TYPE=static, then provide the details here.
	# If you have multiple nameservers, please separate them with spaces like this: NAMESERVER='192.168.204.2 192.168.204.3'
	ADDRESS: '172.16.81.127'
	NETMASK: '255.255.255.0'
	GATEWAY: '172.16.81.1'
	NAMESERVER: '200.16.16.1 200.16.16.2 8.8.8.8'
	DOMAIN: 'sonionmaster.local.psi' 

	# LOG_SIZE_LIMIT
	# This setting controls how much disk space Elastic uses.
	# 100GB = 100000000000
	# LOG_SIZE_LIMIT='100000000000'
	LOG_SIZE_LIMIT: '100000000000'

	# IDS_RULESET
	# Which IDS ruleset would you like to use?
	# Emerging Threats Open (no oinkcode required): ETOPEN
	# Emerging Threats PRO (requires ETPRO oinkcode): ETPRO
	# Sourcefire Talos (requires Talos oinkcode): TALOS
	# TALOS and ET (requires TALOS oinkcode): TALOSET
	IDS_RULESET: 'ETOPEN'

	# IDS_ENGINE
	# Which IDS engine would you like to run?  snort/suricata
	# Whatever you choose here will apply to the master server
	# and then sensors inherit this setting from the master server.
	# To run Snort:
	# IDS_ENGINE='snort'
	# To run Suricata:
	# IDS_ENGINE='suricata'
	IDS_ENGINE: 'suricata'


	# WARN_DISK_USAGE
	# Begin warning when disk usage reaches this level
	WARN_DISK_USAGE: '80'
	# CRIT_DISK_USAGE
	# Begin purging old files when disk usage reaches this level
	CRIT_DISK_USAGE: '90'
	# DAYSTOKEEP
	# Only applies to Sguil database ('securityonion_db')
	DAYSTOKEEP: '30'
	# DAYSTOREPAIR
	# Only applies to Sguil database ('securityonion_db')
	DAYSTOREPAIR: '7'



	#########################################################
	#Install Telegraf 
	#(Modify File roles/telegraf_install/files/telegraf.conf)
	#########################################################

	INSTALL_TELEGRAF: 'yes' #'no'

	#########################################################
	#Configuring firewall for analyst for administration purposes
	#Please enter the IP address (or CIDR range) you'd like 
	#to allow to connect to port(s): 22,443,7734 
	#########################################################

	allow_analyst_network: 'yes' #no

	analyst_network: '172.16.81.0/24' #IP address (or CIDR range like 172.16.81.0/24)


    ```

        
## Despliegue con Ansible


*  Agregar nombre de usuario del nodo Master al archivo `hosts` en el grupo `[master]` 
(Ej. como en el paso anterior se crea el archivo `sonionmaster.yml` agregar `sonionmaster` en el archivo `hosts`):

    ```yaml
    [master]
    sonionmaster
    
    ```


*  Ejecutar Ansible sobre el servidor `"sonionmaster"` (el username se define en la opcion extra_var):

    ```bash
    $ ansible-playbook -i hosts -l master so_setup.yml --extra-var "target=sonionmaster" --ask-become-pass
    ```
Una vez ejecutado el comando se le solicitara el pass root para el servidor.
