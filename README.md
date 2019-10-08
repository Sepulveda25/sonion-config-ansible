# Ansible para la instalacion de nodos FORWARD y MASTER para Security Onion.

## Tabla de contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Instrucciones para el despliegue de un nodo Master](#instrucciones-para-el-despliegue-de-un-nodo-master)
3. [Instrucciones para el despliegue de un nodo Forward](#instrucciones-para-el-despliegue-de-un-nodo-forward)
4. [Referencias](#referencias)


## Pre-requisitos

1. Instalacion de las maquinas virtuales, con la ultima version de Security Onion:
    
    LINK DE REPO DE TINCHO.    

2. Agregar clave SSH publica del dispositivo desde el cual se realiza el despliegue en las maquinas vituales con Security Onion,
   (no efectuar ninguna operacion sobre el usuario ROOT).



## Instrucciones para el despliegue de un nodo Master

*  Agregar nombre de usuario del nodo master al archivo `host` en el grupo `master` (Ej. user sonionmaster):

    ```
    [master]
    sonionmaster
    ```
    
*  En la carpeta `host_vars` agregar un archivo yml (`nombre_usuario.yml`) en la que se especifiquen las variables
   del archivo template_master.yml (Ej. sonionmaster.yml):
   
    ```
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
        LOG_SIZE_LIMIT: '75000000000'
        
        # IDS_RULESET
        # Which IDS ruleset would you like to use?
        # Emerging Threats Open (no oinkcode required): ETOPEN
        # Emerging Threats PRO (requires ETPRO oinkcode): ETPRO
        # Sourcefire Talos (requires Talos oinkcode): TALOS
        # TALOS and ET (requires TALOS oinkcode): TALOSET
        IDS_RULESET: 'ETOPEN'
        
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
    ```
    
*  Ejecutar ansible sobre el servidor `"sonionmaster"` (el username se define en la opcion extra_var):
    
    ```
    $ ansible-playbook -i hosts -l master so_setup.yml --extra-var "target=sonionmaster" --ask-become-pass
    ```
   Una vez ejecutado el comando se le solicitara el pass root para el servidor.
   

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
        SSH_USERNAME: 'soniontest' 
        
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
        pin_cpu: '5,6,7'
        
        
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

















