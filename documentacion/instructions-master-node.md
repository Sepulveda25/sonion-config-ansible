## Instrucciones para el despliegue de un servidor Master

## Tabla de contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Template Forward Node](#template-forward-node)
3. [Despliegue con Ansible](#despliegue-con-ansible)


## Pre-requisitos

Se utilizo Ansible 2.8.4 o superior.

1. Instalar pexpect en nuestro host:

    ```
    sudo apt install python-pexpect
    ```
    ```
    sudo apt install python3-pexpect


2. Contar con un servidor con la ISO de Security Onion instalada o Ubuntu Server 16.04: [Repositorio con instrucciones para la instalacion de Security Onion](https://gitlab.unc.edu.ar/csirt/csirt-docs)

3. Agregar clave SSH publica del host desde el cual se realiza el despliegue en el servidor con Security Onion, para hacer esto se puede copiar manualmente la public key del host en el archivo autorized_keys de la carpeta /home/USUARIOMASTER/.ssh o con el comando: ssh-copy-id USUARIOMASTER@IPFORWARD (tambien ejecutado desde el host).

4. [Opcional] Contar con un servidor con InfluxDB y Grafana. Los servidores Master y Forwards configurados con Ansible seran integrados con Grafana. 
   Comprobar configuracion de archivo: `roles/telegraf_install/files/telegraf.conf`
 
   Este paso es opcional, en caso de no contar con el servidor con InfluxDB y Grafana instalados setear a la variable
   `INSTALL_TELEGRAF: 'no'` en lugar de  `INSTALL_TELEGRAF: 'yes'`



## Template Master Node

*  En la carpeta `host_vars` copiar un archivo .yml que tiene la forma del archivo `template_master.yml` (que se encuentra en la carpeta Documentacion), y renombrar por ejemplo `sonionmaster.yml`, luego modificar las variables de configuracion para el despliegue del Master Node (El nombre que le damos a `template_master.yml` luego es agregado en el grupo correspondiente en el archivo `hosts`, por ejemplo si renombramos al archivo como `sonionmaster.yml` en el archivo host agregamos en el grupo Master a  `sonionmaster` - esto se reliza en pasos posteriores - ).
    
   

Dentro del archivo `template_master.yml` tenemos las siguientes variables:


- `ansible_host` y `ansible_user` corresponden a la IP y Username del host objetivo (el Master Node).

```yaml
ansible_host: '172.16.81.127'
ansible_user: 'soniontest2'
```

- La seccion `Hostname variable` define el hostname que tendra el Server Forward:
 
```yaml
    
    HOST_NAME: 'csirt-sonion-master'
        
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
ADDRESS: '172.16.81.127'
NETMASK: '255.255.255.0'
GATEWAY: '172.16.81.1'
NAMESERVER: '200.16.16.1 200.16.16.2 8.8.8.8'
DOMAIN: 'sonionmaster.local.psi' 
```

En la variable `LOG_SIZE_LIMIT` se configura la cantidad de disco que puede utilizar Elastic.

```yaml
# LOG_SIZE_LIMIT
# This setting controls how much disk space Elastic uses.
# 100GB = 100000000000
# LOG_SIZE_LIMIT='100000000000'
LOG_SIZE_LIMIT: '100000000000'
```

En la variable `IDS_RULESET` se indica el ruleset de reglas que utilizara el motor de IDS.

```yaml
# IDS_RULESET
# Which IDS ruleset would you like to use?
# Emerging Threats Open (no oinkcode required): ETOPEN
# Emerging Threats PRO (requires ETPRO oinkcode): ETPRO
# Sourcefire Talos (requires Talos oinkcode): TALOS
# TALOS and ET (requires TALOS oinkcode): TALOSET
IDS_RULESET: 'ETOPEN'
```

En la variable `IDS_ENGINE` se indica el motor IDS (Suricata o Snort) que se utilizara en los Forwards Nodes.

```yaml
# IDS_ENGINE
# Which IDS engine would you like to run?  snort/suricata
# Whatever you choose here will apply to the master server
# and then sensors inherit this setting from the master server.
# To run Snort:
# IDS_ENGINE='snort'
# To run Suricata:
# IDS_ENGINE='suricata'
IDS_ENGINE: 'suricata'
```


En las siguientes variables se definen caracteristicas que tiene que ver con el uso del almacenamiento. <br/>
La variable `WARN_DISK_USAGE` indica a partir de que porcentaje de uso de disco se comenzara a emitir alertas. <br/>
La variable `CRIT_DISK_USAGE` indica a partir de que porcentaje de uso de disco se comenzara a eliminar archivos viejos. <br/>
La variable `DAYSTOKEEP` indica la cantidad de dias que se deben retener los datos en la base de SGUIL. <br/>
La variable `DAYSTOREPAIR` indica en caso de falla la longitud en dias de los datos a restaurar de la base de SGUIL. <br/>


```yaml
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

- La seccion `Install Telegraf` indica si se llaveara a cabo la integracion del host con Telegraf. 
  En caso de ser 'yes' la variable `INSTALL_TELEGRAF`,  un servidor con InfluxDB y Grafana. 
  Comprobar configuracion de archivo: `roles/telegraf_install/files/telegraf.conf`

```yaml
INSTALL_TELEGRAF: 'yes' #'no'
```        

- La seccion `Configuring firewall for analyst for administration purposes` indica si se deben habilitar 
  los puertos 22,443 y 7734 para el acceso de un analista desde una red determinada.
  En caso de ser 'yes' la variable `allow_analyst_network`, se debe indicar en la variable `analyst_network`
  la red desde la que accedera el analista.

```yaml
allow_analyst_network: 'yes' #no
analyst_network: '172.16.81.0/24' #IP address (or CIDR range like 172.16.81.0/24)
```

        
## Despliegue con Ansible


`[IMPORTANTE]` Como se indico de forma detallada en la seccion pre-requistos se debe:  
    
    1. Pegar el public key de nuestro host en el Master Node 

`[IMPORTANTE]` Cuando se ejecute el comando para el despliegue del Ansible se le solicitara el pass de SUDO (BECOME_PASSWORD o SUDO_PASSWORD) para el usuario del servidor Master, en este caso tenemos tres alternativas:
   * Ingresar el password en caso de conocerlo.
   * Si el usuario es root presionar enter y no ingresar nada (presionar enter).
   * O en caso de no cumplirse ninguna de las opciones anteriores se le debe permitir ejecutar sudo sin solicitar la contraseña, esto se hace agregando una entrada al archivo sudoers (sudo visudo), la entrada es: USUARIOMASTER ALL=(ALL) NOPASSWD: ALL (lo mismo se puede realizar creando un archivo temporal en /etc/sudoers.d/temporal_USUARIOMASTER y agregando la misma linea). En este caso no se debe ingresar contraseña (presionar enter).

  `[IMPORTANTE]`  Tambien se solicitara el ingreso de una contraseña para un usuario `admincsirt` que se va a crear en el Master Node, a este usuario se puede ingresar para propositos de administracion una vez completado el despliegue. 

Agregar el nombre con el que renombramos el archivo `template_master.yml` al archivo `hosts` en el grupo `[master]`. (Ej. como en el paso anterior se crea el archivo `sonionmaster.yml` agregar `sonionmaster` en el archivo `hosts`):

```yaml
    [master]
    sonionmaster
    
```

Ejecutar Ansible sobre `"sonionmaster"` que se define en `extra-var`:

```bash
     $ ansible-playbook -i hosts -l master so_setup.yml --extra-var "target=sonionmaster" --ask-become-pass
```


