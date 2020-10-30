---
title: "Practica Cms"
date: 2020-10-30T18:03:23+01:00
draft: true
---

# Instalación local de un CMS PHP

Esta tarea consiste en instalar un CMS de tecnología PHP en un servidor local. Los pasos que tendrás que dar los siguientes:

## Tarea 1: Instalación de un servidor LAMP

* Crear una instancia de vagrant basado en un box debian o ubuntu

* Instala en esa máquina virtual toda la pila LAMP

## Tarea 2: Instalación de drupal en mi servidor local

* Configura el servidor web con virtual hosting para que el CMS sea accesible desde la dirección: www.nombrealumno-drupal.org.

* Crea un usuario en la base de datos para trabajar con la base de datos donde se van a guardar los datos del CMS.

* Descarga la versión que te parezca más oportuna de Drupal y realiza la instalación.

* Realiza una configuración mínima de la aplicación (Cambia la plantilla, crea algún contenido, …)

* Instala un módulo para añadir alguna funcionalidad a drupal.

## Tarea 3: Configuración multinodo

* Realiza un copia de seguridad de la base de datos

* Crea otra máquina con vagrant, conectada con una red interna a la anterior y configura un servidor de base de datos.

* Crea un usuario en la base de datos para trabajar con la nueva base de datos.

* Restaura la copia de seguridad en el nuevo servidor de base datos.

* Desinstala el servidor de base de datos en el servidor principal.

* Realiza los cambios de configuración necesario en drupal para que la página funcione.

## Tarea 4: Instalación de otro CMS PHP

* Elige otro CMS realizado en PHP y realiza la instalación en tu infraestructura.

* Configura otro virtualhost y elige otro nombre en el mismo dominio.

## Tarea 5: Necesidad de otros servicios

* La mayoría de los CMS tienen la posibilidad de mandar correos electrónicos (por ejemplo para notificar una nueva versión, notificar un comentario,…)

* Instala un servidor de correo electrónico en tu servidor. debes configurar un servidor relay de correo, para ello en el fichero /etc/postfix/main.cf, debes poner la siguiente línea:

    ```shell
    relayhost = babuino-smtp.gonzalonazareno.org
    ```


### Tarea 1

En esta tarea vamos a realizar el entorno LAMP para poder realizar la practica, para ello vamos a usar el siguiente fichero de Vagrantfile:

```ruby
# vi: set ft=ruby :


$install_LAMP = <<-SCRIPT
#actualizamos la lista de paquetes de la maquina
sudo apt update

#instalamos mariadb
sudo apt install -y mariadb-server mariadb-client

#instalamos apache2
sudo apt install -y apache2

#instalamos php y una de sus librerias
sudo apt install -y php libapache2-mod-php
SCRIPT




Vagrant.configure("2") do |config|
  config.vm.box = "debian/buster64"
  config.vm.hostname = "maquina"
  config.vm.network :private_network, ip: "192.168.100.100"

  config.vm.provision "shell", inline: $install_LAMP
end
```

Donde vemos que tenemos el script creado que nos instala todo lo necesario para poder empezar con nuestro entorno LAMP.

Una vez dentro de nuestra maquina con el entorno LAMP instalado, vamos a pasar a la configuracion de los elementos LAMP.

#### MariaDB

Lo primero es la creacion de una nueva base de datos en la que vamos a alojar nuestros cms, para ello vamos a usar la siguiente pila de comandos cmabiando los elementos "nombredb" por el nombre de la base de datos y "nombreuser" por el nombre del usuario que vamos a crear:

```shell
#### Entramos como root en mariadb ####
sudo mysql -u root -p

#### Creamos la base de datos ####
CREATE DATABASE nombredb;

#### Creamos el usuario ####
CREATE USER 'nombreuser'@'localhost' IDENTIFIED BY 'contraseñauser';

#### Le damos todos los privilegios al nuevo usuario sobre la nueva base de datos ####
GRANT ALL PRIVILEGES ON nombredb.* to 'nombreuser'@'localhost';

#### Salimos de mariadb ####
FLUSH PRIVILEGES;
quit
```

#### Apache

Aqui vamos a crear nuestro virtualhost en apache, para ello vamos a seguir los siguientes pasos:

1. Nos dirigimos al directorio /etc/apache2/sites-available/ y ahi vamos a crear nuestro nuesvo virtual host al que vamos a llamar drupal.conf:

2. Entramos en el directorio /etc/apache2/sites-available/

```shell
cd /etc/apache2/sites-available/
```

3. Creamos el nuevo fichero de configuracion

```shell
sudo cp 000-default.conf ./drupal.conf
```

4. Dentro del fichero drupal.conf tendremos lo siguiente 

```shell
<VirtualHost *:80>
        ServerName www.francisco-drupal.org
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/drupal
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
5. Creamos la carpeta drupal en /var/www/ 

```shell
sudo mkdir /var/www/drupal
```

7. Dentro de /var/www/drupal vamos a crear un fichero index.html con un mensaje de bienvenida 
```shell
sudo nano /var/www/drupal/index.html
```

8. Activamos el nuevo virtualhost 
```shell
  sudo a2ensite drupal.conf
```
9. Reiniciamos el servicio de apache 
```shell
sudo systemctl reload apache2.service
```

Asi, a la hora de entrar en la direccion www.francisco-drupal.org, primero modificamos el fichero /etc/hosts y añadimos la ip de nuestro servidor apache y la direccion de nuestro virtualhost y al entrar en la pagina nos saldra lo siguiente:

![Prueba de drupal](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/CMS-LAMP/drupal-prueba.png)

#### PHP

Vamos a comprobar que el modulo de php para apache esta bien instalado y funciona, para ello vamos a realizar lo siguiente:

1. Iniciamos el modulo de php:
```shell
sudo a2enmod php7.3
```

2. Creamos un pequeño script en /var/www/drupal para ver si esta funcionando.
```shell
echo "<?php phpinfo(); ?>" | sudo tee /var/www/drupal/phpinfo.php
```

3. Comprobamos que en la pagina http://www.francisco-drupal.org/phpinfo.php sale este mensaje:

![imagen de php](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/CMS-LAMP/php-drupal.png)


### Tarea 2

Los dos primeros apartados de esta tarea ya las hemos realizado en la anterior tarea, ahora nos vamos a ir a la instalacion de drupal, para ello vamos a hacer es descargar drupal desde el siguiente [enlace](https://www.drupal.org/download).

1. Una vez instalado lo que debemos de hacer es descargar el fichero en nuestro servidor, dentro de la carpeta de nuestro virtualhost para drupal:

```shell
sudo wget https://www.drupal.org/download-latest/zip
```

2. Ahora vamos a pasar a descomprimir el fichero y a entrar en el instalador de nuestro drupal, para ello usamos lo siguiente:
```shell
#### Descomprimimos el archivo zip
sudo unzip zip

#### Damos los permisos de www-data a la carpeta www/* ####
sudo chown -R www-data:www-data www/

#### Entramos en la siguiente url ####
http://www.francisco-drupal.org/drupal-9.0.7
```

Una vez hayamos entrado en el enlace anterior nos debe salir una pagina como la siguiente, mostrandonos el instalador de drupal:

![Imagen instalacion](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/CMS-LAMP/drupal-instalacion.png)


Una vez lleguemos a la zona de verificar requisitos, es posible que nos falten algunos requisitos, en mi caso me hacen falta los siguientes:

![Requisitos no cumplidos](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/CMS-LAMP/drupal-requisitos.png)

Estos requisitos debemos de solucionarlos para poder avanzar.

* Solucion de los requisitos de *dom*, *simplexml* y *xml*: Para la solucion de estos requisitos nos dirigimos a nuestro servidor y ahi instalamos la extension de php sobre xml:
```shell
sudo apt install php-xml
```
Una vez instalados los modulos de xml para php, reiniciamos el servicio de apache2 y recargamos la pagina en la que se habran eliminado las advertencias anterirores.

* Solucion para el requisito *gd*: Para ello debemos de instalar la extension de php llamado *php7.3-gd*:
```shell
sudo apt install php-gd
```
Una vez instalado, reiniciamos el servicio y recargamos la pagina, y asi ya habriamos solucionado el problema de gd.

* Soulucion para el requisito sobre la funcionalidad de la base de datos: Para ello debemos de instalar la siguiente extension de php llamada *php-mysql*:
```shell
sudo apt install php-mysql
```
Una vez instalado, reiniciamos el servicio de apache y recargamos la pagina.

Llegados a este punto solo nos deberian de aparecer las advertencias siguientes:

![advertencias grupal](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/CMS-LAMP/advertencias-drupal.png)

* Solucion de bibioteca unicode: Para ello solo debemos de instalar la extension de php llamada *php7.3-mbstring*:
```shell
sudo apt install php-mbstring
```
Una vez hecho esto, reiniciamos el servicio de apache y recargamos la pagina.

* Solucion para URL Limpias: Para ello debemos de iniciar el modulo de **rewrite** y añadir unos parametros en el fichero de **drupal.conf**:
```shell
#### Iniciamos el modulo de rewrite ####
sudo a2enmod rewrite

#### Añadimos las siguientes lineas en el fichero drupal.conf ####
<directory /var/www/drupal>
  AllowOverride All
</directory>
Le estamos diciendo a nuestro fichero que si acepte el fichero .htaccess del cms drupal
```
Una vez realizado estos pasos, reiniciamos nuestros servicio de apache2 y recargamos la pagina y veremos que ya no aparece ninguna advertencia ni ningun error.


3. El siguiente paso que debemos de realizar es rellenar los campos de la base de datos que nos pide en la pagina de configurar la base de datos. Nos pediran el nombre de la base de datos, el nombre del usuario y la contraseña de dicho usuario. Solo debemos de rellenar los campos adecuadamente y tener en cuenta que el directorio donde tengamos nuestro virtualhost debe ser propiedad del usuario de apache2 llamado **www-data**.

4. Ahora solo debemos esperar que termine su instalacion, nos pediran que rellenemos algunos campos como el nombre del sitio y nuestro nombre, correo, etc. Una vez terminado nos aparecera la siguiente pagina cuando entremos en la direccion ***http://www.francisco-drupal.org/drupal-9.0.7***:

![Pagina principal drupal](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/CMS-LAMP/pagina-drupal.png)

#### Instalar nuevo tema

Ahora que ya tenemos nuestro sitio funcionando vamos a hacer una configuración basica del sitio como por ejemplo el cambio del tema de drupal y vamos a añadir un contenido de prueba para ver que todo funciona correctamente. Para añadir contenido solo debemos de seguir los pasos que nos indica al ir a la pestaña de contenido, al igual que si cambiamos el tema.

* Contenido añadido: Para añadir contenido nos dirigimos al menú de administrar/contenido y una vez ahi solo debemos de ir a *añadir contenido* y seguimos los pasos que nos muestra drupal.

* Cambiando el tema: En mi caso el tema que voy a elegir es [Vani](https://www.drupal.org/project/vani) y para la instalacion del tema solo nos vamos a tener que ir al menu de *apariencia* y ahi seleccionamos el boton de **instalar un nuevo tema**. Una vez dentro copiamos la url de descarga de nuestro tema comprimido en **tar.gz** y lo pegamos donde dice *instalar desde url* y ya solo nos queda disfrutar de nuestro nuevo tema.

#### Instalar un modulo nuevo

Para ello vamos a instalar un modulo llamado [Add to any](https://www.drupal.org/project/addtoany), para ello nos dirigimos a la pestaña de ampliar y a la zona donde dice **Instalar nuevo modulo**.
Le damos y cuando estemos en la nueva pagina tenemos que copiar la url de descargar y pegarlo en la zona donde dice **Instalar desde una URL**.

Lo pegamos y una vez pegado seguimos los pasos que nos indica y elegimos el nuevo modulo en la pestaña de ampliar y lo instalamos. Ahoara solo vamos a ver el resultado del modulo.

Este modulo lo que hace es que, cuando creamos un nuevo articulo nos agrega los botones de las redes sociales que podemos ir configurando para que sea con nuestras redes sociales.

![Prueba de modulo](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/CMS-LAMP/prueba-modulo.png)

### Tarea 3

1. Realizamos la copia de seguridad: Para poder hacer una copia de seguridad en MySQL usaremos una herramienta llamada **mysqldump**. Este comando se incluye dentro de las utilidades del propio servidor MySQL, por lo que ya se instaló cuando instalamaos MySQL. Para poder realizar una copia de seguridad basica de la base de datos que estamos usando en nuestro cms usamos el siguiente comando:
```shell
#### Copia de la base de datos basica ####

mysqldump --user=USUARIO -p NOMBRE_DATA_BASE > copia-db.sql
```
Ahora nos vamos a llevar la copia de la base de datos a nuestra maquina local, para ello lo vamos a hacer con el siguiente comando:
```shell
scp copia-db.sql francisco@192.168.1.111:/home/francisco
```

Entonces si vemos en nuestro directorio ya tendremos nuestra copia de la base de datos de drupal.

2. Ahora vamos a crear una maquina virtual que este en una red interna con la anterior, para ello usamos el siguiente Vagrantfile:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :


$install_LAMP = <<-SCRIPT
#actualizamos la lista de paquetes de la maquina
sudo apt update

#instalamos mariadb
sudo apt install -y mariadb-server mariadb-client

#instalamos apache2
sudo apt install -y apache2

#instalamos php 
sudo apt install -y php libapache2-mod-php
SCRIPT




Vagrant.configure("2") do |config|
  config.vm.define :maquinacms do |maquinacms|
    maquinacms.vm.box = "debian/buster64"
    maquinacms.vm.hostname = "maquinacms"
    maquinacms.vm.network :private_network, ip: "192.168.100.100", virtualbox__intnet: "lan1"
    maquinacms.vm.network :private_network, ip: "192.168.200.100"
    maquinacms.vm.provision "shell", inline: $install_LAMP

  end
  config.vm.define :maquinadb do |maquinadb|
    maquinadb.vm.box = "debian/buster64"
    maquinadb.vm.hostname = "maquinadb"
    maquinadb.vm.network :private_network, ip: "192.168.100.101", virtualbox__intnet: "lan1"
    maquinadb.vm.provision "file", source: "/home/francisco/copia-db.sql", destination: "/home/vagrant/copia-db.sql"
  end
end

```

3. Ahora debemos de instalar en la maquina ***maquinadb*** la base de datos, en este caso vamos a instalar MySQL y crear un usuario:
```shell
#### Instalamos la base de datos MySQL ####
sudo apt install -y mariadb-server mariadb-client php-mysql

#### Entramos como root en mariadb ####
sudo mysql -u root -p

#### Creamos la base de datos ####
CREATE DATABASE nombredb;

#### Creamos el usuario ####
CREATE USER 'nombreuser'@'localhost' IDENTIFIED BY 'contraseñauser';

#### Le damos todos los privilegios al nuevo usuario sobre la nueva base de datos ####
GRANT ALL PRIVILEGES ON nombredb.* to 'nombreuser'@'localhost';

Salimos de mysql y restauramos la copia de la base de datos

#### Recuperamos la copia de la base de datos ####
mysql --user=USUARIO --password=CONTRASEÑA nombredb < copia-db.sql
```

Ahora nos dirigimos a la maquina de maquinacms y en ella desinstalamos mariadb:
```shell
sudo apt purge mariadb-client-10.3 mariadb-server-10.3
```

Una vez desinstalado nos saldra el siguiente error en la pagina de drupal:

![error drupal](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/CMS-LAMP/falta-db.png)

A partir de aqui no me ha funcionado el cambio de base de datos.

### Tarea 4

En mi caso voy a usar como cms extra a instalar voy a usar anchor, que es un liviano sistema de creacion de blogs.

Para ello vamos a realizar los siguientes pasos:

1. Cambiamos el documentroot a ***/var/www***, cambiamos el nombre de drupal.conf y creamos una carpeta en ***/var/www***:
```shell
#### Cambiamos el nombre de drupal.conf y lo volvemos a activar ####
sudo mv drupal.conf ./cms.conf
sudo a2ensite cms.conf

#### Contenido de cms.conf ####
<VirtualHost *:80>
  ServerName www.francisco-drupal.org
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
  <directory /var/www>
    AllowOverride All
  </directory>
</VirtualHost>

#### Cramos la carpeta anchor ####
sudo mkdir /var/www/anchor
```

2. Creamos una nueva base de datos en la cual vamos a guardar nuestro cms:
```shell
####Creamos la nueva base de datos ####
CREATE DATABASE anchor;

#### Le damos todos los privilegios al nuevo usuario sobre la nueva base de datos ####
GRANT ALL PRIVILEGES ON anchor.* to 'nombreuser'@'localhost';

#### Salimos de mariadb ####
FLUSH PRIVILEGES;
quit
```

3. Descargamos nuestro cms desde [aqui](https://github.com/anchorcms/anchor-cms/releases):
```shell
#### Dentro de /var/www/anchor ####
sudo wget https://github.com/anchorcms/anchor-cms/releases/download/0.12.7/anchor-cms-0.12.7-bundled.zip

#### Descomprimimos el fichero ####

sudo unzip anchor-cms-0.12.7-bundled.zip
```

4. Reiniciamos el servicio de apache y entramos en la url www.francisco-drupal.org/anchor/anchor-cms-0.12.7:
```shell
#### Reiniciamos apache2 ####
sudo systemctl reload apache2.service
```
![imagen de instalador de anchor](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/CMS-LAMP/anchor-cms.png)

5. Iniciamos la instalacion de anchor y seguimos sus pasos, en este caso no nos pedira nada en especial, solo el nombre de nuestro sitio, una descripcion y el path que queremos usar. Este ultimo es recomendable cambiarlo.