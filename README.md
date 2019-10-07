### COMANDO PARA CORRER
ansible-playbook -i hosts -l master so_setup.yml  -vvv
ansible-playbook -i hosts -l forward_nodes so_setup.yml  -vvv






# Ansible para la instalacion de nodos FORWARD y MASTER para Security Onion.

## Tabla de contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Instrucciones para el despliegue de un nodo Forward](#Instrucciones-para-el-despliegue-de-un-nodo-Forward)



## Pre-requisitos

1. Instalacion de las maquinas virtuales, con la ultima version de Security Onion:
    
    LINK DE REPO DE TINCHO.    

2. Agregar clave SSH publica del dispositivo desde el cual se realiza el despliegue en las maquinas vituales con Security Onion,
   (no efectuar ninguna operacion sobre el usuario ROOT), un tutorial ejemplo:


```
https://adamdehaven.com/blog/how-to-generate-an-ssh-key-and-add-your-public-key-to-the-server-for-authentication/

```

## Instrucciones para el despliegue de un nodo Forward

*  Agregar nombre de usuario del nodo forward al archivo hosts en el gurpo forward_nodes (ejemplo user sonionforward):

    ```
    [forward_nodes]
    sonionforward
    ```
    
*  En la carpeta host_vars agregar un archivo yml (nombre_usuario.yml) en la que se especifiquen las siguiente variables (sonionforward.yml):
   
    ```
        ansible_host: '172.16.81.126'
        ansible_user: 'sonionforward'
        
        MGMT_INTERFACE: 'ens160'
        MGMT_CONFIG_TYPE: 'static'
        ADDRESS: '172.16.81.126'
        NETMASK: '255.255.255.0'
        GATEWAY: '172.16.81.1'
        NAMESERVER: '200.16.16.1 200.16.16.2 8.8.8.8'
        DOMAIN: 'sonioneco.local.psi' 
        
        SNIFFING_INTERFACES: 'ens160'
        
        SERVERNAME: '172.16.81.127'
        SSH_USERNAME: 'soniontest2' 
        
        PF_RING_SLOTS: 4096 
         
        IDS_ENGINE: 'suricata' 
        IDS_LB_PROCS: '3'
        HOME_NET: '192.168.1.1/16,10.0.0.0/8,172.16.0.0/12,200.16.0.0/16'
        
        BRO_LB_PROCS: '3'
        
        PCAP_OPTIONS: '--mmap -b0'
        
        BRO_PIN_CPU: '5,6,7'
        
        set_cpu_affinity: 'yes'
        management_cpu_set: '1'
        receive_cpu_set: '1'
        worker_cpu_set: '2-4'
        
        SURICATA_CAPTURE: 'pfring' #af-packet
        
        dirty_background_ratio: 50
        dirty_ratio: 80
        swappiness: 10
    ```
    
*  Ejecutar ansible sobre el servidor "soniontest" (el username se define en extra var):
        ```
    $ ansible-playbook -i hosts -l forward_nodes so_setup.yml --extra-var "target=soniontest" --ask-sudo-pass
    ```
   Una vez ejecutado el comando se le solicitara el pass root para el servidor Forward y el pass del servidor SONIONMASTER

## Referencias

* https://github.com/TheHive-Project/TheHiveDocs/blob/master/admin/webhooks.md#configuration
* https://github.com/TheHive-Project/TheHiveHooks
* https://github.com/cybergoatpsyops/TheHive-SideProjects
* https://github.com/TheHive-Project/TheHive4py/blob/master/thehive4py/api.py
* https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-18-04
* https://securityonion.readthedocs.io/en/latest/hive.html

















