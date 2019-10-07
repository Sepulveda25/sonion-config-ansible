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


 

    
''


3. s
4. 

## Instrucciones para el despliegue

(Para ver detalles del desarollo ver archivo `Instrucciones de desarollo` en carpeta Documentacion).


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

















