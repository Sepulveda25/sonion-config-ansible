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

Se indica el nombre de la interfaz de monitoreo `SNIFFING_INTERFACES`, esta interfaz debe estar conectado a un port mirror del trafico a monitorear.

```yaml
    # Which interface(s) will be sniffing network traffic?
    # For multiple interfaces, please separate them with spaces.
    SNIFFING_INTERFACES: 'ens160'
```


Las variable `SERVERNAME` y `SSH_USERNAME` corresponden a la direccion IP y Username del Master Node. 

```yaml
    # SERVERNAME 
    # This should be the name/IP of the separate Master server:
    SERVERNAME: '172.16.81.127'
    # SSH_USERNAME
    # This should be the name of an account on the separate Master server.
    SSH_USERNAME: 'soniontest2' 
```

La variable `PF_RING_SLOTS` indica la cabtidad de slots de pf_ring. ###???????????????????

```yaml
    # PF_RING Config. The default is 4096. High traffic networks may need to increase this.
    PF_RING_SLOTS: 4096
```

Configuracion del motor IDS, se selecciona que IDS (suricata o snort) se utilizara mediante la variable `IDS_ENGINE`.
La variable `IDS_LB_PROCS` designa la cantida de de procesos del IDS, este debe ser un valor menor al numero de CPUs. 
La variable `HOME_NET` especifica las direcciones IPs privadas del trafico a monitorear.

```yaml
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
```

Configuracion del motor IDS BRO, la variable `BRO_LB_PROCS` indica la cantidad de procesos del BRO,
este debe ser un valor menor al numero de CPUs.  

```yaml
    # BRO_LB_PROCS
    # How many PF_RING load-balanced processes would you like Bro to run?
    # This value should be lower than your number of CPU cores.
    BRO_LB_PROCS: '3'
```

Configuracion de netsniff-ng, la variable `PCAP_OPTIONS` permite configurar opciones para netsniff-ng.

```yaml
    # PCAP_OPTIONS
    # The default option here of '-c' is intended for low-volume environments.
    # If monitoring lots of traffic, you will want to remove the -c to use
    # netsniff-ng's default scatter/gather I/O or consider netsniff-ng's --mmap option. 
    #option: b <cpu>, --bind-cpu <cpu> Pin netsniff-ng to a specific CPU and also pin resp.
    PCAP_OPTIONS: '--mmap -b0'
```

- La seccion `Variables for file /opt/bro/etc/node.cfg` asigna los procesos de BRO a cada uno de los CPUs inidicados en la variable `BRO_PIN_CPU`. 
  La cantidad de CPUs inidicados debe ser menor o igual al numero de `BRO_LB_PROCS` definido en la seccion `Variables for file sosetup_forward.conf`.

```yaml
    #pin_cpus
    #Pin the BRO lb_procs processes to certain CPU cores.
    BRO_PIN_CPU: '5,6,7'
```

- La seccion `Variables for file suricata.yaml` asigna los procesos de suricata a los CPU indicados. <br/>
  La variable `set_cpu_affinity` indica si queremos asignacion de los procesos a los CPUs indicados
  en las variables que se encuentran debajo de esta, puede ser 'yes' o 'no'. <br/>
  En la variable `management_cpu_set` se indica a que CPU se le asigna el proceso management de Suricata. <br/>
  En la variable `receive_cpu_set` se indica a que CPU se le asigna el proceso receive de Suricata. <br/>
  En la variable `worker-cpu-set` indica a que CPUs se les asignan los workers de Suricata, 
  esta variable puede ser un rango de CPUs como por ejemplo 2-4 (incluye los CPUs 2,3,4) 
  y la cantidad de CPUs indicados en esta variable debe ser menor o igual a `IDS_LB_PROCS`
  definido en la seccion `Variables for file sosetup_forward.conf`.

```yaml
    # Tune cpu affinity of threads. Each family of threads can be bound on specific CPUs.
    set_cpu_affinity: 'yes'
    # management-cpu-set is used for flow timeout handling, counters
    management_cpu_set: '1'
    # receive-cpu-set is used for capture threads
    receive_cpu_set: '1'
    # worker-cpu-set is used for 'worker' threads
    worker_cpu_set: '2-4'
```

- La seccion `Variables for file sensor.conf` indica el modo de captura del IDS (af-packet o pfring) en la variable `SURICATA_CAPTURE`.

```yaml
    #Set pfring as the load balancer (af-packet is the default)
    SURICATA_CAPTURE: 'pfring' #af-packet
```

- La seccion `Variables for file /etc/sysctl.conf` modifican las variables `dirty_background_ratio`, `dirty_ratio` y `swappiness` del archivo /etc/sysctl.conf.

```yaml
    #Configure Linux Disk Caching
    dirty_background_ratio: 50
    dirty_ratio: 80
    swappiness: 10
```

- La seccion `Install Telegraf` indica si se llaveara a cabo la integracion del host con Telegraf. 
  En caso de ser 'yes' la variable `INSTALL_TELEGRAF`,  un servidor con InfluxDB y Grafana. 
  Comprobar configuracion de archivo: `roles/telegraf_install/files/telegraf.conf`

```yaml
   INSTALL_TELEGRAF: 'yes' #'no'
```        

-  La seccion `Copy Rules TheHive to ElastAlert in Master Node` indica si las reglas para TheHive ubicadas en 
  `/roles/securityonion_setup_master/files/clasiffication_rules` seran copiadas en el Master Node.
   En caso de setear la variable `COPY_THEHIVE_RULES` como 'yes' sera necesario contar con un servidor con TheHive 
   instalado y mantener actualizadas las reglas de TheHive que seran copiadas en el Master Node desde el Forward Node. 
   Las reglas actualizadas se encuentran en el [Repositorio con The Hive Rules](https://gitlab.unc.edu.ar/csirt/elastalert-thehive) 
   y deben guardarse en `/roles/copy_hive_rules_to_master/files/thehive_rules`, la ejecucion del Ansible modificara las reglas automaticamente
   y le asignara las variables `hive_host` (IP del servidor de TheHive), `hive_port` (puerto del servicio de TheHive) y `hive_apikey` 
   (api key del usuario creado por el Admin de TheHive). 

```yaml
        COPY_THEHIVE_RULES: 'yes' #'no'
        #IP del servidor con TheHive
        hive_host: 'http://172.16.81.110'
        #Puerto del servicio TheHive
        hive_port: '9000'
        #API-KEY del user creado en TheHive
        hive_apikey: 'xLuixewzDP6zXcVgLvyaFUoeUZcZgKu4'
```        


## Despliegue con Ansible

*  Agregar nombre de usuario del nodo forward al archivo `hosts` en el grupo `[forward_nodes]` 
(Ej. como en el paso anterior se crea el archivo `sonionforward.yml` agregar `sonionforward` en el archivo `hosts`):

    ```yaml
    [forward_nodes]
    sonionforward

*  Ejecutar Ansible sobre el servidor `"sonionforward"` (el username se define en la opcion extra_var):
    
    ```bash
    $ ansible-playbook -i hosts -l forward_nodes so_setup.yml --extra-var "target=sonionforward" --ask-become-pass
    ```
   Una vez ejecutado el comando se le solicitara el pass root para el servidor Forward y el pass del servidor Master.


