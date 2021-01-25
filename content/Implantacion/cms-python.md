---
title: "Cms Python"
date: 2021-01-22T20:22:10+01:00
draft: true
---

# Instalación de aplicación web python

En esta tarea vamos a desplegar un CMS python. Para ello he elegido de la pagina https://djangopackages.org/grids/g/cms/ el cms llamado **Mezzanine** y lo vamos a desplegar en el entorno de produccion con **uwsgi**.

## Creacion en el entorno de desarrollo

Lo primero que debemos de hacer es desplegar nuestra aplicacion en nuestro entorno de desarrollo para asi, mediante un repositorio en github, podamos llevarnos todo el despliegue a nuestra maquina en desarrollo, en este caso va a ser la maquina **Quijote**.

Lo primero que debemos de hacer es crear un entorno virtual python en el cual vamos a estar instalando los paquetes necesarios para que no interfieran con los paquetes de nuestra maquina por lo que debemos de crear el entorno de desarrollo:
```shell
#### Instalamos los paquetes necesarios ####
francisco@debian10:~$ sudo apt install python3-venv

#### Creamos el entorno virtual ####
francisco@debian10:~/Documentos/Implantacion/Python/practica-cms$ python3 -m venv mezzanine

#### Iniciamos el entorno virtual ####
francisco@debian10:~/Documentos/Implantacion/Python/practica-cms$ source mezzanine/bin/activate
(mezzanine) francisco@debian10:~/Documentos/Implantacion/Python/practica-cms$

#### Instalamos el paquete llamado mezzanine ####
(mezzanine) francisco@debian10:~/Documentos/Implantacion/Python/practica-cms$ pip install mezzanine

#### Creamos un nuevo projecto de mezzanine ####
(mezzanine) francisco@debian10:~/Documentos/Implantacion/Python/practica-cms$ mezzanine-project mezzanine_openstack

#### Entramos en la carpeta del projecto y creamos la base de datos ####
(mezzanine) francisco@debian10:~/Documentos/Implantacion/Python/practica-cms/mezzanine_openstack$ python3 manage.py createdb

#### Corremos el servidor localmente ####
(mezzanine) francisco@debian10:~/Documentos/Implantacion/Python/practica-cms/mezzanine_openstack$ python3 manage.py runserver
```

Hecho esto ya tenemos la aplicacion corriendo en nuestro equipo, para ver su contenido nos dirigimos a la url que nos indique en la consola, en mi caso http://localhost:8000/ y nos debe de salir algo como lo siguiente:

![imagen principal produccion](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/mezzanine/desarrollo-mezzanine.png)


## Personalizando la pagina

Ya que la tenemos lista en nuestro entorno de desarrollo, vamos a personalizar nuestra pagina y le vamos a cambiar el nombre a esta para ponerle el nuestro, para ello debemos de irnos a la zona de admin con url [admin](http://localhost:8000/admin/login/?next=/admin/) y ahi ponemos las credenciales que anteriormente en el comando **python3 manage.py createdb** habiamos introducido por lo que entraremos en la zona de administracion:

![zona de admin desarrollo](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/mezzanine/admin-mezzanine-desarrollo.png)

Una vez ahi, debemos de irnos a la pestaña donde dice **Settings** y ahi debemos de buscar la zona donde pone **Site** y ahi tendremos dos campos, uno en el que viene una pequeña descripcion de la pagina y en la otra el nombre de la pagina. Lo editamos a nuestro gusto y volvemos a la pagina de inicio donde ya deberiamos de ver los cambios:

![cambios en desarrollo](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/mezzanine/cambios-desarrollo-mezanine.png)

Despues de esto debemos de realizar una copia de la base de datos para llevarnosla a nuestro entorno de produccion donde nosotros la cargaremos en nuestra base de datos **mariadb**, por lo que vamos a ejecutar la copia de la base de dtaos en el entorno de desarrollo:
```shell
#### Realizamos la copia de la base de datos ####
(mezzanine) francisco@debian10:~/Documentos/Implantacion/Python/practica-cms/python-cms-mezzanine/mezzanine_openstack$ python manage.py dumpdata > copia_db.json
```

Despues de esto solo debemos de llevarnoslo a nuestro repositorio de git:
```shell
#### Añadimos todos los ficheros que tenemos ####
francisco@debian10:~/Documentos/Implantacion/Python/practica-cms/python-cms-mezzanine$ git add mezzanine_openstack/

#### Añadimos los commits que deseemos y lo subimos al repositorio ####
francisco@debian10:~/Documentos/Implantacion/Python/practica-cms/python-cms-mezzanine$ git push
```

## Despliegue en produccion

Para ello vamos a hacer primero los preparativos con la base de datos mariadb ya que, esta base de datos se encuentra en otra maquina, por lo que debemos de crear en nuestra maquina con la base de datos un usuario especifico para esta aplicacion que vamos a usar, en mi caso la maquina es **Sancho** y su ip es **10.0.1.8**:
```shell
#### Creamos el usuario con una contraseña ####
MariaDB [(none)]> CREATE USER 'mezzanine'@'10.0.2.3' IDENTIFIED BY 'mezzanine';
Query OK, 0 rows affected (0.100 sec)
```

Una vez hecho esto debemos de asegurarnos de que nuestra base de datos mariadb es capaz de ser accedida desde la ip donde tenemos nuestra aplicacion, por lo que debemos de dirigirnos a nuestro fichero de configuracion **/etc/mysql/mariadb.conf.d/50-server.cnf** y vemos que la linea *bind-address* esta de la siguiente forma:
```shell
#### Comprobamos que esta correcto ####
bind-address            = 0.0.0.0

#### Reiniciamos el servicios de mariadb ####
root@sancho:~# systemctl restart mariadb
```

De esta forma ya tendremos la base de datos accesible desde nuestra maquina **quijote**, por lo que vamos a hacer una prueba desde quijote para ver que podemos acceder, aunque debemos de tener en cuenta que tenemos que tener instalado un paquete llamado mariadb para poder realizar la conexion a la base de datos remota:
```shell
#### Instalamos (si no lo tuvieramos) el paquete mariadb ####
[centos@quijote ~]$ sudo dnf install mariadb

#### Hacemos la comprobacion ####
[root@quijote ~]# mysql -u mezzanine -p -h bd.franjavier.gonzalonazareno.org
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 37
Server version: 10.3.25-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

Vemos que sin problemas hemos podido conectar, por lo que ya podemos seguir con el despliegue en produccion.

## Instalacion de paquetes necesarios

Antes de seguir debemos de instalar algunos paquetes que nos haran falta para poder poner nuestra aplicacion en produccion, para ello debemos de tener instalado **python3-venv** para poder crear el entorno virtual de python, debemos de tener instalado **uwsgi** que es el servidor de aplicaciones que usaremos junto a apache2 para servir nuestra pagina y tambien debemos de instalar el conector con la base de datos que es **python3-mysqldb**. Si no fueran suficientes, mas adelante indicare los paquetes que debemos de tener instalados ya que iremos paso a paso.

Primero necesitamos nuestro repositorio, por lo que vamos a instalar el paquete llamado **git** y clonamos el repositorio via https:
```shell
#### Instalamos el paquete git ####
[centos@quijote ~]$ sudo dnf install git

#### Clonamos el repositorio en /var/www/ ####
[centos@quijote www]$ sudo git clone https://github.com/FranJaviMN/python-cms-mezzanine.git
```

Una vez ya tengamos clonado el repositorio podemos seguir con nuestro despliegue.

Hecho esto ahora vamos a crear nuestro entorno virtual con python3-venv por lo que lo vamos a llamar mezzanine:
```shell
#### Creamos el entorno virtual ####
[centos@quijote ~]$ python3 -m venv mezzanine

#### Entramos en el entorno virtual y copiamos los paquetes del fichero requirement.txt ####
(mezzanine) [centos@quijote ~]$ pip install -r /var/www/python-cms-mezzanine/mezzanine_openstack/requirements.txt
```

Una vez hecho esto tendremos los mismos paquetes instalados en el entorno de produccion como en el entorno de desarrollo. Ahora necesitamos varios paquetes, priemro el paquete que permite a apache2 ejecutar codigo pyhton, segundo debemos de instalar el modulo para que python trabaje con mariadb y luego, en el entorno virtual debemos de instalar el conector:
```shell
#### Instalamos el modulo de wsgi ####
[centos@quijote ~]$ sudo dnf install python3-mod_wsgi

#### Instalamos en el entorno virtual el conector ####
(mezzanine) [centos@quijote ~]$ pip install mysql-connector-python

#### Instalamos el paquete de uwsgi con pip ####

Antes debemos de tener instalados en nuestra maquina los siguientes paquetes:

[centos@quijote ~]$ sudo dnf install gcc python3-devel

# Ahora si instalamos con pip

(mezzanine) [centos@quijote ~]$ pip install uwsgi
```

Ahora si nosotros intentamos ejecutar **uwsgi** veremos que se ejecuta sin problemas, en mi caso vamos a ejecutar el cms llamado mezzanine:
```shell
#### Desde el entorno virtual ejecutamos uwsgi ####
(mezzanine) [centos@quijote ~]$ mezzanine/bin/uwsgi --http 8080 --plugin python35 --chdir /var/www/python-cms-mezzanine/mezzanine_openstack/ --wsgi-file /var/www/python-cms-mezzanine/mezzanine_openstack/mezzanine_openstack/wsgi.py --process 4 --threads 2 --master

#### Nos debe salir de la siguiente manera ####
!!! UNABLE to load uWSGI plugin: ./python35_plugin.so: cannot open shared object file: No such file or directory !!!
*** Starting uWSGI 2.0.19.1 (64bit) on [Wed Jan 20 09:27:22 2021] ***
compiled with version: 8.3.1 20191121 (Red Hat 8.3.1-5) on 19 January 2021 11:18:26
os: Linux-4.18.0-240.1.1.el8_3.x86_64 #1 SMP Thu Nov 19 17:20:08 UTC 2020
nodename: quijote
machine: x86_64
clock source: unix
detected number of CPU cores: 1
current working directory: /home/centos
detected binary path: /home/centos/mezzanine/bin/uwsgi
!!! no internal routing support, rebuild with pcre support !!!
chdir() to /var/www/python-cms-mezzanine/mezzanine_openstack/
your processes number limit is 1647
your memory page size is 4096 bytes
detected max file descriptor number: 1024
lock engine: pthread robust mutexes
thunder lock: disabled (you can enable it with --thunder-lock)
uWSGI http bound on 8080 fd 4
uwsgi socket 0 bound to TCP address 127.0.0.1:39089 (port auto-assigned) fd 3
Python version: 3.6.8 (default, Aug 24 2020, 17:57:11)  [GCC 8.3.1 20191121 (Red Hat 8.3.1-5)]
Python main interpreter initialized at 0xbed860
python threads support enabled
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 416720 bytes (406 KB) for 8 cores
*** Operational MODE: preforking+threaded ***
WSGI app 0 (mountpoint='') ready in 2 seconds on interpreter 0xbed860 pid: 19143 (default app)
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI master process (pid: 19143)
spawned uWSGI worker 1 (pid: 19145, cores: 2)
spawned uWSGI worker 2 (pid: 19146, cores: 2)
spawned uWSGI worker 3 (pid: 19147, cores: 2)
spawned uWSGI worker 4 (pid: 19148, cores: 2)
spawned uWSGI http 1 (pid: 19149)

#### Para cancelar solo usamos Ctrl+c ####
^CSIGINT/SIGQUIT received...killing workers...
gateway "uWSGI http 1" has been buried (pid: 19149)
worker 1 buried after 1 seconds
worker 2 buried after 1 seconds
worker 3 buried after 1 seconds
worker 4 buried after 1 seconds
goodbye to uWSGI.
```

## Conexion con la base de datos

Para ello debemos de tener en cuenta que la base de datos se encuentra en otra maquina y en otra red, mas concretamente en la maquina sancho por lo que debemos de asegurarnos que nos podemos conectar a ella sin problemas como hemos visto al principio del post.

Ahora debemos de crear una nueva base de datos en la que vamos copiar las tablas que necesitamos para poder seguir con el ejercicio por lo que nos dirigimos a nuestra maquina **Sancho** y ahi vamos a crear la base de datos llamada **mezzanine** y le vamos a dar al usuario **mezzanine** los privilegios necesarios para poder interactuar con esa tabla, por lo que hacemos lo siguiente:
```shell
#### Creamos nuestra nueva tabla ####
MariaDB [(none)]> create database mezzanine;

#### Le damos los privilegios necesarios al usuario mezzanine ####
MariaDB [(none)]> GRANT ALL privileges ON `mezzanine`.* TO 'mezzanine'@'10.0.2.3';
```

De esta forma, desde nuestra maquina vamos a conectarnos a la base de datos y vamos a usar la base de datos que hemos creado para ver si funciona correctamente:
```shell
#### Nos conectamos remotamente ####
[centos@quijote ~]$ mysql -u mezzanine -p -h bd.franjavier.gonzalonazareno.org
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 53
Server version: 10.3.25-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

#### Usamos la nueva base de datos que hemos creado ####
MariaDB [(none)]> use mezzanine
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mezzanine]> 
```

Como vemos funciona correctamente por lo que podemos seguir con la copia de la base de datos a mysql, para ello nos dirigimos al fichero de **mezzanine** que se llama **settings.py** y ahi buscamos el apartado llamado **DATABASES** y debemos de añadirle las credenciales de la base de datos asi como su ip y el puerto que usa:
```python
DATABASES = {
    "default": {
        # Add "postgresql_psycopg2", "mysql", "sqlite3" or "oracle".
        "ENGINE": "django.db.backends.mysql",
        # DB name or path to database file if using sqlite3.
        "NAME": "mezzanine",
        # Not used with sqlite3.
        "USER": "mezzanine",
        # Not used with sqlite3.
        PASSWORD": "contraseña-del-usuario",
        # Set to empty string for localhost. Not used with sqlite3.
        "HOST": "10.0.1.8",
        # Set to empty string for default. Not used with sqlite3.
        "PORT": "",
    }
}
```

Asi ya tendremos la base de datos asignada por lo que vamos a proceder a copiar los datos a la base de datos, para ello debemos de usar el entorno virtual de python y ejecutar el siguiente comando:
```shell
#### Usamos el entorno virtual ####
[centos@quijote ~]$ source mezzanine/bin/activate

#### Creamos la migracion a la base de datos remota ####
(mezzanine) [centos@quijote mezzanine_openstack]$ python manage.py migrate

#### Cargamos los datos que hemos recolectado con copia_db.json ####
(mezzanine) [centos@quijote mezzanine_openstack]$ python manage.py loaddata copia_db.json
```

De esta forma ya tenemos toda la informacion en una base de datos mariadb que se encuentra en otra maquina.

## Despliegue del CMS en apache2

Para ello debemos de tener ya instalado apache2 en nuestra maquina, recalco que la maquina es **centos8** por lo que la instalacion de apache2 cambia un poco, pero puedes ver como lo instale en el siguiente [documento](https://franjavimn.onrender.com/servicios/trabajo-openstack/).

Cuadno lo tengamos instalado lo que vamos a hacer es formar un nuevo virtualhost el cual vamos a llamar **mezzanine.conf** el cual va a tener solo la siguiente información:
```shell
<VirtualHost *:80>
    ServerName python.franjavier.gonzalonazareno.org
    DocumentRoot /var/www/python-cms-mezzanine/mezzanine_openstack
    ErrorLog /var/www/python-cms-mezzanine/log/error.log
    CustomLog /var/www/python-cms-mezzanine/log/requests.log combined
	
    <Proxy "unix:/run/php-fpm/www.sock|fcgi://php-fpm">
    ProxySet disablereuse=off
    </Proxy>

    <FilesMatch \.php$>
           SetHandler proxy:fcgi://php-fpm
    </FilesMatch>

    Redirect permanent / https://python.franjavier.gonzalonazareno.org
```

Donde vamos a tener nuestro fichero con la redireccion hacia **https**, por lo que tendremos un fichero llamado **mezzanine-https.conf** con la siguiente configuración:
```shell
<VirtualHost *:443>
    ServerName python.franjavier.gonzalonazareno.org
    DocumentRoot /var/www/python-cms-mezzanine/mezzanine_openstack
    ErrorLog /var/www/python-cms-mezzanine/log/error.log
    CustomLog /var/www/python-cms-mezzanine/log/requests.log combined
	
    <Proxy "unix:/run/php-fpm/www.sock|fcgi://php-fpm">
    ProxySet disablereuse=off
    </Proxy>

    <FilesMatch \.php$>
           SetHandler proxy:fcgi://php-fpm
    </FilesMatch>

    Alias /static "/var/www/python-cms-mezzanine/mezzanine_openstack/static"
    
    <Directory /var/www/python-cms-mezzanine/mezzanine_openstack/static>
    Require all granted
    Options FollowSymlinks
    </Directory> 
    ProxyPass /static !
    ProxyPass / http://127.0.0.1:8080/

    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/wildcard-franjavier.crt
    SSLCertificateKeyFile /etc/pki/tls/private/wildcard.key

</VirtualHost>
```

Donde vemos que vamos a usar como proxy nuestro apache como proxy inverso, por lo que vamos a usar uwsgi y vamos a usar el puerto **http :8080**, por lo que le indicamos como va a hacer el proxy y en que puerto. La opcion que estamos añadiendo de **ProxyPass /static !** nos sirve para que, la carpeta de static no pase por el proxy para asi poder servir adecuadamente los ficheros estaticos. De tal forma que si reinciamos el servicio de apache debe de ir bien.

Ahora necesitamos iniciar el proxy con uwsgi, para ell podemos, o bien usar un fichero **.ini** o un comando, para este vamos a usar un fichero .ini:
```shell
#### Fichero uwsgi.ini ####
[uwsgi]
http = :8080
chdir = /var/www/python-cms-mezzanine/mezzanine_openstack 
wsgi-file = /var/www/python-cms-mezzanine/mezzanine_openstack/mezzanine_openstack/wsgi.py
master
```

Lo inciamos y veremos lo siguiente:
```shell
(mezzanine) [centos@quijote ~]$ uwsgi uwsgi.ini 
[uWSGI] getting INI configuration from uwsgi.ini
*** Starting uWSGI 2.0.19.1 (64bit) on [Mon Jan 25 10:37:57 2021] ***
compiled with version: 8.3.1 20191121 (Red Hat 8.3.1-5) on 19 January 2021 11:18:26
os: Linux-4.18.0-240.1.1.el8_3.x86_64 #1 SMP Thu Nov 19 17:20:08 UTC 2020
nodename: quijote
machine: x86_64
clock source: unix
detected number of CPU cores: 1
current working directory: /home/centos
detected binary path: /home/centos/mezzanine/bin/uwsgi
!!! no internal routing support, rebuild with pcre support !!!
chdir() to /var/www/python-cms-mezzanine/mezzanine_openstack
your processes number limit is 1647
your memory page size is 4096 bytes
detected max file descriptor number: 1024
lock engine: pthread robust mutexes
thunder lock: disabled (you can enable it with --thunder-lock)
uWSGI http bound on :8080 fd 4
uwsgi socket 0 bound to TCP address 127.0.0.1:35777 (port auto-assigned) fd 3
Python version: 3.6.8 (default, Aug 24 2020, 17:57:11)  [GCC 8.3.1 20191121 (Red Hat 8.3.1-5)]
*** Python threads support is disabled. You can enable it with --enable-threads ***
Python main interpreter initialized at 0x139f470
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 145808 bytes (142 KB) for 1 cores
*** Operational MODE: single process ***
WSGI app 0 (mountpoint='') ready in 1 seconds on interpreter 0x139f470 pid: 44150 (default app)
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI master process (pid: 44150)
spawned uWSGI worker 1 (pid: 44152, cores: 1)
spawned uWSGI http 1 (pid: 44153)
```

## Entrando en nuestro sitio web

Para ello debemos de irnos a nuestro servidor DNS para poder añadirle nuestro nuevo virtualhost, por lo que en la vista externa de nuestro servidor DNS debemos de poner una opcion **CNAME** de python hacia nuestra maquina **Dulcinea**, una vez hecho si nos dirigimos a nuestro navegador y entramos en **python.franjavier.gonzalonazareno.org** y debe de salirnos la siguiente pagina:

![pagina mezzanine](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/mezzanine/mezzanine-pagina.png)

Y esta en nuestra pagina de admin, que vemos que se sirven bien los ficheros:

![pagina admin mezzanine](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/mezzanine/mezzanine-pagina-admin.png)