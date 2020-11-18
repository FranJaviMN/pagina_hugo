---
title: "Instalacion Nginx Ovh"
date: 2020-11-18T18:21:55+01:00
draft: true
---

# Realizar la migración de la aplicación drupal que tienes instalada en el entorno de desarrollo a nuestro entorno de producción, para ello ten en cuenta lo siguiente:

## Tarea 1

La aplicación se tendrá que migrar a un nuevo virtualhost al que se accederá con el nombre portal.iesgn13.es.

De esta forma ya tenemos listo nuestro documentroot para nuestro nuevo virtualhost, ahora el siguiente paso es crear dicho virtualhost, para ello seguimos los pasos siguientes:
```shell
#### Copiamos el nuevo virtualhost llamado drupal.conf ####
debian@omega:/etc/nginx/sites-available$ sudo cp default ./drupal.conf

#### El contenido de drupal.conf debe de ser el siguiente ####
server {
    listen 80;

    ...

    root /srv/www/drupal;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html;

    server_name portal.iesgn13.es;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        #try_files $uri $uri/ =404;
        try_files $uri /index.php?$query_string;
    }

    # pass PHP scripts to FastCGI server
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        # With php-fpm (or other unix sockets):
        fastcgi_pass unix:/run/php/php7.3-fpm.sock;
    }
}
```

Ahora debemos de crear la zona DNS **CNAME** en nuestro proveedor OVH para que, al entrar en la url de **portal.iesgn13.es** debe de salirnos el instalador de drupal pero aun no debemos de instalarlo ya que debemos de configurar nuestra base de datos antes.


## Tarea 2

Vamos a nombrar el servicio de base de datos que tenemos en producción. Como es un servicio interno no la vamos a nombrar en la zona DNS, la vamos a nombrar usando resolución estática. El nombre del servicio de base de datos se debe llamar: bd.iesgn13.es.

Para ello debemos de crear una resolucion estatica de la siguiente forma:
```shell
127.0.0.1   bd.iesgn13.es
```

## Tarea 3

Por lo tanto los recursos que deberás crear en la base de datos serán (respeta los nombres):
* Dirección de la base de datos: bd.iesgn13.es
* Base de datos: bd_drupal
* Usuario: user_drupal
* Password: pass_drupal

Por lo que vamos a crear en nuestra maquina de ovh todos estos elementos para el cms drupal, por lo que debemos de crearlo:
```shell
#### Creamos la base de datos vacia ####
MariaDB [(none)]> CREATE DATABASE db_drupal;

#### Creamos el usuario con contraseña ####
MariaDB [(none)]> CREATE USER 'user_drupal'@'localhost' IDENTIFIED BY 'pass_drupal';

#### Le damos los privilegios de la base de datos al usuario ####
MariaDB [(none)]> GRANT ALL PRIVILEGES ON db_drupal.* to 'user_drupal'@'localhost';
```


## Tarea 4

Realiza la migración de la aplicación.

Ahora debemos de llevarnos nuestra base de datos del entorno de desarrollo a nuestra maquina ovh, para ello, mediante la utilidad de **scp** nos llevamos todo drupal a nuestra maquina de ovh. Una vez hayamos hecho esto, debemos de crear una copia de la base de datos de drupal de nuestro entorno de desarrollo, para ello usamos el siguiente comando:
```shell
#### Realizamos la copia de la base de datos ####
debian@cms:~$ sudo mysqldump drupal > copia-drupal.sql

#### Copiamos a nuestra maquina ovh ####
debian@cms:~$ sudo scp copia-drupal.sql debian@146.59.196.94:/home/debian/

#### Restauramos la copia de la base de datos ####
debian@omega:~$ sudo mysql db_drupal < copia-drupal.sql
```

Ahora debemos de llevarnos todo el contenido de la carpeta drupal del servidor de desarrollo al de produccion, por lo que volvemos a usar scp pero con la carpeta drupal:
```shell
debian@cms:~$ sudo scp -r /srv/drupal debian@146.59.196.94:/home/debian/
```

Una vez la tengamos, debemos de modificar el ficher llamado setting.php y buscar las lineas sobre la base de datos y debemos de introducir la nuevas credenciales de nuestra base de datos para que asi podamos usarla con drupal en esta maquina.


## Tarea 5

La aplicación debe estar disponible en la URL: portal.iesgn13.es

Ahora, si entramos en la url portal.iesgn13.es veremos que entraremos en drupal que hemos migrado desde el servidor de desarrollo hasta el servidor en produccion. Ademas vemos que tenemos las url limpias en nginx.

# Instalación / migración de la aplicación Nextcloud

## Tarea 1

Instala la aplicación web Nextcloud en tu entorno de desarrollo.

Para ello, en nuestro entorno de desarrollo debemos de crear, primero, un nuevo virtualhost que lo vamos a llamar nextcloud.conf el cual va a tener el siguiente contenido:
```shell
<VirtualHost *:80>
        ServerName www.francisco-nextcloud.org
        ServerAdmin webmaster@localhost
        DocumentRoot /srv/nextcloud
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <directory /srv/nextcloud>
        AllowOverride All
        </directory>


</VirtualHost>

```

Ahora, en nuestra documentroot vamos a descargar el cms de nextcloud mediando wget:
```shell
#### Lo descargamos ####
debian@cms:/srv$ sudo wget https://download.nextcloud.com/server/releases/nextcloud-20.0.1.tar.bz2

#### Lo descomprimimos ####
debian@cms:/srv$ sudo tar -xf nextcloud-20.0.1.tar.bz2
```

Despues de hacer nuestro nuevo virtualhost vamos a crear una nueva base de datos llamada nextcloud con privilegios del usuario **fran**.

Una vez lo tengamos lo descomprimimos y tendremos una carpeta llamada nextcloud. Cuando ya la tengamos nos vamos a nuestra pagina con resolucion estatica y nos saldra el instalador de nextcloud. En este paso es conveniente crear una carpeta en /srv/ la cual vamos a llamar datanextcloud que es donde vamos a guardar los ficheros de nuestro cloud.

Cuando hayamos completado la instalacion debemos de tener una pagina como la siguiente:

![pagina next cloud](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/php-nginx/nextcloud.png)



## Tarea 2

Realiza la migración al servidor en producción, para que la aplicación sea accesible en la URL: www.iesgnXX.es/cloud

Para ello debemos de crear una nueva base de datos y un nuevo usuario, seguiremos los prefijos que se han usado en la cracion de drupal:

* Dirección de la base de datos: bd.iesgn13.es
* Base de datos: bd_nextcloud
* Usuario: user_nextcloud
* Password: pass_nextcloud

```shell
#### Creamos la base de datos ####
MariaDB [(none)]> create database bd_nextcloud;

#### Creamos el usuario ####
MariaDB [(none)]> create user user_nextcloud@localhost identified by "pass_nextcloud";

#### Le damos los permisos ####
MariaDB [(none)]> GRANT ALL PRIVILEGES ON db_nextcloud.* to 'user_nextcloud'@'localhost';

#### Restauramos la base de datos ####
debian@omega:~$ sudo mysql bd_nextcloud < copia-nextcloud.sql
```

Ahora debemos de instalar las librerias de php que habiamos instalado en el servidor de desarrollo que son php-zip php-curl:
```shell
debian@omega:~$ sudo apt install php-zip php-curlcd
``` 

Ahora debemos mover la carpeta de nuestro entorno de desarrollo a produccion con el comando scp. Una vez la tengamos en producion la movemos a la carpeta de documentroot de **www.iesgn13.es** con el nombre de **cloud**:
```shell
debian@omega:/srv/www/iesgn13$ sudo cp -r /home/debian/nextcloud/ ./cloud
```

Ahora si intentamos entrar en la pagina **www.iesgn13.es/cloud** no nos dejara ya que debemos de modificar el fichero de configuracion que se encuentra en **config/config.php**, lo editamos poniendo las credenciales de nuestra base de datos para nextcloud. Una vez hecho esto debemos de dirigirnos a nuestro fichero de nuestro virtualhost de **www.iesgn13.es** y alli debemos de añadir la siguiente informacion, teniendo en cuenta que donde pone **/cloud** debemos de cambiarlo por el nombre del directorio del documentroot donde se encuentre la carpeta con nextcloud:
```shell
upstream php-handler {
	server unix:/run/php/php7.3-fpm.sock;
}

server {
	listen 80 default_server;
    listen [::]:80 default_server;

    ...

    add_header X-Content-Type-Options nosniff;
	add_header X-XSS-Protection "1; mode=block";
	add_header X-Robots-Tag none;
	add_header X-Download-Options noopen;
	add_header X-Permitted-Cross-Domain-Policies none;
	add_header Referrer-Policy no-referrer;

    fastcgi_hide_header X-Powered-By;

	location = /robots.txt {
		allow all;
		log_not_found off;
		access_log off;
	}

    location = /.well-known/carddav {	
		return 301 $scheme://$host:$server_port/nextcloud/remote.php/dav;	
	}

	location = /.well-known/caldav {
		return 301 $scheme://$host:$server_port/nextcloud/remote.php/dav;
	}

    location /.well-known/acme-challenge { }

	location ^~ /cloud {
        client_max_body_size 512M;
		fastcgi_buffers 64 4K;
		gzip on;
		gzip_vary on;
		gzip_comp_level 4;
		gzip_min_length 256;
		gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

        location /cloud {
			rewrite ^ /cloud/index.php;
		}

		location ~ ^\/cloud\/(?:build|tests|config|lib|3rdparty|templates|data)\/ {
			deny all;
		}	

		location ~ ^\/cloud\/(?:\.|autotest|occ|issue|indie|db_|console) {
			deny all;
		}

        location ~ ^\/cloud\/(?:index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+)\.php(?:$|\/) {
			fastcgi_split_path_info ^(.+?\.php)(\/.*|)$;
			set $path_info $fastcgi_path_info;
			try_files $fastcgi_script_name =404;
			include fastcgi_params;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			fastcgi_param PATH_INFO $path_info;
			fastcgi_param modHeadersAvailable true;
			fastcgi_param front_controller_active true;
			fastcgi_pass php-handler;
			fastcgi_intercept_errors on;
			fastcgi_request_buffering off;
		}

        location ~ ^\/nextcloud\/(?:updater|oc[ms]-provider)(?:$|\/) {
			try_files $uri/ =404;
			index index.php;
		}

        location ~ ^\/cloud\/.+[^\/]\.(?:css|js|woff2?|svg|gif|map)$ {
			try_files $uri /cloud/index.php$request_uri;
			add_header Cache-Control "public, max-age=15778463";
			add_header X-Content-Type-Options nosniff;
			add_header X-XSS-Protection "1; mode=block";
			add_header X-Robots-Tag none;
			add_header X-Download-Options noopen;
			add_header X-Permitted-Cross-Domain-Policies none;
			add_header Referrer-Policy no-referrer;
			access_log off;
		}

        location ~ ^\/cloud\/.+[^\/]\.(?:png|html|ttf|ico|jpg|jpeg|bcmap)$ {
			try_files $uri /cloud/index.php$request_uri;
			# Optional: Don't log access to other asse	
			access_log off;
		}
}

    root /srv/www/iesgn13;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name www.iesgn13.es;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}

    # Redireccion permanente para principal
	rewrite ^/$ principal permanent;

	
	# pass PHP scripts to FastCGI server
	#
	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
        # With php-fpm (or other unix sockets):
		fastcgi_pass unix:/run/php/php7.3-fpm.sock;
    }
}
```

Una vez hecho esto y al entrar en **www.iesgn13.es/cloud** veremos que entramos en el login de nextcloud por lo que ya tendriamos nuestro cloud funcionando.

## Tarea 3

Instala en un ordenador el cliente de nextcloud y realiza la configuración adecuada para acceder a "tu nube".

Para obtener el cliente de nextcloud debemos de entrar en la pagina de nextcloud y bajarnos el cliente para nuestra maquina, en mi caso para una maquina linux aunque vemos que podemos obtenerlo para otros sistemas operativos incluso para moviles.

Una vez tengamos el fichero del cliente nextcloud, lo ejecutamos, en mi caso debemos de cambiar los permisos a ejecutable y ejecutar el fichero **.AppImage** como si fuera un script. Una vez este ejecutando nos saldra una ventana en la que tendremos que seleccionar la opcion de **log in** y ponemos la url de nuestro servidor nextcloud, en mi caso es **http://www.iesgn13.es/cloud**, nos abrira el navegador y nos mostrara un mensaje de autorizacion a nuestra maquina, aceptamos y nos vamos de nuevo a la ventana del cliente. Si vemos que ya estamos configurando el lugar donde queremos tener los fichero de nuestro cloud, aceptamos y ya tendriamos listo el cliente nextcloud. Tendremos en nuestra barra de tareas un simbolo de una nube, que indica que estamos ya conectados a nextcloud.

![nextcloud cliente](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/php-nginx/cliente-nextcloud.png)
