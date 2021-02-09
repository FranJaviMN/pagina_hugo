---
title: "Varnish + Nginx"
date: 2021-02-09T13:29:29+01:00
draft: true
---

# Varnish + Nginx

En esta practica vamos a montar un escenario con nginx y varnish para ver como mejora el rendimiento de nuestro sitio web de forma significativa pero antes vamos a ecplicar que es Varnish.

## ¿Qué es Varnish?

Varnish Cache es un acelerador de aplicaciones web, también conocido como caché de proxy HTTP inversa. Se instala delante de cualquier servidor HTTP y se configura para almacenar en el caché del servidor una copia del recurso solicitado. Está ideado para aumentar el rendimiento de aplicaciones web con contenidos pesados y APIs altamente consumidas.

Es una alternativa a otras opciones existentes que plantean caché de cliente o servidores de origen. Además Varnish Cache está orientado exclusivamente a HTTP a diferencia de otros que ofrecen soporte FTP, SMTP u otros protocolos de red. 

# Ejercicio a realizar

## Tarea 1

Vamos a configurar una máquina con la configuración ganadora: nginx + fpm_php (socket unix). Para ello ejecuta la receta ansible que encontraras en este [repositorio](https://github.com/FranJaviMN/ansible_nginx_fpm_php). Accede al wordpress y termina la configuración del sitio.

Una vez tengamos el repositorio solo debemos de ejecutar el *playbook* teniendo en cuenta que debemos de cambiar la ip que tenemos en el fichero **hosts** por la ip de nuestra maquina a la que queramos dirigir este *playbook* en mi caso a una maquina con ip **172.22.200.145**

Una vez lo tengamos debemos de terminar la instalacion de wordpress de forma manual, para ello solo debemos de entrar en nuestra pagina de wordpress y seguir.

![wordpress nginx](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/varnish/wordpress-nginx.png)

## Tarea 2

Vamos a hacer las pruebas de rendimiento desde la misma máquina, es decir vamos a ejecutar instrucciones similares a esta:
```shell
ab -t 10 -c 200 -k http://localhost/wordpress/index.php
```

Realiza algunas prueba de rendimiento con varios valores distintos para el nivel de concurrencia (50,100,250,500) y apunta el resultado de peticiones/segundo (parámetro Requests per second de ab). Puedes hacer varias pruebas y quedarte con la media. Reinicia el servidor nginx y el fpm-php entre cada prueba para que los resultados sean los más reales posibles.

Para realizar estas pruebas solo debemos de instalar el paquete donde tenemos la funcionalidad de **ab** que esta en el paquete llamado **apache2-utils** por lo que lo instalamos en nuestra maquina donde tenemos el wordpress:
```shell
debian@varnishnginx:~$ sudo apt install apache2-utils
```

Una vez hayamos instalado el paquete vamos a empezar con las pruebas, para ello vamos aumentando poco a poco el nivel de concurrencia para ver como reacciona nuestro servidor web, para ello usamos el comando que hemos puesto anteriormente y vamos a indicar que las peticiones la van a hacer 10 usuarios a la vez:
```shell
#### Nivel de concurrencia de 50 ####
debian@varnishnginx:~$ ab -t 10 -c 50 -k http://localhost/wordpress/index.php
...
Requests per second:    29.33 [#/sec] (mean)
...

#### Nivel de concurrencia de 100 ####
debian@varnishnginx:~$ ab -t 10 -c 100 -k http://localhost/wordpress/index.php
...
Requests per second:    31.71 [#/sec] (mean)
...

#### Nivel de concurrencia de 250 ####
debian@varnishnginx:~$ ab -t 10 -c 250 -k http://localhost/wordpress/index.php
...
Requests per second:    2601.04 [#/sec] (mean)
...

#### Nivel de concurrencia de 500 ####
debian@varnishnginx:~$ ab -t 10 -c 500 -k http://localhost/wordpress/index.php
...
Requests per second:    3348.60 [#/sec] (mean)
...
```

## Tarea 3

Configura un proxy inverso - caché Varnish escuchando en el puerto 80 y que se comunica con el servidor web por el puerto 8080. Entrega y muestra una comprobación de que varnish está funcionando con la nueva configuración. Realiza pruebas de rendimiento (quedate con el resultado del parámetro Requests per second) y comprueba si hemos aumentado el rendimiento. Si hacemos varias peticiones a la misma URL, ¿cuantas peticiones llegan al servidor web? (comprueba el fichero access.log para averiguarlo).

Para ello lo primero que debemos de hacer es descargar el paquete donde tenemos el programa de **Varnish**, para ello descargamos el paquete con el mismo nombre:
```shell
#### Descargamos el paquete ####
debian@varnishnginx:~$ sudo apt install varnish

#### Comprobamos su version ####
debian@varnishnginx:~$ sudo varnishd -V
varnishd (varnish-6.1.1 revision efc2f6c1536cf2272e471f5cff5f145239b19460)
Copyright (c) 2006 Verdens Gang AS
Copyright (c) 2006-2015 Varnish Software AS
```

Una vez tengamos instalado **Varnish** debemos de tener en cuenta que, por defecto, varnish escucha en el puerto **6081** y en el puerto **6082** para la interfaz de administración pero nosotros queremos que escuche en el puerto **80** por lo que primero vamos a realizar unas modificaciones en el servidor web **Nginx**

Lo primero que vamos a hacer es irnos al virtualhost que tenemos en nuestro servidor web y vamos a cambiar el puerto por donde escucha nuestro servidor web por el puerto **8080** asi que debemos de tener las siguientes lineas:
```shell
#### Cambiamos el puerto 80 por el puerto 8080 ####
server {
        listen 8080 default_server;
        listen [::]:8080 default_server;
...

#### Reiniciamos el servicio ####
debian@varnishnginx:~$ sudo systemctl restart nginx.service

#### Comprobamos que este escuchando en el puerto 8080 ####
debian@varnishnginx:~$ netstat -plntu | grep 8080
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -         
```

Una vez lo tengamos podemos ver que esta escuchando en el puerto 8080 gracias a **netstat** vemos que esta escuchando en el puerto **8080/tcp**.

Ahora que tenemos nuestro servidor web escuchando en el puerto **8080** debemos de cambiar la configuracion de **Varnish** para que acepte las peticiones por el puerto **80** y las redirija al puerto **8080** que es donde tenemos nuestro servidor web, para ello debemos de ir al fichero **/etc/varnish/default.vcl** donde le vamos a definir donde tiene que redirigir el trafico, en este caso seria a la propia maquina, es decir **localhost** y al puerto **8080**:
```shell
#### Confirguamos el fichero /etc/varnish/default.vcl ####
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

host = Dirección del servidor web.
port = Puerto por donde escucha el servidor web.
```

Una vez tengamos esta parte configurada debemos de confirgurar **Varnish** para que trabaje sobre el puerto **80** por lo que debemos de modificar el fichero **/etc/default/varnish** donde le vamos a indicar el puerto que va a escuchar:
```shell
#### Modificamos el fichero /etc/default/varnish ####
...
DAEMON_OPTS="-a :80 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
...
```

Y por ultimo debemos de configurar el demonio de systemd y cambiar los parametros por el puerto **80**, en el fichero **/lib/systemd/system/varnish.service**:
```shell
#### Configuramos el fichero /lib/systemd/system/varnish.service ####
...
ExecStart=/usr/sbin/varnishd -j unix,user=vcache -F -a :80 -T localhost:6082 -f /etc/varnish/default.vcl -S /etc/varnish/secret -s malloc,256m
...

#### Reiniciamos los servicios y demonios ####
debian@varnishnginx:~$ sudo systemctl daemon-reload

debian@varnishnginx:~$ sudo systemctl restart varnish

#### Comprobamos los puertos ####
debian@varnishnginx:~$ netstat -plntu | grep 80*
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -                   
...
```

Asi ya tendriamos configurado nuestro servidor web y varnish como proxy cache, asi que si entramos en nuestro wordpress veremos que entramos sin problemas:

![varnish cache wordpress](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/varnish/varnish-wordpress.png)

Ahora vamos a hacer unas pruebas de concurrencia como habiamos hecho cuando no teniamos **Varnish** instalado:
```shell
#### Nivel de concurrencia de 50 ####
debian@varnishnginx:~$ ab -t 10 -c 50 -k http://localhost/wordpress/index.php
...
Requests per second:    11971.84 [#/sec] (mean)
...

#### Nivel de concurrencia de 100 ####
debian@varnishnginx:~$ ab -t 10 -c 100 -k http://localhost/wordpress/index.php
...
Requests per second:    11859.68 [#/sec] (mean)
...

#### Nivel de concurrencia de 250 ####
debian@varnishnginx:~$ ab -t 10 -c 250 -k http://localhost/wordpress/index.php
...
Requests per second:    12127.14 [#/sec] (mean)
...

#### Nivel de concurrencia de 500 ####
debian@varnishnginx:~$ ab -t 10 -c 500 -k http://localhost/wordpress/index.php
...
Requests per second:    11424.25 [#/sec] (mean)
...
```

Como vemos la mejora es muy considerable, esto se debe a nuestro proxy cache que, a la hora de redirigir el trafico hacia el servidor web y este a su vez a **php-fpm** lo hace solo una vez ya que a la segunda vez ya lo tiene cacheado por lo que la respuesta es mucho mas rapida de ahi la mejora de redimiento. Si bien vamos ahora las peticiones que hace veremos que solo debe de haber una sola petición:
```shell
#### Consultamos el fichero /var/log/nginx/access.log ####
debian@varnishnginx:~$ tail -f /var/log/nginx/access.log
...
127.0.0.1 - - [09/Feb/2021:11:25:17 +0000] "GET /wordpress/index.php HTTP/1.1" 301 5 "-" "ApacheBench/2.3"
```

Como podemos comprobar solo tenemos una sola petición, aunque hayamos hecho muchas peticiones con el comando **ab**