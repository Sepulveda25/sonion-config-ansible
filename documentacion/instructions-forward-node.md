## Instrucciones para el despliegue de un nodo Forward

## Tabla de contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Template Forward Node](#template-forward-node)
3. [Despliegue con Ansible](#despliegue-con-ansible)

## Pre-requisitos

1. Instalar pexpect en nuestro host:

    ```
    sudo apt install python-pexpect
    ```
    ```
    sudo apt install python3-pexpect
    ```

2. Contar con un servidor con la ISO de Security Onion instalada o Ubuntu Server 16.04: [Repositorio con instrucciones para la instalacion de Security Onion](https://gitlab.unc.edu.ar/csirt/csirt-docs)
   Se debe contar con al menos dos interfaces una para administracion y otra para realizar el monitoreo. 

3. Agregar clave SSH publica del host desde el cual se realiza el despliegue en el servidor con Security Onion, para hacer esto se puede copiar manualmente la public key del host en el archivo autorized_keys de la carpeta /home/USUARIOFORWARD/.ssh o con el comando: ssh-copy-id USUARIOFORWARD@IPFORWARD (tambien ejecutado desde el host).

4. El host (desde el que realizamos el despliegue) tambien debe tener conexion con el Master Node, para ello repetir el paso 3 para el Master Node sobre el usuario del Master que es el mismo que configuraremos mas adelante en el template (SSH_USERNAME: 'USERMASTER') - Esto se hace ya que se crea localmente en el host un par de claves ssh (RSA), y se distribuye en el Master la public key y en el Forward la private key para la comunicacion entre ellos al momento de ejecutar el comando SOSETUP y ademas se utiliza para pegar las reglas de TheThive en el Master correspondientes al Forward (esto es en caso de estar habilitada la opcion copiar reglas de TheHive - se configura mas adelante).

5. Si el usuario del Master mencionado en el paso anterior (SSH_USERNAME) va a ser el usuario `root` saltar este paso, caso contrario en el servidor MASTER, al usuario USERMASTER (que se define en SSH_USERNAME del template - se configura mas adelante ) se le debe permitir ejecutar sudo sin solicitar la contraseña, esto se hace agregando una entrada al archivo sudoers (sudo visudo), la entrada es: USERMASTER ALL=(ALL) NOPASSWD: ALL (lo mismo se puede realizar creando un archivo temporal en /etc/sudoers.d/temporal_USERMASTER y agregando la misma linea).

6. Mantener actualizado el archivo `/roles/securityonion_setup_master/files/clasiffication_rules` con la clasificacion de reglas del Forward node.

7. [Opcional] Contar con un servidor con InfluxDB y Grafana. Los servidores Master y Forwards configurados con Ansible seran integrados con Grafana. 
   Comprobar configuracion de archivo: `roles/telegraf_install/files/telegraf.conf`
 
   Este paso es opcional, en caso de no contar con el servidor con InfluxDB y Grafana instalados setear a la variable
   `INSTALL_TELEGRAF: 'no'` en lugar de  `INSTALL_TELEGRAF: 'yes'`

8. [Opcional] Si se cuenta con servidores de The Hive instalados (uno principal -o master- y otro para la dependencia -o secundario-) es posible copiar para ambos The Hive (principal y secundario) las reglas en el Master de Security Onion (en el Master de Security Onion se crearan dos carpetas con reglas para cada The Hive). Es necesario mantener actualizadas las reglas de TheHive que seran copiadas en el Master Node. Las reglas actualizadas se encuentran en el [Repositorio con The Hive Rules](https://gitlab.unc.edu.ar/csirt/elastalert-thehive) y deben guardarse en `/roles/copy_hive_rules_to_master/files/thehive_rules`. Mas adelante en el template del Forward se definen las variables, para cada TheHive:
    *  hive_host
    *  hive_port
    *  hive_apikey
   
   Este paso es opcional, en caso de no contar con el servidor con TheHive instalado setear a las variables en el archivo template Forward de la carpeta `host_vars`: 
   `COPY_THEHIVE_RULES_MASTER: 'no'` en lugar de `COPY_THEHIVE_RULES_MASTER: 'yes'` para el nodo principal o master de TheHive.
   `COPY_THEHIVE_RULES_SECONDARY: 'no'` en lugar de `COPY_THEHIVE_RULES_SECONDARY: 'yes'` para el nodo secundario y de dependencia.


## Template Forward Node

*  En la carpeta `host_vars` copiar un archivo .yml que tiene la forma del archivo `template_forward.yml` (que se encuentra en la carpeta Documentacion), y renombrar por ejemplo `sonionforward.yml`, luego modificar las variables de configuracion para el despliegue del Forward Node (El nuevo nombre que le damos a `template_forward.yml` luego es agregado en el grupo correspondiente en el archivo `hosts`, por ejemplo, si renombramos al archivo como `sonionforward.yml` en el archivo host agregamos en el grupo Forward  `sonionforward` - esto se reliza en pasos posteriores - ).

Dentro del archivo `template_forward.yml` tenemos las siguientes variables:

- `ansible_host` y `ansible_user` corresponden a la IP y Username del host objetivo (el Forward Node).

```yaml
    ansible_host: '172.16.81.126'
    ansible_user: 'sonionforward'
```

- La seccion `Hostname variable` define el hostname que tendra el Forward Node:
 
```yaml
    
    HOST_NAME: 'csirt-sonion-forward'
        
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
    SNIFFING_INTERFACES: 'ens192'
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

La variable `PF_RING_SLOTS` indica la cantidad de slots de pf_ring.

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

En las siguientes variables se definen caracteristicas que tiene que ver con el uso del almacenamiento. <br/>
La variable `WARN_DISK_USAGE` indica a partir de que porcentaje de uso de disco se comenzara a emitir alertas. <br/>
La variable `CRIT_DISK_USAGE` indica a partir de que porcentaje de uso de disco se comenzara a eliminar archivos viejos. <br/>


```yaml
# WARN_DISK_USAGE
# Begin warning when disk usage reaches this level
WARN_DISK_USAGE: '80'
# CRIT_DISK_USAGE
# Begin purging old files when disk usage reaches this level
CRIT_DISK_USAGE: '90'
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
   Si se cuenta con servidores de The Hive instalados (uno principal -o master- y otro para la dependencia -o secundario-) es posible copiar para ambos The Hive (principal y secundario) las reglas en el Master de Security Onion (en el Master de Security Onion se crearan dos carpetas con reglas para cada The Hive). Es necesario mantener actualizadas las reglas de TheHive que seran copiadas en el Master Node. Las reglas actualizadas se encuentran en el [Repositorio con The Hive Rules](https://gitlab.unc.edu.ar/csirt/elastalert-thehive) y deben guardarse en `/roles/copy_hive_rules_to_master/files/thehive_rules`. Mas adelante en el template del Forward se definen las variables, para cada TheHive:
    *  hive_host
    *  hive_port
    *  hive_apikey
   
   Este paso es opcional, en caso de no contar con el servidor con TheHive instalado setear a las variables en el archivo template Forward de la carpeta `host_vars`: 
   `COPY_THEHIVE_RULES_MASTER: 'no'` en lugar de `COPY_THEHIVE_RULES_MASTER: 'yes'` para el nodo principal o master de TheHive.
   `COPY_THEHIVE_RULES_SECUNDARY: 'no'` en lugar de `COPY_THEHIVE_RULES_SECUNDARY: 'yes'` para el nodo secundario y de dependencia.
   
   
   
   En caso de setear la variable `COPY_THEHIVE_RULES` como 'yes' sera necesario contar con un servidor con TheHive 
   instalado y mantener actualizadas las reglas de TheHive que seran copiadas en el Master Node desde el Forward Node. 
   Las reglas actualizadas se encuentran en el [Repositorio con The Hive Rules](https://gitlab.unc.edu.ar/csirt/elastalert-thehive) 
   y deben guardarse en `/roles/copy_hive_rules_to_master/files/thehive_rules`, la ejecucion del Ansible modificara las reglas automaticamente
   y le asignara las variables `hive_host` (IP del servidor de TheHive), `hive_port` (puerto del servicio de TheHive) y `hive_apikey` 
   (api key del usuario creado por el Admin de TheHive), por ello dentro de las reglas no hay que realizar ninguna modificacion manualmente.

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

Se utilizo Ansible 2.8.4 o superior.

`[IMPORTANTE]` Como se indico de forma detallada en la seccion pre-requistos se debe:  
    
    1. Pegar el public key de nuestro host en el Forward Node.
    2. Tener conexion con el master Master node desde nuestro host.
    3. Permitir al usuario del Master (SSH_USERNAME: 'USERMASTER') ejecutar sudo sin solicitar el password (en caso de que no se trate del usuario root en el Master).  
    
`[IMPORTANTE]` Cuando se ejecute el comando para el despliegue del Ansible se le solicitara el pass de SUDO (BECOME_PASSWORD o SUDO_PASSWORD) para el usuario del servidor Forward, en este caso tenemos tres alternativas:
   * Ingresar el password en caso de conocerlo.
   * Si el usuario es root presionar enter y no ingresar nada (presionar enter).
   * O en caso de no cumplirse ninguna de las opciones anteriores se le debe permitir ejecutar sudo sin solicitar la contraseña, esto se hace agregando una entrada al archivo sudoers (sudo visudo), la entrada es: USUARIOFORWARD ALL=(ALL) NOPASSWD: ALL (lo mismo se puede realizar creando un archivo temporal en /etc/sudoers.d/temporal_USUARIOCREADO y agregando la misma linea). En este caso no se debe ingresar contraseña (presionar enter). 

`[IMPORTANTE]`  Tambien se solicitara el ingreso de una contraseña para un usuario `admincsirt` que se va a crear en el Forward Node, a este usuario se puede ingresar para propositos de administracion una vez completado el despliegue. Tambien esta contraseña se utilizara para la creacion de un usuario en el Master que se utiliza para integrar el nodo Forward y el Master (esto se realiza de forma automatica)( en caso de existir el usuario en el Master Node lo que se hace es chequear que la contraseña ingresada coincida con el user del Master).


*  Agregar nombre de usuario del nodo forward al archivo `hosts` en el grupo `[forward_nodes]` 
(Ej. como en el paso anterior se crea el archivo `sonionforward.yml` agregar `sonionforward` en el archivo `hosts`):

```yaml
    [forward_nodes]
    sonionforward
```
    
*  Ejecutar Ansible sobre el servidor `"sonionforward"` (el username se define en la opcion extra_var):
    
```bash
    $ ansible-playbook -i hosts -l forward_nodes so_setup.yml --extra-var "target=sonionforward" --ask-become-pass
```
    


