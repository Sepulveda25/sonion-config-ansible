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
	LOG_SIZE_LIMIT: '100000000000'

	# IDS_RULESET
	# Which IDS ruleset would you like to use?
	# Emerging Threats Open (no oinkcode required): ETOPEN
	# Emerging Threats PRO (requires ETPRO oinkcode): ETPRO
	# Sourcefire Talos (requires Talos oinkcode): TALOS
	# TALOS and ET (requires TALOS oinkcode): TALOSET
	IDS_RULESET: 'ETOPEN'

	# IDS_ENGINE
	# Which IDS engine would you like to run?  snort/suricata
	# Whatever you choose here will apply to the master server
	# and then sensors inherit this setting from the master server.
	# To run Snort:
	# IDS_ENGINE='snort'
	# To run Suricata:
	# IDS_ENGINE='suricata'
	IDS_ENGINE: 'suricata'


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