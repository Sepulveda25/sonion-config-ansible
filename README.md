### COMANDO PARA CORRER
ansible-playbook -i hosts -l master so_setup.yml  -vvv
ansible-playbook -i hosts -l forward_nodes so_setup.yml  -vvv






# Ansible para la instalacion de nodos FORWARD y MASTER para Security Onion.

## Tabla de contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Instrucciones para el despliegue](#instrucciones-para-el-despliegue)



## Pre-requisitos

1. Instalacion de las maquinas virtuales, con la ultima version de Security Onion:
    
    LINK DE REPO DE TINCHO.    

2. Agregar clave SSH publica del dispositivo desde el cual se realiza el despliegue en las maquinas vituales con Security Onion,
   (no efectuar ninguna operacion sobre el usuario ROOT), un tutorial ejemplo:

```
https://adamdehaven.com/blog/how-to-generate-an-ssh-key-and-add-your-public-key-to-the-server-for-authentication/

```

3. Darle permisos de sudo al usuario creado del nodo Forward o Master. En el nodo Forward o Master hacer:

    *  $ sudo visudo
    * Agregar la siguiente linea al final del archivo (ejemplo nodo forward):
    ```
    soniontest ALL=(ALL) NOPASSWD: ALL
    ```
    * Una vez finalizada el despliegue de Ansible la linea agregada anteriormente debe ser borrada.

## Instrucciones para el despliegue

*  Despliegue de un nodo forward:

    * Agregar nombre de usuario del nodo forward al archivo hosts en el gurpo forward_nodes:
    ```
    [forward_nodes]
    soniontest
    ```
    * En la carpeta host_vars agregar un archivo en la que se especifiquen las siguiente variables:
    
    ```
ansible_host: '172.16.81.126'
ansible_user: 'soniontest'

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
    
    
    *  Nodo master

1.  Pegar carpeta webhooks en home y acceder a dicha carpeta.
2.  Eliminar carpeta `webhooksenv` en caso de existir.
3. Crear entorno virtual:
    <br />`$ python3.6 -m venv webhooksenv`
4.  Activar entorno virtual:
	<br />`$ source webhooksenv/bin/activate`
5.  Instalar librerias Flask, Gunicorn, Wheel, Request y Netaddr:
	<br />`$ pip install wheel`
	<br />`$ pip install gunicorn`
	<br />`$ pip install flask`
	<br />`$ pip install requests`
    <br />`$ pip install netaddr`
    <br />`$ pip install thehive4py`
6.  Salir del entorno virtual:
    <br />`$ deactivate`
7.  Permitir acceso puerto 5000:
	<br />`$ sudo ufw allow 5000`
8.  Dentro de la carpeta webhooks modificar el archivo `parametros.py` con los valores de `hiveUR`L y `hookURL` correspodientes.
9.  Copiar archivo `webhooks.service` en `/etc/systemd/system/`
10. Iniciar el servicio:
	<br />`$ sudo systemctl start webhooks.service`
11. Comprobar el estado del servicio:
	<br />`$ sudo systemctl enable webhooks.service`
12. Modificar TheHive para que envie acciones al ENDPOINT HTTP creado, agregando al archivo `/etc/thehive/application.conf`:

```
webhooks {
  myLocalWebHook {
    url = "http://my_HTTP_endpoint/webhook"
  }
}
```


## Referencias

* https://github.com/TheHive-Project/TheHiveDocs/blob/master/admin/webhooks.md#configuration
* https://github.com/TheHive-Project/TheHiveHooks
* https://github.com/cybergoatpsyops/TheHive-SideProjects
* https://github.com/TheHive-Project/TheHive4py/blob/master/thehive4py/api.py
* https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-18-04
* https://securityonion.readthedocs.io/en/latest/hive.html

















