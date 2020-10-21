---
title: "Ej4 Apache"
date: 2020-10-21T18:10:55+02:00
draft: true
---

# Ejercicio 4: Mapear URL a ubicaciones de un sistema de ficheros

Crea un nuevo host virtual que es accedido con el nombre www.mapeo.com, cuyo DocumentRoot sea /srv/mapeo

1. Cuando se entre a la dirección www.mapeo.com se redireccionará automáticamente a www.mapeo.com/principal, donde se mostrará el mensaje de bienvenida.

2. En el directorio principal no se permite ver la lista de los ficheros, no se permite que se siga los enlaces simbólicos y no se permite negociación de contenido. Muestra al profesor el funcionamiento. ¿Qué configuración tienes que poner?

3. Si accedes a la página www.mapeo.com/principal/documentos se visualizarán los documentos que hay en /home/usuario/doc. Por lo tanto se permitirá el listado de fichero y el seguimiento de enlaces simbólicos siempre que el propietario del enlace y del fichero al que apunta sean el mismo usuario. Explica bien y pon una prueba de funcionamiento donde se vea bien el seguimiento de los enlaces simbólicos.

4. En todo el host virtual se debe redefinir los mensajes de error de objeto no encontrado y no permitido. Para el ello se crearan dos ficheros html dentro del directorio error. Entrega las modificaciones necesarias en la configuración y una comprobación del buen funcionamiento.

Antes de empezar vamos a tener en claro las siguientes opciones de apache2:

* All: Todas las opciones excepto MultiViews.

* FollowSymLinks: Todas las opciones excepto MultiViews.

* Indexes: Cuando accedemos al directorio y no se encuentra un fichero por defecto, se muestra la lista de ficheros.

* MultiViews: Permite la negociación de contenido.

* SymLinksOwnerMatch: Se pueden seguir enlaces simbólicos, sólo cuando el fichero destino es del mismo propietario que el enlace simbólico

* ExecCGI: Permite ejecutar script CGI.

Podemos activar o desactivar una opción en referencia con la configuración de un directorio padre mediante el signo + o -.

Usaremos el siguiente fichero de Vagrantfile:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

$install_apache = <<-SCRIPT
sudo apt update 
sudo apt install apache2 -y
SCRIPT



Vagrant.configure("2") do |config|
  config.vm.box = "debian/buster64"
  config.vm.hostname = "ej4-apache"
  config.vm.network :private_network, ip: "192.168.100.100"
  config.vm.provision "shell", inline: $install_apache


end
```

Lo primero que debemos de hacer es crear nuestro virtualhost, para ello lo que debemos de hacer es crear nuestro virtualhost. Nos dirigimos al directorio */etc/apache2/sites-available* y ahi copiamos el fichero de configuracion de default con un nuevo nombre, en este caso mapeo:

```shell
sudo cp 000-default.conf ./mapeo.conf
```

De esta forma vamos a tener el fichero de configuracion para mapeo, ahora debemos de habilitar el directorio */srv* en el fichero de *apache2.conf* y buscar una linea como la siguiente:

```shell
<Directory /srv/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

Y asi tendremos para elegir el documentroot de */srv/*, asi en nuestro fichero de configuracion de mapeo debemos de tenerlo de la siguiente forma:

```shell
<VirtualHost *:80>
    ServerName www.mapeo.com
    ServerAdmin webmaster@localhost
    DocumentRoot /srv/mapeo
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Ahora que tenemos el fichero configurado vamos a crear una carpeta llamada *mapeo* en el directorio */srv*. Una vez tengamos creado este directorio en el vamos a crear un fichero llamado *index.html* y vamos a crear una linea con un mensaje de bienvenida:

```shell
#### Creamos el index.html ####
sudo nano /srv/mapeo/index.html

#### Escribimos un mensaje de bienvenida ####

<h1> Hola que tal, bienvenido a mapeo </h1>
```

Ahora nos vamos al directorio */srv/* y ahi tenemos que darle los permisos a el usuario de apache2, para ello usamos el siguiente comando:

```shell
sudo chown -R www-data:www-data mapeo/
```

Una vez tengamos hecho los anteriores pasos solo debemos de reiniciar el servicio de apache con el siguiente comando:

```shell
sudo systemctl reload apache2.service
```

Cuando lo hayamos reiniciado solo debemos de añadir en nuestra maquina local el nuevo host con su ip y el nombre de la pagina a la que vamos a entrar:

```shell
192.168.100.100 www.mapeo.com
```

## Tarea 1

Para poder realizar este ejercicios debemos de usar las redirecciones. Las redirecciones son usadas para pedir al cliente que haga otra petición a una URL diferente. Normalmente la usamos cuando el recurso al que queremos acceder ha cambiado de localización.

En este caso nos pide que, al acceder a la pagina ***www.mapeo.com*** nos redirija a la pagina ***www.mapeo.com/principal*** en la cual encontraremos un mensaje de bienvenida. Lo primero que debemos hacer es dirigirnos al directorio */srv/mapeo* y ahi crear una carpeta llamada ***principal*** en la cual vamos a guardar un fichero ***index.html***:

```shell
#### Creamos la carpeta principal ####

sudo mkdir /srv/mapeo/principal

#### Dentro de esta nueva carpeta creamos el fichero Index.html con un mensaje de bienvenida ####

sudo nano /srv/mapeo/principal/index.html

Y dentro escribimos el mensaje:

<h1>Hola esto es principal</h1>

#### Cambiamos el propietario del directorio al usuario de apache ####

sudo chown -R www-data:www-data mapeo/
```

Una vez hayamos realizado las acciones anteriores nos dirigimos al fichero de configuracion ***mapeo.conf*** para añadir nuestra nueva redirección hacia /principal.

Debemos de tener en cuenta que al usar la opcion de ***redirect*** si queremos redireccionar desde **/** hasta **/principal** veremos que entraremos en un bucle infinito y el propio navegador nos indicara que la redireccion no se esta realizando correctamente por lo que debemos usar la opcion ***RedirectMatch*** que nos va a permitir usar las expresiones regulares

```shell
#### Sintaxis de redirecciones ####
RedirectMatch "path_inicial" "path_redireccionado"

#### Resultado del fichero mapeo.conf ####

<VirtualHost *:80>
    ServerName www.mapeo.com
    ServerAdmin webmaster@localhost
    DocumentRoot /srv/mapeo
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    RedirectMatch ^/$ /principal

</VirtualHost>

#### Reiniciamos el servicio de apache2 ####

sudo systemctl reload apache2.service
```

Ahoara si entramos en nuestro navegador y entramos en la pagina de www.mapeo.com veremos que nos redirecciona a una pagina llamada /principal en la cual vamos a ver el mensaje que habiamos puesto en nuestro Index.html que hemos creado anteriormente, señal de que la redireccion ha sido un exito.

![Pagina redirect principal](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/captura-principal.png)

## Tarea 2

En esta tarea debemos de hacer que, en nuestro directorio ***principal*** no se puedan listar los los ficheros, no se pueden seguir los enlaces simbolicos y tampoco se permite la negociacion de contenido. Para ello debemos de ir a nuestro fichero de configuracion de *mapeo.conf* y alli debemos de añadir la siguiente linea:

```shell
#### Lineas a añadir en el fichero mapeo.conf ####
...
<directory "/srv/mapeo/principal">
    Options -Indexes -FollowSymLinks -Multiviews
</directory>
...

#### Reiniciamos el servicio ####

sudo systemctl reload apache2.service
```

Donde vemos lo siguiente:
    -Indexes: Indicamos que no queremos que se muestre el listado de fichero que contiene el directorio.
    -FollowSymLinks: Indicamos que no se puedan seguir los enlaces simbolicos.
    -Multiviews: Indicamos que no queremos que haya negociacion de contenido.

Si ahora nos vamos a nuestra pagina ***mapeo.com/principal*** veremos que solo nos va a mostrar el mensaje de bienvenida que le habiamos puesto en principal.

![Pagina sin opciones principal](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/captura-principal.png)


## Tarea 3

AHoara debemos de hacer que al entrar en *www.mapeo.com/principal/documentos* podamos ver todos los fichero que tenemos en el directorio /home/vagrant/doc. Para poder hacer este ejercicio necesitamos crear un alias que nos permita ver todo el contenido de la carpeta doc, para ello nos vamos a dirigir al fichero de configuracion de *mapeo.conf* y ahi tenemos que añadir una linea como la siguiente:

```shell
alias "/principal/documentos" "/home/vagrant/doc/"
```

De esta forma vamos a crear una nueva carpeta en nuestro usuario llamada *doc* y en ella vamos a crear algunos ficheros de prueba:

```shell
#### Creamos una carpeta llamada doc ####

mkdir /home/vagrant/doc

#### Creamos unos ficheros de prueba ####

touch /home/vagrant/doc/{prueba1.txt,prueba2.txt}
```

Despues de hacer todos los anteriores pasos debemos de añadir el directorio con los permisos que nos indica en el enunciado, para ello nos dirigimos al fichero de *mapeo.conf* y ahi vamos a añadir la siguiente linea:

```shell
alias "/principal/documentos" "/home/vagrant/doc/"
<directory "home/vagrant/doc">
    Options Indexes SymLinksIfOwnerMatch
    AllowOverride None
    Require all granted
</directory>

Donde vemos que:
    Indexes: Cuando accedemos al directorio y no se encuentra un fichero por defecto se muestra la lista de ficheros
    SymLinksIfOwnerMatch: Se pueden seguir enlaces simbólicos, sólo cuando el fichero destino es del mismo propietario que el enlace simbólico.
    AllowOverride: controla qué directivas se pueden situar el los ficheros .htaccess.
    Require all granted: El acceso es permitido incondicionalmente

```

Ahora solo nos queda reiniciar el servicio de apache y entrar en la url www.mapeo.com/principal/documentos:

![Prueba documentos](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/principal-documentos.png)

## Tarea 4

Ahora debemos de crear mensajes de error personalizados para los errores ***404 que es objeto no encontrado*** y para el error ***403 que es el error de no permitido***.

Para ello nos vamos a crear dos ficheros, uno por cada error y lo vamos a guardar en el directorio que vamos a crear en */srv/mapeo/error*:

```shell
#### Primero creamos el directorio error ####

sudo mkdir /srv/mapeo/error

#### Creamos los dos fichero de error ####

touch /srv/mapeo/error{404.html,403.html}

#### En cada uno de ellos ponemos el mensaje de error que queramos ####
fichero 404: 
    <h1> No se encuentra el elemento </h1>
fichero 403:
    <h1> No se le permite el acceso </h1>

#### Cambiamos el propietario de los nuevos ficheros ####

sudo chown -R www-data:www-data mapeo/
```

Cuando ya hayamos creado los fichero solo debemos de inidicar en el fichero de *mapeo.conf* que cuando haya un error con el codigo 404 o 403 muestre los ficheros de error que hemos creado, para debemos de introducir las siguientes lineas en el fichero *mapeo.conf*:

```shell
#### Para el error 404 ####

ErrorDocument 404 /error/404.html

#### Para el error 403 ####

ErrorDocument 403 /error/403.html
```

De tal forma, para el error 404 de no encontrar el recurso nos mostrara lo siguiente:

![prueba de error 404](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/error-404.png)

Y para el error 403 probamos quitando los priviliegios del diretorio documentos y al entrar nos mostrara los siguiente:

![prueba error 403](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/error-403.png)