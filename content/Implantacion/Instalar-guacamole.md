---
title: "Instalar Guacamole"
date: 2020-12-30T16:13:32+01:00
draft: true
---

# Instalcion de Guacamole

En mi caso, en la practica sobre el despliegue de una aplicacion hecha en java con Tomcar vamos a elegir el cms llamado **Guacamole**.

## ¿Qué es Guacamole apache?

Guacamole es un proyecto de escritorio remoto desarrollado por la Apache Software Foundation. Es un sistema compuesto por una parte cliente y una parte servidor. La aplicación servidor es la que se encarga de la autenticación y precisa de la instalación en infraestructura propia o acogerse la un servicio de terceros y estar accesible desde la parte cliente.

Sus partes principales son:
* Cliente web hecho en HTML5 para usar en el navegador sin necesidad de instalar ninguna aplicación ni plugin
* Almacén de las conexiones preferidas con los datos
* Soporte de varios protocolos como VNC, RSP y SSH
* API pública y bien documentada que permite la integración con otros sistemas.

## Instalacion de Guacamole en tomcat

LO primero que debemos de hacer es instalar el contenedor de servlets **tomcat** en su verison **9**, para ello lo estamos realizando en una maquina con un sistema debian 10 buster, por lo que el nombre del paquete con **tomcat** se llama **tomact9**, por lo que vamos a proceder a instalar dicho paquete en nuestra maquina:
```shell
#### Instalamos tomcat9 ####
debian@pruebas-apache-tomcat:~$ sudo apt install tomcat9
```

Una vez lo tengamos instalado vemos que como dependendia a nuestro paquete de tomcat 9 nos ha instalado el paquete **openjdk-11-jre-headless**, que corresponde a una implementación de la JVM mínima para poder ejecutar nuestros programas java.

Una vez hecho esta ya tendremos tomcat disponible escuchando en el puerto 8080:

![tomcat inicio](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/java/principio-tomcat.png)

Ya que tenemos tomcat instalado vamos a proceder a instalar el cms llamado **Guacamole**, para ello nos dirigimos a la pagina principal del proyecto guacamole y descargamos el fichero **.war**, una vez lo tengamos descargado debemos de llevarnos ese fichero al directorio donde vamos a desplegar nuestra aplicacion gaucamole que es en **/var/lib/tomcat9/webapps/** y ahi lo movemos y como resultado debemos de tener algo como lo siguiente:
```shell
debian@pruebas-apache-tomcat:/var/lib/tomcat9/webapps$ ls
guacamole-1.2.0  guacamole-1.2.0.war  ROOT
```

Como vemos, al meter el fichero .war en nuestra carpeta de despliegues nos ha generado una carpeta que es de la descompresion del fichero .war, por lo que si ahora entramos en la url **http://172.22.200.147:8080/guacamole-1.2.0/#/**

![guacamole tomcat](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/java/guacamole-tomcat.png)

De esta forma ya tenemos el cms de guacamole instalado en tomcat pero ahora queremos usar apache con tomcat para poder asi crear un proxy inverso.

## Instalacion de apache para trabajar con tomcat

Para ello lo que vamos a hacer es instalar el paquete de apache2 en debian y una vez lo tengamos instalado vamos a confirgurar el virtualhost para que sirva una pagina sencilla:
```shell
#### Instalamos apache2 ####
debian@pruebas-apache-tomcat:~$ sudo apt install apache2

#### Configuramos el virtualhost ####
debian@pruebas-apache-tomcat:~$ nano /etc/apache2/sites-available/000-default.conf

<VirtualHost *:80>
        ServerName www.prueba-tomcat.org
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

#### Generamos un fichero index.html en el documentroot ####
debian@pruebas-apache-tomcat:~$ sudo nano /var/www/html/index.html

<h1> Esta funcionando!!!! </h1>

#### Reiniciamos el servicio ####
debian@pruebas-apache-tomcat:~$ sudo systemctl reload apache2.service
```

Ahora si entramos en **www.prueba-tomcat.org** nos debe de salir algo como lo siguiente:

![apache2 funcionando](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/java/apache2-funcionando.png)

Ahora que ya tenemos nuestro apache funcionando vamos a conectar tanto apache2 como tomcat para asi poder usarlo como proxy inverso, para ello primero debemos de instalar unos modulos de apache para poder realizar la conexion entre ellos:
```shell
#### Instalamos los modulos necesarios ####
debian@pruebas-apache-tomcat:~$ sudo apt install libapache2-mod-jk

#### Habilitamos los modulos de los conectores ####
debian@pruebas-apache-tomcat:~$ sudo a2enmod proxy proxy_http
```

UNa vez hayamos hecho el paso anterior debemos de irnos al virtualhost que tenemos creado y ahi debemos de añadir las siguientes lineas:
```shell
<Location /guacamole/>
    Order allow,deny
    Allow from all
    ProxyPass http://localhost:8080/guacamole-1.2.0/ flushpackets=on
    ProxyPassReverse http://localhost:8080/guacamole-1.2.0/
</Location>
```

Y en nuestro fichero index.html vamos a añadir lo siguiente:
```html
<p>enlace hacia guacamole</p><a href=http://www.prueba-tomcat.org/guacamole/#/> Ir a guacamole </a>
```

Reinciamos el servicio de apache y al entrar en la url http://www.prueba-tomcat.org/ veremos un enlace que nos llevara hasta la aplicacion de guacamole:

![guacamole tomcat apache](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/java/guacamole-apache-tomcat.png)