#<u>Práctica2: Aislamiento de una aplicación web.</u>
###Crear una mini-aplicación web y aislarlo en una jaula chroot.


En esta práctica vamos a familiarizarnos con el tipo de infraestructura virtual que se usa generalmente para dar un acceso limitado a una aplicación o un servicio tal como un servidor web. Vamos a centrarnos en el despliegue de una jaula, la gestión de permisos de los usuarios que pueden acceder a ella y las herramientas que debe disponer dicho sistema para dar soporte a una aplicación web, por lo que esta práctica no se centra en la aplicación en sí.


##La jaula

Para crear un sistema mínimo vamos a usar `debootstrap`con la siguiente orden y opciones:
	
	$ sudo debootstrap --arch=i386 saucy /home/jaulas/saucy/ http://archive.ubuntu.com/ubuntu

Las opciones corresponden a:

	deboostrap [arquitectura] [distribución] [directorio local] [dirección remota]	

Donde la arquitectura usada es de 32 bits, la distribución es una distribución de Ubuntu Saucy mínima, nuestra jaula la vamos a alojar en el directorio local /home/jaulas/saucy/  y vamos a descargar la distribución de los archivos oficiales de Ubuntu: [http://archive.ubuntu.com/ubuntu](http://archive.ubuntu.com/ubuntu). En esta dirección podemos comprobar la lista de distribuciones y sus versiones. Vemos en la siguiente captura como el sistema está montado.

![Jaula montada](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/jaula_montada.png)


Una vez creado el sistema podemos acceder a él como `root` para configurar las herramientas y dependencias necesarias para que funcione como servidor web. Para acceder a la jaula:

	$ sudo chroot /home/jaulas/saucy/

Vemos como el prompt ha cambiado y nos indica que hemos accedido como root a la jaula.


##Configurar aplicaciones y dependencias necesarias.
Para ejecutar nuestra aplicación web necesitaremos `apache2` y `php`.

##1. Apache2 

Para instalar apache de forma sencilla con apt-get:
	$ apt-get install apache2

Una vez instalado el servicio intenta arrancar sin éxito. Comprobando los mensajes al final de la instalación y parece que el `server name` no está definido y no es posible arrancar, también tenemos que resolver un conflicto con el puerto definido por defecto porque parece que ya está en uso.


![Apache2 error](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/apache_error.png)


Para solucionar este pequeño inconveniente, vamos a abrir el archivo de configuración de apache y a definir un nombre de servidor. Necesitamos un editor para poder editar el fichero de configuración, en nuestro caso vamos a instalar nano.

	$ apt-get install nano

Ahora sí:

	$ nano /etc/apache2/apache2.conf

Y al final del fichero escribimos:

	ServerName localhost

![ServerName apache](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/apache_servername.png)


Ahora vamos a modificar el puerto de escucha, para ello modificaremos el archivo de configuración de puertos y cambiar el puerto 80 por el 8080 para nuestro servidor:

	$ nano /etc/apache2/ports.conf
	Listen 8080


![Puerto apache2](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/puerto.png)



Ahora si podemos arrancar el servicio de apache2:
	
	$ service apache2 start 


Vemos que ahora no se muestra ningún error al iniciar el servicio. Para comprobar que todo funciona correctamente abrimos el explorador de internet y accederemos a [http://localhost:8080](http://localhost:8080). Se nos mostrará una página de inicio de apache2 como esta:

![Página inicio](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/pag_ini_apache.png)

#####Ya tenemos apache configurado y funcionando.


##2. PHP5

Lo que tendremos que hacer es instalar el paquete de php.

	$ apt-get install php5

Para comprobar que todo esta bien y php está funcionando podemos crear un pequeño fichero que nos muestre la configuración de php. En la ruta `/var/www/` creamos un fichero `php.info` que contendrá lo siguiente
	
	<?php phpinfo(); ?>

![Php info](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/php_info.png)


Reiniciamos el servidor apache para actualizar los cambios:
	
	$ service apache2 restart


Ahora desde el navegador vamos a acceder [http://localhost:8080/info.php](http://localhost:8080/info.php) y nos deberá mostrar la información de php y el servidor apache2.

![Php info2](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/php_info2.png)



###### Ahora ya tenemos todo configurado y preparado para que nuestra aplicación funcione.


##Los usuarios y sus permisos.
Una vez creada la jaula vamos a crear un usuario que pueda conectarse a ella. Este será el encargado de administrar la jaula pero no debe tener acceso al resto del sistema, solo y únicamente a la jaula.

* **¡Ojo!** Los usuarios los creamos desde el sistema completo, no desde la jaula.

Creamos el usuario mediante la siguiente orden:
	
	sudo useradd -s /bin/bash -m -d /home/jaulas/saucy/./home/p2 -c "Usuario Practica 2" -g users p2
	
Para restringir el usuario a la jaula editamos el fichero de configuración de ssh: 

	$ sudo gedit /etc/ssh/sshd_config

y añadimos estas líneas:

	Match User p2
		ChrootDirectory /home/jaulas/saucy/



![Ssh config](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/sshconfig.png)

Ahora el usuario `p2` puede conectarse a la jaula con ssh y comprobar que no tiene acceso fuera de ella. Lo vemos en la siguiente captura:

![Ssh conectado](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/ssh_conectado.png)

Como  podemos ver en la captura, el usuario está en la raíz de la jaula, y al intentar subir de directorio con `cd ..` se mantiene en el mismo directorio, no puede acceder a directorios fuera de la jaula.

Ahora damos permisos al usuario para que pueda trabajar en la aplicación que habrá que alojarla en `/var/www/`.

	
	

##La aplicación.

La aplicación web elegida para alojar en la jaula es la que realizamos en la [práctica 1](http://proyecto-ivejercicio14.rhcloud.com/), con algunos retoques actualizando contenidos y añadiendo funcionalidades de descarga de los materiales de clase.

La aplicación es un índice de los materiales que estamos usando durante la asignatura Infraestructura Virtual. Una vez creada la jaula, configurados los paquetes necesarios y dados permisos al usuario `p2` podemos alojar la aplicación, en nuestro caso en `/var/www` y acceder a ella mediante el navegador.

![Alojar aplicacion](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/copiar_aplicacion.png)


![Aplicacion](https://raw.github.com/rogegg/IV_Practica2/master/imagenes/aplicacion.png)



##Repositorio en GitHub
* [https://github.com/rogegg/IV_Practica2](https://github.com/rogegg/IV_Practica2)



##Bibliografía

* [http://jj.github.io/IV/documentos/temas/Tecnicas_de_virtualizacion](http://jj.github.io/IV/documentos/temas/Tecnicas_de_virtualizacion)
* [http://www.ubuntu-guia.com/](http://www.ubuntu-guia.com/)
* [http://www.ubuntu-es.org/node/6304](http://www.ubuntu-es.org/node/6304)
* [https://wiki.ubuntu.com/DebootstrapChroot](https://wiki.ubuntu.com/DebootstrapChroot)
* [http://planetubuntu.es/post/servidor-apache-en-ubuntu-instalacion-y-configuracion](http://planetubuntu.es/post/servidor-apache-en-ubuntu-instalacion-y-configuracion)


