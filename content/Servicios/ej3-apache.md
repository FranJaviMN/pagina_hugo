---
title: "Ej3 Apache"
date: 2020-10-16T09:11:56+02:00
draft: true
---

# Ejercicio 3: VirtualHosting con Apache

## Introducción al VirtualHosting

El término Hosting Virtual se refiere a hacer funcionar más de un sitio web (tales como www.company1.com y www.company2.com) en una sola máquina. Los sitios web virtuales pueden estar “basados en direcciones IP”, lo que significa que cada sitio web tiene una dirección IP diferente, o “basados en nombres diferentes”, lo que significa que con una sola dirección IP están funcionando sitios web con diferentes nombres (de dominio). Apache fue uno de los primeros servidores web en soportar hosting virtual basado en direcciones IP.

El servidor web Apache2 se instala por defecto con un host virtual. La configuración de este sitio la podemos encontrar en:

```shell
/etc/apache2/sites-available/000-default.conf
```

Y cuyo contenido es el siguiente:

```shell
<VirtualHost *:80>
        #ServerName www.example.com	-> Nombre del sitio web
        ServerAdmin webmaster@localhost -> Correo del admin del servidor apache
        DocumentRoot /var/www/html	-> Indicamos el directorio con los fichero que va a servir apache
        ErrorLog ${APACHE_LOG_DIR}/error.log -> El nombre del fichero donde se refejaran los errores que encuentre
        CustomLog ${APACHE_LOG_DIR}/access.log combined	-> proporciona un registro flexible de las solicitudes de los clientes
</VirtualHost>

Donde:
    ServerName: Nombre del sitio web
    ServerAdmin: Correo del admin del servidor apache
    DocumentRoot: Indicamos el directorio con los fichero que va a servir apache
    ErrorLog: El nombre del fichero donde se refejaran los errores que encuentre
    CustomLog: Proporciona un registro flexible de las solicitudes de los clientes
```

Y por defecto este sitio virtual está habilitado, por lo que podemos comprobar que existe un enlace simbólico a este fichero en el directorio /etc/apache2/sites-enables:

```shell
lrwxrwxrwx 1 root root   35 Oct  3 15:24 000-default.conf -> ../sites-available/000-default.conf
```

Podemos habilitar o deshabilitar nuestros host virtuales utilizando los siguientes comandos:

```shell
a2ensite
a2dissite
```

En el fichero de configuración general ***/etc/apache2/apache2.conf*** nos encontramos las opciones de configuración del directorio padre del indicado en la directiva DocumentRoot (suponemos que todos los host virtuales van a estar guardados en subdirectorios de este directorio).

## Configuración de VirtualHosting

El objetivo de esta práctica es la puesta en marcha de dos sitios web utilizando el mismo servidor web apache. Hay que tener en cuenta lo siguiente:

* Cada sitio web tendrá nombres distintos.

* Cada sitio web compartirán la misma dirección IP y el mismo puerto (80).

Queremos construir en nuestro servidor web apache dos sitios web con las siguientes características:

* El nombre de dominio del primero será *www.iesgn.org*, su directorio base será ***/srv/www/iesgn*** y contendrá una página llamada *index.html*, donde sólo se verá una bienvenida a la página del Instituto Gonzalo Nazareno.

* En el segundo sitio vamos a crear una página donde se pondrán noticias por parte de los departamento, el nombre de este sitio será *www.departamentosgn.org*, y su directorio base será ***/srv/www/departamentos***. En este sitio sólo tendremos una página inicial *index.html*, dando la bienvenida a la página de los departamentos del instituto.

Para poder conseguir crear nuestro virtualhosting vamos a usar el siguiente esquema de pasos:

1. Lo primero que debemos de hacer es crear los ficheros de configuración de los dos sitios web que vamos a crear, para ello vamos a hacerlo de una forma facil copiando el fichero de configuracion que viene por defecto al instalar apache2 que es ***000-default.conf***, este fichero se encuentra en el directorio ***/etc/apache2/sites-available***. Una vez estemos en el directorio anterior copiamos el fichero por defecto dos veces, uno por cada sitio que vamos a crear:

```shell
#### Primero nos dirigimos al directorio /etc/apache2/sites-available ####

cd /etc/apache2/sites-available

#### Copiamos el fichero 000-default.conf dos veces ####

cp 000-default.conf iesgn.conf
cp 000-default.conf departamentos.conf
```

Asi tendremos dos fichero de configuración para nuestros dos sitios web.

2. Ahora debemos de configurar nuestro fichero de configuracion de apache2 para que asi podamos tener como directorio de trabajo la ruta ***/srv/www***, para ello realizamos los siguientes cambios:

```shell
#### Nos dirigimos al fichero /etc/apache2/apache2.conf y descomentamos las siguientes lineas ####

<Directory /srv/www>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

3. Ahora modificamos el fichero de configuracion de los dos sitios que hemos generado anteriormente para que el ServerName sea ***www.iesgn.org*** y ***www.departamentosgn.org*** respectivamente y debemos de poner el DocumentRoot en los directorios ***/srv/www/iesgn*** y ***/srv/www/departamentos***:

```shell
#### Configuracion del fichero iesgn.conf ####

<VirtualHost *:80>
    ServerName www.iesgn.org
    ServerAdmin webmaster@localhost
    DocumentRoot /srv/www/iesgn
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


#### Configuracion del fichero departamentos.conf ####

<VirtualHost *:80>
    ServerName www.departamentosgn.org
    ServerAdmin webmaster@localhost
    DocumentRoot /srv/www/departamentos
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

4. Ahora lo que debemos de hacer es que estos ficheros de configuracion sean funcionales, para hacer esto debemos de crear un enlace simbolico a estos ficheros dentro del directorio ***/etc/apache2/sites-enabled***, para ello usamos el siguiente comando:

```shell
#### Crear el enlace simbolico con a2ensite ####

a2ensite iesgn -> enlace simbolico para el fichero iesgn.conf
a2ensite departamentos -> enlace simbolico para el fichero departamentos.conf

#### Eliminar el enlace simbolico con a2dissite ####

a2dissite iesgn -> eliminar el enlace simbolico para el fichero iesgn.conf
a2dissite departamentos -> eliminar el enlace simbolico para el fichero departamentos.conf
```

5. Ahora que tenemos los enlaces creados solo tenemos que añadir el contenido al DocumentRoot de cada uno de los sitios que hemos creado, para ello debemos de dirigirnos al directorio ***/srv/www*** y ahi debemos de crear dos carpetas llamadas *iesgn* y *departamentos* respectivamente. Una vez creados los directorios, dentro de cada uno de ellos añadimos un fichero index.html con un mensaje de bienvenida a cada pagina:

```shell
#### Index.html del sitio iesgn ####

<h1> Hola, bienvenido a la pagina de iesgn </h1>

#### Index.html del sitio departamentos ####

<h1> Hola, bienvenido a la pagina de departamentos </h1>
```

Una vez creados estos fichero solo debemos de cambiarle el propietario al usuario de opache llamado ***www-data*** de la siguiente forma:

```shell
sudo chown -R www-data:www-data (directorio)
```

Despues de realizar todos estos cambios vamos a reiniciar el servicio de apache2 para que todos estos cambios se realicen:

```shell
#### Reiniciamos el servicio apache ####

sudo systemctl reload apache2.service
```

6. Ahora en nuestra maquina cliente, que en este caso va a ser nuestro pc, nos dirigimos al fichero ***/etc/hosts*** este fichero se utiliza para obtener una relación entre un nombre de máquina y una dirección IP. Por lo tanto debemos de añadir las siguientes lineas:

```shell
#IP SERVIDOR#        #NOMBRE DEL SITIO#

192.168.100.100        www.iesgn.org 
192.168.100.100        www.departamentosgn.org
```

Y ahora solo debemos de entrar en nuestro navegador e introducir el nombre del sitio que queramos ver, por ejemplo el nombre del sitio *www.iesgn.org*. Para ver la imagen de prueba ir [aqui](https://github.com/FranJaviMN/elementos-grado/blob/main/Servicios/apache/captura-www.iesgn.png)