# Configuración de Servidor para Certificados Académicos       
              
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
## 3. Creación de Credenciales de Api de Google plus

Para realizar la autenticación de usuarios la aplicación utiliza el Api de Google plus. Debido a que la URL de redirección cambia por el Nombre de Dominio que se defina en su servidor, es necesario actualizar las credenciales o crear unas. Para solicitar credenciales del Api de Google navegue a esta ['dirección'](https://console.developers.google.com/). Navegue a la pestaña 'Biblioteca', busque 'Google Plus' y habilite el API, como se muestra en las imágenes.

![Consola](img/Consola.png?raw=true "Consola")

![GooglePlusApi](img/GooglePlusApi.png?raw=true "GooglePlusApi")

Ahora navegue a la pestaña 'Credenciales' y seleccione de la lista desplegable 'Crear Credenciales' la opción 'ID de Cliente de OAuth'. De clic en la opción Web y llene los datos solicitados, como se muestra en la imagen. En 'Nombre', escriba el nombre de su aplicación. En 'Orígenes de Javascript Autorizados', escriba la url de su aplicación, por ejemplo en este caso sería: http://www.certificates.com. Y en 'URIs de redirección autorizados' escriba su url de redirección, por ejemplo: http://www.certificates.com/auth/google/callback

![GenerateCredentials](img/GenerateCredentials.png?raw=true "GenerateCredentials")

Al dar clic en 'Crear' se generarán unas nuevas credenciales de cliente que se deberán utilizar más adelante en la configuración de los REST-servers. Copie el 'ID de Cliente' y el 'Secreto de Cliente' y guárdelos para más adelante.

## 4. Creación de un 'Production Build' en Angular
	
La forma más simple de hacerlo es correr el siguiente comando en la carpeta del proyecto
	
	ng build --prod --base-href /<project_name>/
	
Los archivos de salida se almacenarán en la carpeta dist. Estos archivos deberán ser copiados a la carpeta del servidor que los requiera para hacer el sitio web visible.
	
## 5. Instalación de Apache
	
A continuación se explica la instalación y configuración Apache Server. No es la única opción de servidor pero nos permite explicar los pasos para hacer la aplicación visible a internet.
	
Para instalar apache en Ubuntu corra el comando:
	
	sudo apt-get install apache2

Para permitirle al servidor soportar enlaces profundos ('deep links') y añadir un proxy en caso de que la aplicación lo requiera, se debe modificar uno de los archivos de configuración de Apache. En Ubuntu navegue al archivo que se indica a continuación y modifíquelo según las instrucciones
	
	etc/apache2/sites-available/000-default.conf
	
Remueva el comentario de la línea:
		
	#ServerName www.example.com
	
Y añada su nombre de Dominio, por ejemplo
	
	ServerName www.certificates.com
		
Posterioremente vaya a la línea

	DocumentRoot /var/www/html

Y reescríbala de esta forma, reemplazando <project_name> con el nombre que haya definido previamente

	Alias /<project_name> /var/www/<project_name>
	<Directory /var/www/<project_name>>
	     Options All
	     AllowOverride All
	     order allow,deny
	     allow from all

	RewriteEngine On
	RewriteBase /<project_name>/
	Options +FollowSymLinks

	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^(.*)$ index.html [L,QSA]

	</Directory>

Ubíquese justo antes de la parte final del archivo donde se encuentra la etiqueta de final

	</VirtualHost>

Y agruegue las siguientes líneas que definen el comportamiento del proxy. Reemplace <project_name> por el nombre de proyecto que haya definido

	RewriteEngine On
	RewriteCond %{REQUEST_METHOD} OPTIONS
	RewriteRule ^(.*)$ $1 [R=200,L]

	RedirectMatch ^/$ /<project_name>/

	ProxyPreserveHost On

	<Proxy *>
		Order allow,deny
		Allow from all
	</Proxy>
	ProxyPass /auth http://localhost:3000/auth
	ProxyPassReverse /auth http://localhost:3000/auth

	ProxyPass /api http://localhost:3000/api
	ProxyPassReverse /api http://localhost:3000/api

Lo anterior es necesario en el caso de que su aplicación Angular defina una configuración de proxy en el archivo proxy.conf.js. Para este caso se definen para las peticiones a /api y /auth que se deben redirigir al REST server por el puerto 3000

## 6. Habilitar la aplicación

Copie los archivos generados en el paso 4 a la carpeta correspondiente en el servidor Apache. Hágalo ejecutando el siguiente comando en la carpeta del repositorio de su aplicación Angular

	sudo cp -r dist/* /var/www/casaur/

Reinicie el servidor ejecutando:

	sudo /etc/init.d/apache2 restart
	
## 7. Habilitar REST-Servers

Siga los pasos descritos en el repositorio ['https://github.com/Blockchain4openscience/casaur-frontend'](https://github.com/Blockchain4openscience/casaur-frontend) para desplegar Fabric ('Deployment of Hyperledger Fabric onto a single-organization') e interactuar con el modelo de negocio través de los REST-servers (Interacting with the business network using the REST server). Tenga especial atención en reemplazar los campos de la variable COMPOSER_PROVIDERS por el ID de Cliente ('clientID') y el Secreto de Cliente ('clientSecret') creados en el paso 3 de este tutorial. Esto es:

	export COMPOSER_PROVIDERS='{    "google": {        "provider": "google",        "module": "passport-google-oauth2",        "clientID": <Reemplace por su ID de cliente>,        "clientSecret": <Reemplace por su Secreto de Cliente>,        "authPath": "/auth/google",        "callbackURL": "/auth/google/callback",        "scope": "https://www.googleapis.com/auth/plus.login",        "successRedirect": "http://localhost:4200/callback",        "failureRedirect": "/"    }}'
	
Cree al menos un participante administrador para acceder a la aplicación
	
	composer participant add -c admin@casaur -d 		'{"$class":"org.degree.Administrator","email":"block4opsc@gmail.com","firstName":"Juan","lastName": "Admin","publicKey": "adminPK"}'
	
Ahora está todo listo para que navegue a su URL e interactúe con la aplicación desde cualquier lugar usando un navegador web.

	
