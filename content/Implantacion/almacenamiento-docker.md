---
title: "Almacenamiento Docker"
date: 2021-02-09T13:22:05+01:00
draft: true
---

# Almacenamiento

**Los contenedores son efímeros**, es decir, los ficheros, datos y configuraciones que creamos en los contenedores sobreviven a las paradas de los mismos pero, sin embargo, son destruidos si el contenedor es destruido.

Por lo que todo contenido que tengamos dentro del contenedor puede ser destruido y todo lo que haya en su interior, podemos verlo facilmente con el siguiente ejemplo de un servidor web:
```shell
#### Creamos el contenedor con la imagen de apache2 ####
debian@omega:~$ docker run -d --name my-apache-app -p 8080:80 httpd:2.4

#### Le introducimos un comando para crear un fichero en el documentroot ####
debian@omega:~$ docker exec my-apache-app bash -c 'echo "<h1>Hola Docker!!!</h1>" > htdocs/index.html'

#### Comprobamos si tenemos ese mensaje ####
debian@omega:~$ curl http://localhost:8080
<h1>Hola Docker!!!</h1>

#### Borramos el contenedor y lo volvemos a crear ####
debian@omega:~$ docker rm -f my-apache-app
my-apache-app

debian@omega:~$ sudo docker run -d --name my-apache-app -p 8080:80 httpd:2.4

#### Comprobamos el mensaje ####
debian@omega:~$ curl http://localhost:8080
<html><body><h1>It works!</h1></body></html>
```

Como vemos en el ejemplo anterior podemos ver como los contenedores son efimeros. Pero esto lo podemos solucionar usando volumenes, en este caso podemos hacer mediante dos formas como pueden ser **volumenes docker** o **bind mount**

* Volúmenes Docker: Si elegimos conseguir la persistencia usando volúmenes estamos haciendo que los datos de los contenedores que nosotros decidamos se almacenen en una parte del sistema de ficheros que es gestionada por docker y a la que, debido a sus permisos, sólo docker tendrá acceso. En linux se guardan en /var/lib/docker/volumes. Este tipo de volúmenes se suele usar en los siguiente casos:
   * Para compartir datos entre contenedores. Simplemente tendrán que usar el mismo volumen.
   * Para copias de seguridad ya sea para que sean usadas posteriormente por otros contenedores o para mover esos volúmenes a otros hosts.
   * Cuando quiero almacenar los datos de mi contenedor no localmente si no en un proveedor cloud.

* Bind mounts: Si elegimos conseguir la persistencia de los datos de los contenedores usando bind mount lo que estamos haciendo es “mapear” (montar) una parte de mi sistema de ficheros, de la que yo normalmente tengo el control, con una parte del sistema de ficheros del contenedor. De esta manera conseguimos:
   * Compartir ficheros entre el host y los containers.
   * Que otras aplicaciones que no sean docker tengan acceso a esos ficheros, ya sean código, ficheros etc…


## Gestión de volúmenes Docker 

Algunos comando útiles para trabajar con volúmenes docker:
* docker volumen create: Crea un volumen con el nombre indicado.
* docker volume rm: Elimina el volumen indicado.
* docker volumen prune: Para eliminar los volúmenes que no están siendo usados por ningún contenedor.
* docker volume ls: Nos proporciona una lista de los volúmenes creados y algo de información adicional.
* docker volume inspect: Nos dará una información mucho más detallada de el volumen que hayamos elegido.

## ¿Qué información tenemos que guardar?

Para terminar: ¿Qué debemos guardar de forma persistente en un contenedor?
* Los datos de la aplicación
* Los logs del servicio
* La configuración del servicio: En este caso podemos añadirla a la imagen, pero será necesaria la creación de una nueva imagen si cambiamos la configuració
* Si la guardamos en un volumen hay que tener en cuanta que ese fichero lo tenemos que tener en el entorno de producción (puede ser bueno, porque l
* configuraciones de los distintos entornos puede variar).

## Asociando almacenamiento a los contenedores

Veamos como puedo usar los volúmenes y los bind mounts en los contenedores. Para cualquiera de los dos casos lo haremos mediante el uso de dos flags de la orden *docker run*: 


## Ejercicios

1. Vamos a trabajar con volúmenes docker:
   * Crea un volumen docker que se llame miweb
   * Crea un contenedor desde la imagen php:7.4-apache donde montes en el directorio/var/www/html(que sabemos que es el docuemntroot del servidor que nos ofrece esa imagen) el volumen docker que has creado.
   * Utiliza el comando **docker cp** para copiar un fichero info.php en el directorio /var/www/html.
   * Accede al contenedor desde el navegador para ver la información ofrecida por el fichero info.php.
   * Borra el contenedor
   * Crea un nuevo contenedor y monta el mismo volumen como en el ejercicio anterior.
   * Accede al contenedor desde el navegador para ver la información ofrecida por el fichero info.php. ¿Seguía existiendo ese fichero?

Para ello lo primero que debemos de hacer es crear el volumen, para ello debemos de usar comando **docker volume create**:
```shell
#### Creamos el volumen llamado miweb ####
debian@omega:~$ sudo docker volume create miweb
miweb

#### Creamos el nuevo contenedor con la imagen php:7.4-apache ####
debian@omega:~$ sudo docker run -d --name prueba-volumenes -v miweb:/var/www/html -p 8080:80 php:7.4-apache

#### Le añadimos un fichero info.php con docker cp ####
debian@omega:~$ cat info.php 
<?php phpinfo();?>

debian@omega:~$ sudo docker cp info.php prueba-volumenes:/var/www/html
```

De esta forma ya lo tendremos funcionando y si entramos en el direccion de la maquina indicando el puerto y el fichero *info.php* podremos ver la información que nos ofrece php.

Ahora vamos a destruir el contenedor y lo vamos a volver a crear para comprobar como sigue sirviendo el fichero info.php porque este fichero esta en un volumen, el cual no se borra al destruir un contenedor:
```shell
#### Destruimos el contenedor ####
debian@omega:~$ sudo docker rm -f prueba-volumenes
prueba-volumenes

#### Volvemos a crear el volumen ####
debian@omega:~$ sudo docker run -d --name prueba-volumenes -v miweb:/var/www/html -p 8080:80 php:7.4-apache
```

Si volvemos a entrar en la dirección del contenedor veremos que nos sigue mostrando el fichero de *info.php*


2. Vamos a trabajar con bind mount:
   * Crea un directorio en tu host y dentro crea un fichero index.html.
   * Crea un contenedor desde la imagen php:7.4-apache donde montes en el directorio /var/www/html el directorio que has creado por medio de bind mount.
   * Accede al contenedor desde el navegador para ver la información ofrecida por el fichero index.html.
   * Modifica el contenido del fichero index.html en tu host y comprueba que al refrescar la página ofrecida por el contenedor, el contenido ha cambiado.
   * Borra el contenedor
   * Crea un nuevo contenedor y monta el mismo directorio como en el ejercicio anterior.
   * Accede al contenedor desde el navegador para ver la información ofrecida por el fichero index.html. ¿Se sigue viendo el mismo contenido?

Lo primero es crear el directorio que lo vamos a llamar *vol-html* y le vamos a añadir un nuevo fichero *index.php* con un mensaje de saludo. Una vez hecho debemos de crear un nuevo contenedor en el que vamos a montar el directorio mediando **bind mount** para que nos sirva el contenido de ese directorio, para ell seguimos los siguientes pasos:
```shell
#### Creamos el directorio y el fichero ####
debian@omega:~$ mkdir vol-html
debian@omega:~$ cd vol-html/
debian@omega:~/vol-html$ nano index.html

#### Creamos el contenedor montando el dirctorio vol-html ####
debian@omega:~$ sudo docker run -d --name prueba-volumenes -v /home/debian/vol-html/:/var/www/html -p 8080:80 php:7.4-apache

#### Comprobamos el mensaje de bienvenida ####
debian@omega:~$ curl http://localhost:8080
<h1>Hola Docker!!!</h1>

#### Modificamos el mensaje y vemos si nos lo muestra de nuevo ####
debian@omega:~$ nano vol-html/index.html
debian@omega:~$ curl http://localhost:8080
<h1>Hola Docker!!!, Bienvenido</h1>
```

Ahora lo que vamos a hacer es borrar el contenedor y vamos a volver a crearlo para ver que no hemos perdido el contenido que teniamos:
```shell
#### Borramos el contenedor ####
debian@omega:~$ sudo docker rm -f prueba-volumenes
prueba-volumenes

#### Lo volvemos a crear ####
debian@omega:~$ sudo docker run -d --name prueba-volumenes -v /home/debian/vol-html/:/var/www/html -p 8080:80 php:7.4-apache

#### Y lo comprobamos ####
debian@omega:~$ curl http://localhost:8080
<h1>Hola Docker!!!, Bienvenido</h1>
```

3. Contenedores con almacenamiento persistente
   * Crea un contenedor desde la imagen nextcloud (usando sqlite) configurando el almacenamiento como nos muestra la documentación de la imagen en Docker Hub* (pero utilizando bind mount). Sube algún fichero.
   * Elimina el contenedor.
   * Crea un contenedor nuevo con la misma configuración de volúmenes. Comprueba que la información que teníamos (ficheros, usuaurio, …), sigue existiendo.
   * Comprueba el contenido de directorio que se ha creado en el host.

Para ello vamos a seguir los siguientes pasos:
```shell
#### Creamos el contenedor ####
debian@omega:~$ sudo docker run -d --name prueba-volumenes -v /home/debian/vol-nextcloud/:/var/www/html -p 8080:80 nextcloud
```

Una vez hecho, nos vamos al navegador y ponemos la ip de nuestra maquina y el puerto, y completamos la instalacion de nextcloud. Una vez hecho eso subimos un fichero a nextclodu para ver si, al borrar el contenedor persisten los datos, por lo que subimos un fichero y lo volvemos a crear y veremos que tenemos el mismo fichero que habiamos subido ademas de que ya no tendremos que completar la instalacion de nextcloud ya que la información ya esta almacenada en la base de datos de tipo **sqlite**.