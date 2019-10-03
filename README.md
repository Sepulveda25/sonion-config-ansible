# Uso basico de Ansible

## Instalar ultima version de ansible

sudo apt-add-repository ppa:ansible/ansible   
sudo apt-get update
sudo apt install ansible

### Especificar inventarios (direcciones y dominios a desplegar)

Existe un inventario global por defecto en el archivo /etc/ansible/hosts, se modifica y se agregan los host a desplegar, o bien, se puede crear un txt para hacer un inventario local :

```bash
[all:vars]
master_ip=XXX.XXX.XXX.XXX

[forward_nodes]
172.16.81.71 

[master]
172.16.81.70 ##para pasar contraseña agrego: ansible_sudo_pass=csirt

[storage_nodes]
```
### Prueba de alcance y conexion de hosts con comandos ad-hoc

```bash
ansible nombrehost (o all) -m ping          ##-m es para indicar el modulo que vamos a usar
ansible all -m ping --ask-pass -u csirt     ##para probar un ping en el server de prueba
##por defecto el modulo que se usa es el shell
```

### Comandos sueltos

Por ejemplo, instalar programa usando apt:

```bash
ansible hostobjetivo -m apt -a 'name=vim state=present' -b -K   ##instalo el vim y me aseguro q este presente, -b es para cambiar el usuario a root, -K es para pasar la contraseña si pide
```
### Playbook ejemplo para la ejecución anterior

playbook ejemplo.yml 

```bash
	---                     //indica que es un yml
	hosts: all              //o un grupo, o un especifico
	//become: true          //especificar el become aca indica que todo el playbook se ejecute como root
	remote_user: username	
	tasks:
		-name: instalar vim
		  apt: name=vim state=present
		  become: true
```

### Ejecutar un playbook pasando usuario y contraseña por linea de comandos

```bash
ansible-playbook nombre_playbook.yml -i inventory.ini --user=username \--extra-vars "ansible_sudo_pass=yourPassword"  ##Siendo inventory.ini un inventario, username un nombre de usuario y 'yourPassword' la contraseña
```

### Usar vault para encriptar variables,passwords, keys, etc:

```bash
~/.ansible$ ansible-vault create my_vault.yml   #crea el vault
su_password: <myreallyspecialpassword>          #Guardar los datos
```

Ahora la contraseña será encriptada y la unica forma de verla será con:

```
ansible-vault edit my_vault.yml
```

Para usarla en el playbook, se usa el comando:
```
ansible-playbook myawesome_playbook.yml --ask-vault-pass  ##ask-vault-pass especifica buscar la contraseña en el vault
```

El playbook queda:

```
---
- name: My Awesome Playbook
  hosts: remote
  become: yes

  vars_files:
    - ~/.ansible/my_vault.yml 

  vars:
    ansible_become_pass: '{{ su_password }}'

  roles:
      - some_awesome_role
```

### Escribir comando para conectar con un usario remotamente

```
ansible -i hosts.txt webservers -m ping -u nombreuser
```


### COMANDO PARA CORRER

ansible-playbook -i hosts -l forward_nodes so_setup.yml  -vvv




















