## Instrucciones para el despliegue de un nodo Forward

## Tabla de contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Template Foward Node](#template-foward-node)
3. [Referencias](#referencias)

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

## Template

    
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
        
        #########################################################
        #Install Telegraf 
        #(Modify File roles/telegraf_install/files/telegraf.conf)
        #########################################################
        
        INSTALL_TELEGRAF: 'yes' #'no'
        
        
        #########################################################
        #Copy Rules TheHive to ElastAlert in Master Node
        #(From file /roles/securityonion_setup_master/files/thehive_rules)
        #########################################################
        
        COPY_THEHIVE_RULES: 'yes' #'no'
        
        #IP del servidor con TheHive
        hive_host: 'http://172.16.81.110'
        #Puerto del servicio TheHive
        hive_port: '9000'
        #API-KEY del user creado en TheHive
        hive_apikey: 'xLuixewzDP6zXcVgLvyaFUoeUZcZgKu4'

    ```
    
*  Ejecutar ansible sobre el servidor `"sonionforward"` (el username se define en la opcion extra_var):
    
    ```
    $ ansible-playbook -i hosts -l forward_nodes so_setup.yml --extra-var "target=sonionforward" --ask-become-pass
    ```
   Una vez ejecutado el comando se le solicitara el pass root para el servidor Forward y el pass del servidor Master.
   
   
*  Agregar nombre de usuario del nodo forward al archivo `hosts` en el grupo `forward_nodes` (Ej. user sonionforward):

    ```
        [forward_nodes]
        sonionforward
        
    ```
