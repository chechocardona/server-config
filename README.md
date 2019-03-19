# server-config        
              
## 1. Instalación de Entorno de Desarrollo
	
Una vez su aplicación Angular haya sido probada localmente, y quiera hacerla accesible a través de Internet, una buena opción es contratar un servicio de Servidor Virtual Privado (VPS). La opción que elija debe permitirle instalar Ubuntu Linux 14.04 / 16.04 LTS (64-bits), o Mac OS 10.12. Previamente a instalar el entorno de desarrollo aségurese de crear un usuario no root. En ubuntu ejecute el siguiente comando:
	
	adduser username
		
Se le instará a configurar y confirmar un nuevo password. Hágalo y confirme:
	
	Adding user `username' ...
	Adding new group `username' (1001) ...
	Adding new user `username' (1001) with group `username' ...
	Creating home directory `/home/username' ...
	Copying files from `/etc/skel' ...
	New password:
	Retype new password:
	passwd: password updated successfully
		
Una vez haya configurado un password se le instará a configurar la información de usuario. Si quiere dejar esta información en blanco presione Enter para aceptar la información por defecto.
	
	Changing the user information for username
	Enter the new value, or press ENTER for the default
		Full Name []:
		Room Number []:
		Work Phone []:
		Home Phone []:
		Other []:
	Is the information correct? [Y/n]
	
Para añadir el nuevo usuario al grupo sudo corra el comando usermod:
	
	usermod -aG sudo username
		
Ingrese con la identidad de nuevo usuario corriendo:
	
	su - username
	
Para realizar la instalación de los prerrequisitos siga los pasos explicados en los tutoriales de Hyperledger Composer: ['Installing pre-requisites'](https://hyperledger.github.io/composer/v0.19/installing/installing-prereqs) y ['Installing the development environment'](https://hyperledger.github.io/composer/v0.19/installing/development-tools).
	
## 2. Modificación de URLs
	
Antes de desplegar la aplicación Angular al Servidor Web es necesario hacer algunas modificaciones para asegurarse de que funcione correctamente. Seguramente en su versión local las URLs de los REST Servers tienen la forma:
	
	http://localhost:3000
	
Esta dirección deber reemplazarse por el Nombre del Dominio que haya configurado para su sitio Web. Por ejemplo, si su Dominio es: w<span>ww.</span>certificates.com, debería cambiar la URL del REST server en el código para que haga peticiones a la siguiente URL:
	
	http://www.certificates.com:3000
		
Debe reemplazar todas las URLs que hagan llamados al localhost por su Dominio.
	
## 3. Creación de un 'Production Build' en Angular
	
La forma más simple de hacerlo es correr el siguiente comando en la carpeta del proyecto
	
	ng build --prod --base-href /<project_name>/
	
Los archivos de salida se almacenarán en la carpeta dist. Estos archivos deberán ser copiados a la carpeta del servidor que los requiera para hacer el sitio web visible.
	
## 4. Instalación de Apache
	
A continuación se explica la instalación y configuración Apache Server. No es la única opción de servidor pero nos permite explicar los pasos para hacer la aplicación visible a internet.
	
Para instalar apache en Ubuntu corra el comando:
	
	sudo apt-get install apache2
		
	
