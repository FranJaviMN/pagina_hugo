---
title: "Nginx Ovh"
date: 2020-11-09T11:11:39+01:00
draft: true
---

# Instalación de un servidor LEMP

Esta practica se va a realizar en el servidor de OVH que nos han proporcionado, en mi caso, el servidor es **omega.iesgn13.es**. A esta maquina se va a acceder con las claves privadas que tenemos en dicha maquina.

```shell
francisco@debian10:~/.ssh$ ssh -i id_rsa debian@omega.iesgn13.es
Enter passphrase for key 'id_rsa': 
Linux omega 4.19.0-11-cloud-amd64 #1 SMP Debian 4.19.146-1 (2020-09-17) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Oct 30 12:53:39 2020 from 89.37.78.58
debian@omega:~$ 
```

## Tarea 1

En esta primera tarea debemos de instalar un servidor web **nginx**, para ello debemos de usar el siguiente comando:
```shell
debian@omega:~$ sudo apt install nginx
```

De esta forma ya tendriamos nuestro servidor web instalado, de momento no vamos a hacer nada mas con dicho servdor.

## Tarea 2

Instala un servidor de base de datos MariaDB. Ejecuta el programa necesario para asegurar el servicio, ya que lo vamos a tener corriendo en el entorno de producción.

Primero lo que debemos de hacer es instalar los paquetes asociados a la base de datos **Mariadb** que son *mariadb-server* y *mariadbclient*. Una vez los tengamos instalados, como este entorno **LEMP** lo vamos a poner en produccion, debemos de asegurar el servicio de base de datos que hemos instalado:
```shell
#### Instalamos los paquetes asociados a Mariadb ####
debian@omega:~$ sudo apt install mariadb-server mariadb-client

#### Comprobamos el estado del servicio de mariadb ####
debian@omega:~$ sudo systemctl status mariadb.service 
● mariadb.service - MariaDB 10.3.25 database server
   Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-11-05 08:29:18 UTC; 14min ago
```

Ya hemos instalado y comprobado que nuestro servicio de mariadb esta activo, ahora lo que debemos de hacer es asegurar nuestro servicio de base de datos con el comando **mysql_secure_installation** con lo que conseguiremos:
* Estableces una contraseña fuerte de root
* Eliminar los usuarios anonimos
* Deshabilitar el inicio de sesión remoto para el usuario root
* Eliminar la base de datos de prueba y el acceso a ella
```shell
debian@omega:~$ sudo mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

...

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

...

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!

...

Remove anonymous users? [Y/n] y
 ... Success!

...

Disallow root login remotely? [Y/n] y
 ... Success!

...

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

...

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.
```

## Tarea 3

Instala un servidor de aplicaciones php php-fpm.

Para ello solo debemos instalar el paquete **php-fpm**:
```shell
debian@omega:~$ sudo apt install php-fpm
```

# Tarea 4

Crea un virtualhost al que vamos acceder con el nombre www.iesgnXX.es. Recuerda que tendrás que crear un registro CNAME en la zona DNS.

para poder crear nuestro registro **CNAME** debemos de irnos al panel de control de OVH y seleccionar la zona **DNS**. Una vez estemos en la configuracion de nuestro DNS de ovh solo debemos de añadir un nuevo registro CNAME.

Primero vamos a crear el nuevo virtualhost, por lo que podemos seguir las indicaciones de la tarea de [servidor de nginx](https://franjavimn.onrender.com/servicios/practica-servidor-nginx/) donde podemos encontrar, en la tarea 2 de la practica, como se crea un nuevo virtualhost:

```shell
#### Creamos el nuevo virtualhost ####
debian@omega:/etc/nginx/sites-available$ sudo cp default ./iesgn13.conf

#### Contenido del fichero iesgn13.conf ####
server {
      listen 80 default_server;
      listen [::]:80 default_server;
      root /srv/www/iesgn13;

      # Add index.php to the list if you are using PHP
      index index.html index.htm index.nginx-debian.html;

      server_name www.iesgn13.es;

      location / {
            # First attempt to serve request as file, then
            # as directory, then fall back to displaying a 404.
            try_files $uri $uri/ =404;
      }
}

#### Creamos el enlace simbolico ####
debian@omega:/etc/nginx/sites-available$ sudo ln -s /etc/nginx/sites-available/iesgn13.conf /etc/nginx/sites-enabled/

#### Comprobamos la sintaxis de nuestro fichero iesgn13.conf ####
debian@omega:/etc/nginx/sites-available$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful


#### Reiniciamos el servicio de nginx ####
debian@omega:/etc/nginx/sites-available$ sudo systemctl restart nginx.service

#### Cambiamos los permisos para www-data ####
debian@omega:/etc/nginx/sites-available$ sudo chown -R www-data:www-data /srv/
```

Ahora vamos a crear nuestro registro CNAME en nuestro proveedor OVH, por lo que tendremos que borrar el registro de tipo **TXT** con subdominio **www.iesgn13.es** y con valor **"3|welcome"**. Una vez lo tengamos borrado solo debemos de modificar el registro CNAME con subdominio **www.iesgn13.es** y con destino **iesgn13.es** el cual tenemos que cambiar por **omega.iesgn13.es**.

Cuando tengamos cambiado nuestro registro **CNAME**, si entramos en www.iesgn13.es debe de aparecernos la siguiente pagina:

![pagina ovh](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/ovh/prueba-ovh.png)

## Tarea 5

Cuando se acceda al virtualhost por defecto default nos tiene que redirigir al virtualhost que hemos creado en el punto anterior.

## Tarea 6

Cuando se acceda a www.iesgnXX.es se nos redigirá a la página www.iesgnXX.es/principal

Para ello, en nuestro fichero **iesgn13.conf** vamos a añadir la redireccion para cumplir con la tarea, para ello vamos a añadir un nuevo directorio en **/srv/www/iesgn13/** llamado **/principal**:
```shell
#### Creamos la carpeta principal con un fichero de bienvenida ####
debian@omega:/etc/nginx/sites-available$ sudo mkdir /srv/www/iesgn13/principal
debian@omega:/etc/nginx/sites-available$ sudo nano /srv/www/iesgn13/principal/index.html

#### Le cambiamos de propietario a los nuevos ficheros creados ####
debian@omega:/etc/nginx/sites-available$ sudo chown -R www-data:www-data /srv/

#### Lineas a añadir en el fichero de iesgn13.conf ####
rewrite ^/$ principal permanent;
```

Por lo que al entrar en la url de nuestro sitio web **www.iesgn13.es** nos va a redirigir a **www.iesgn13.es/principal** y nos mostrara el mensaje que nosotros le hayamos puesto:

![redireccion](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/ovh/redireccion-ovh.png)

## Tarea 7

En la página www.iesgnXX.es/principal se debe mostrar una página web estática (utiliza alguna plantilla para que tenga hoja de estilo). En esta página debe aparecer tu nombre, y una lista de enlaces a las aplicaciones que vamos a ir desplegando posteriormente.

Para ello he usado una plantilla muy sencilla del siguiente [enlace](https://html5-templates.com/preview/html5-page-layout.html)

Lo descargamos y lo editamos en nuestra maquina local, lo editamos para luego subirlo a nuestra maquina ovh mediante **scp**.
```shell
#### Subimos a nuestra maquina ovh la plantilla ####
francisco@debian10:~$ scp Descargas/html5-page-layout/html5-page-layout.tar.gz debian@omega.iesgn13.es:/home/debian/

#### Comprobamos que tenemos el archivo comprimido ####
debian@omega:~$ ls
html5-page-layout.tar.gz

#### Lo descomprimimos y copiamos los fichero a /srv/www/iesgn13/principal ####
debian@omega:~$ tar -xzvf html5-page-layout.tar.gz

#### Movemos a nuestro directorio /srv/www/iesgn13/principal ####
debian@omega:~$ sudo mv style.css /srv/www/iesgn13/principal/
debian@omega:~$ sudo mv index.html /srv/www/iesgn13/principal/

#### Cambiamos el propietario de los nuevos ficheros ####
debian@omega:~$ sudo chown -R www-data:www-data /srv/
```

Ahora, si accedemos al sitio www.iesgn13.es veremos que ha cambiado y ahora tenemos una pagina estatica donde podremos ir colgando las distintas aplicaciones que iremos desplegando.

## Tarea 8

Configura el nuevo virtualhost, para que pueda ejecutar PHP. Determina que configuración tiene por defecto php-fpm (socket unix o socket TCP) para configurar nginx de forma adecuada.

Ahora si queremos empezar a servir aplicaciones php debemos de habiliar el interprete de php que hemos instalado llamado **php-fpm**. Si entramos en nuestro fichero de **iesgn13.conf** veremos que hay unas lineas comentadas de la siguiente forma:
```shell
# pass PHP scripts to FastCGI server
      #
      #location ~ \.php$ {
      #     include snippets/fastcgi-php.conf;

      #    # With php-fpm (or other unix sockets):
      #     fastcgi_pass unix:/run/php/php7.3-fpm.sock;
      #     # With php-cgi (or other tcp sockets):
      #     fastcgi_pass 127.0.0.1:9000;
      #}
```

En la cual si leemos vemos que, si vamos a usar php debemos de descomentar la linea *location* e *include* y, si seguimos leyendo vemos que nos dice que si estamos usando **php-fpm** descomentemos la linea siguiente. Por lo que podemos ver que **php-fpm** trabaja con socket unix. El fichero de configuracion de nuestro virtualhost quedaria de la siguiente manera:
```shell
server {
      listen 80 default_server;
      listen [::]:80 default_server;

      ...

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

      location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php7.3-fpm.sock;
      }
}

#### Reiniciamos el servicio ####
debian@omega:~$ sudo systemctl reload nginx.service
```

Ahora, para comprobar que podemos servir ficheros php, vamos a crear un fichero llamado **info.php** en el cual vamos a añadir un pequeño script para ver si funciona, dicho fichero lo vamos a crear en **/srv/www/iesgn13/**:
```shell
#### Creamos el fichero info.php ####
debian@omega:~$ sudo nano /srv/www/iesgn13/info.php

#### Añadimos el script ####
<?php phpinfo();
```

Entonces al entrar en **www.iesgn13.es/info.php** nos saldra la siguiente pagina:

![php info](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/ovh/php-nginx.png)






