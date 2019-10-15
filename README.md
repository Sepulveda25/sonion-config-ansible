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

4. Mantener actualizado el archivo `/roles/securityonion_setup_master/files/clasiffication_rules` con la clasificacion de reglas del Master server.

5. Mantener actualizadas las reglas de TheHive. Las mismas se encuentran en el [Repositorio con The Hive Rules](https://gitlab.unc.edu.ar/csirt/elastalert-thehive) 
   y se almacenan en `/roles/securityonion_setup_master/files/clasiffication_rules`, comprobar en cada regla las variables:
    *  hive_host
    *  hive_port
    *  hive_apikey
    


## Instrucciones para el despliegue de un nodo Master

 Seguir las instrucciones del documento: [Instrucciones para el despligue de un Master Node](Documentacion/instructions-master-node.md)

## Instrucciones para el despliegue de un nodo Forward

*  Agregar nombre de usuario del nodo forward al archivo `host` en el grupo `forward_nodes` (Ej. user sonionforward):

    ```
        [forward_nodes]
        sonionforward
    ```
    
*  En la carpeta `host_vars` agregar un archivo yml (`nombre_usuario.yml`) en la que se especifiquen las variables
   del archivo template_forward.yml (Ej. sonionforward.yml):

    ```
	ansible_host: '172.16.81.126'
	ansible_user: 'soniontest'

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
	ADDRESS: '172.16.81.126'
	NETMASK: '255.255.255.0'
	GATEWAY: '172.16.81.1'
	NAMESERVER: '200.16.16.1 200.16.16.2 8.8.8.8'
	DOMAIN: 'soniontest.local.psi'

	# Which interface(s) will be sniffing network traffic?
	# For multiple interfaces, please separate them with spaces.
	SNIFFING_INTERFACES: 'ens160'

	# SERVERNAME 
	# This should be the name/IP of the separate Master server:
	SERVERNAME: '172.16.81.127'
	# SSH_USERNAME
	# This should be the name of an account on the separate Master server.
	SSH_USERNAME: 'soniontest2' 

	# PF_RING Config. The default is 4096. High traffic networks may need to increase this.
	PF_RING_SLOTS: 4096 

	# IDS_ENGINE
	# Which IDS engine would you like to run?  snort/suricata
	# IDS_ENGINE='snort' or IDS_ENGINE='suricata'
	IDS_ENGINE: 'suricata' 
	# IDS_LB_PROCS. 
	#How many PF_RING load-balanced processes would you like to run? This value should be lower than your number of CPU cores.
	IDS_LB_PROCS: '3'
	# HOME_NET
	# Setup by default configures Snort/Suricata's HOME_NET variable
	HOME_NET: '192.168.1.1/16,10.0.0.0/8,172.16.0.0/12,200.16.0.0/16'

	# BRO_LB_PROCS
	# How many PF_RING load-balanced processes would you like Bro to run?
	# This value should be lower than your number of CPU cores.
	BRO_LB_PROCS: '3'

	# PCAP_OPTIONS
	# The default option here of '-c' is intended for low-volume environments.
	# If monitoring lots of traffic, you will want to remove the -c to use
	# netsniff-ng's default scatter/gather I/O or consider netsniff-ng's --mmap option. 
	#option: b <cpu>, --bind-cpu <cpu> Pin netsniff-ng to a specific CPU and also pin resp.
	PCAP_OPTIONS: '--mmap -b0'


	##########################################
	#Variables for file /opt/bro/etc/node.cfg
	##########################################
	#pin_cpus
	#Pin the BRO lb_procs processes to certain CPU cores.
	BRO_PIN_CPU: '5,6,7'


	##########################################
	#Variables for file suricata.yaml 
	##########################################
	# Tune cpu affinity of threads. Each family of threads can be bound on specific CPUs.
	set_cpu_affinity: 'yes'
	# management-cpu-set is used for flow timeout handling, counters
	management_cpu_set: '1'
	# receive-cpu-set is used for capture threads
	receive_cpu_set: '1'
	# worker-cpu-set is used for 'worker' threads
	worker_cpu_set: '2-4'


	##########################################
	#Variables for file sensor.conf
	##########################################
	#Set pfring as the load balancer (af-packet is the default)
	SURICATA_CAPTURE: 'pfring' #af-packet

	##########################################
	#Variables for file /etc/sysctl.conf
	##########################################
	#Configure Linux Disk Caching
	dirty_background_ratio: 50
	dirty_ratio: 80
	swappiness: 10

    ```
    
*  Ejecutar ansible sobre el servidor `"sonionforward"` (el username se define en la opcion extra_var):
    
    ```
    $ ansible-playbook -i hosts -l forward_nodes so_setup.yml --extra-var "target=sonionforward" --ask-become-pass
    ```
   Una vez ejecutado el comando se le solicitara el pass root para el servidor Forward y el pass del servidor Master.
   

## Referencias

* https://adamdehaven.com/blog/how-to-generate-an-ssh-key-and-add-your-public-key-to-the-server-for-authentication/
* https://docs.ansible.com/ansible/latest/modules/expect_module.html
* https://unix.stackexchange.com/questions/453859/when-i-use-ansible-module-expect-i-get-this-msg-the-pexpect-python-module-is-r
* https://stackoverflow.com/questions/35095481/how-to-find-list-membership-in-a-list-in-ansible

















