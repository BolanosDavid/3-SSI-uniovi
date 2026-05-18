Para conectar host con maquina virtual utilizando NAD, 
se crea una regla de reenvio de red: A partir de un puerto, todas las peticiones
que se hagan a este puerto en la maquina host, se redirecciona a
la maquina virtual.Es decir, si se hace un SSH a mi mismo, se conectará
con la máquina virtual

Estructura básica de carpetas:

/etc/:
	ssh
	 apach
	 vasftd
	apt/ sources.list <- para instalar cosas
	passwd
	 shadow
	 sodoers
/home/:
	ssiuser
/bin/:
	programas
/var/
	logs



MobaXterm -> Sesion -> Conexion SSH -> localhost 

	-FIREWALL -> UFW
		  +Crear reglas en iptable
		  +Para conflictos entre reglas, aplicará siempre la primera que encuentre
		  (Se puede cambiar el orden). (Al crear una regla, podemos insertarla donde
		  nos apetezca con el comando insert). Para eliminar, también podemos
		  hacerlo por número
		  +Para crear reglas utilxizando nombres (SSH, vsftd...), requiere
		  que se hayan instalado previamente. Si no se hiciera, aparecerá 
		  un error que menciona la falta de Perfiles



Comandos interesantes para Linux
-su: cambios de usuario (Si quieres volver al inicial Exit)
-ssh
-ufw
-id lista los distintos grupos y permisos que tiene el usuario

sudo grep -i "ssiuser" /var/log/auth.log| grep -i "password\|passwd\|changed\|updated"
sudo  -> permisos de superusuarios
grep -> Buscar
-i 
"ssiuser" /var/log/auth.log -> direccion del log de autenticacion (iniios de sesion, cambios de contraseña...)
| -> pipeline
grep -> buscar en lo que recibe de antes
-i
"password\|passwd\|changed\|updated" -> todos los cambios con contraseñas


sudo lynis audit system > lynis.txt
hardening index -> indica lo segura que es mi maquina