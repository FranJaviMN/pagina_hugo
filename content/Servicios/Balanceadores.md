---
title: "Balanceadores"
date: 2021-02-24T19:25:03+01:00
draft: true
---

# Balanceadores de carga

Un Balanceador de carga fundamentalmente es un dispositivo de hardware o software que se pone al frente de un conjunto de servidores que atienden una aplicación y, tal como su nombre lo indica, asigna o balancea las solicitudes que llegan de los clientes a los servidores usando algún algoritmo (desde un simple round-robin hasta algoritmos más sofisticados).

Para que se considere exitoso un ​balanceador de carga: 

* Debe minimizar tiempos de respuesta

* Mejorar el desempeño del servicio.

* Evitar la saturación

## Metodos de conexión 

* **Round-Robin**: las peticiones son distribuidas entre los servidores de forma cíclica, independientemente de la carga del servidor. Distribuye las peticiones de forma ecuánime pero la carga no.

![Round-robin](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bd/Screen_Shot_2015-09-24_at_12.54.32.png/141px-Screen_Shot_2015-09-24_at_12.54.32.png)

* **Weighted Round-Robin**: Las peticiones se entregan dependiendo del peso que se le de a cada servidor.

![Weighted Round-Robin](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/Weighted_Round-Robin.png/143px-Weighted_Round-Robin.png)

* **LeastConnection**: Cada petición es atendida por el servidor con menos conexiones activas en ese momento.

![LeastConnection](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5b/Least_Connection.png/153px-Least_Connection.png)

* **Weighted LeastConnection**: Las peticiones se entregan dependiendo del peso y el número de conexiones que se tengan

![Weighted LeastConnection](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e2/Weighted_LeastConnection.png/159px-Weighted_LeastConnection.png)

* **Ip-hash**: se selecciona el servidor que atenderá la petición con base en algún dato como la dirección IP, de esta forma todas las peticiones de un usuario son atendidas por el mismo servidor.


## Practica de balanceadores de carga

En el siguiente ejercicio que vamos a realizar vamos a utilizar un escenario en vagrant del cual vamos a necesitar distintos ficheros por lo que para conseguir el escenario podemos descargarlo desde [aqui](https://fp.josedomingo.org/serviciosgs/u08/doc/haproxy/vagrant.zip). El esquema del escenario es el siguiente:

![escenario vagrant balanceadores](https://fp.josedomingo.org/serviciosgs/u08/img/haproxy.jpg)

Una vez tengamos el escenario desplegado vamos a proceder a instalar en la maquina llamada **balanceador** el respectivo balanceador que necesitamos, en este caso va a ser el balanceador llamado **HAproxy**:
```shell
#### Instalamos HAproxy en la maquina balanceador ####
vagrant@balanceador:~$ sudo apt install haproxy
```

Una vez hayamos instalado el balanceador nos dirigimos al fichero de configuración **/etc/haproxy/haproxy.cfg** y ahi vamos a añadir las siguientes lineas que vamos a explicar a continuación:
```shell
#### Añadimos la siguiente configuracion al fichero /etc/haproxy/haproxy.cfg ####
global
    daemon
    maxconn 256
    user    haproxy
    group   haproxy
    log     127.0.0.1       local0
    log     127.0.0.1       local1  notice     
defaults
    mode    http
    log     global
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms        
listen granja_cda 
    bind 192.168.1.117:80 #aquí pon la dirección ip del balanceador
    mode http
    stats enable
    stats auth  cda:cda
    balance roundrobin
    server uno 10.10.10.11:80 maxconn 128
    server dos 10.10.10.22:80 maxconn 128
```

Donde, en la sección *listen* definimos un “proxy inverso” de nombre granja_cda que:

*  trabajará en modo http (la otra alternativa es el modo tcp, pero no analiza las peticiones/respuestas HTTP, sólo retransmite paquetes TCP) **(mode http)**

* atendiendo peticiones en el puerto 80 del balanceador

* con balanceo **round-robin** **(balance roundrobin)**

* que repartirá las peticiones entre dos servidores reales (de nombres uno y dos) en el puerto 80 de las direcciones 10.10.10.11 y 10.10.10.22

* adicionalmente, habilita la consola Web de estadísticas, accesible con las credenciales cda:cda **(stats auth  cda:cda)**

Una vez hecho esto debemos de reiniciar el balanceador de **haproxy** y lo vamos a habilitar al iniciar la maquina:
```shell
#### Reiniciamos el servicio ####
vagrant@balanceador:~$ sudo systemctl restart haproxy.service

#### Lo habilitamos al inicio de la maquina ####
vagrant@balanceador:~$ sudo systemctl enable haproxy.service 
Synchronizing state of haproxy.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable haproxy
```

Una vesz hecho esto solo debemos de irnos a nuestro navegador y poner la ip de nuestra maquina **balanceador** para asi poder ver el contenidor de las paginas que sirven los dos servidores apache ya que, al actuar como balanceador, ira mostrando un servidor u otro dependiendo de cuantas veces hayamos entrado:

![balanceador apache1](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/balanceador/apache1_balanceador.png)

![balanceador apache2](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/balanceador/apache2_balanceador.png)

Una vez visto que nuestro balanceador funciona correctamente, vamos a entrar en la zona de nuestro fichero php que se llama **http://192.168.X.X/haproxy?stats**:

![stats de haproxy](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/balanceador/haproxy_stats.png)

Una vez hecho lo anterior vamos a revisar los logs de uno de los servidores de apache que tenemos en nuestro escneario, para ello lo que vamos a hacer es entrar en dicha maquina y vamos a comprobar los logs que podemos ver en **/var/log/apache2/access.log**:
```shell
vagrant@apache1:~$ sudo cat /var/log/apache2/access.log
10.10.10.1 - - [23/Feb/2021:15:37:37 +0000] "GET / HTTP/1.1" 200 437 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
10.10.10.1 - - [23/Feb/2021:15:38:10 +0000] "GET / HTTP/1.1" 200 437 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
10.10.10.1 - - [23/Feb/2021:15:38:39 +0000] "GET /favicon.ico HTTP/1.1" 404 436 "http://192.168.1.117/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/537.36"
10.10.10.1 - - [23/Feb/2021:15:47:15 +0000] "GET /favicon.ico HTTP/1.1" 404 436 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
10.10.10.1 - - [23/Feb/2021:15:48:26 +0000] "GET /favicon.ico HTTP/1.1" 404 436 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
```

Como podemos apreciar figura como única dirección IP, la IP interna de la máquina balanceador ya que dicho balanceador hemos espcificado un balanceo tipo **round-robin**, este es que se encarga de hacer la peticiones a estos servidores por lo que solo aparece en las peticiones, las peticiones que hace el servidor **balanceador**.

### Configurar la persistencia de conexiones Web (sticky sessions)

En los servidores internos hay una aplicación PHP para trabajar con sesiones, puedes encontrar el fichero sesion.php con el siguiente contenido:
```php
<?php
     header('Content-Type: text/plain');
     session_start();
     if(!isset($_SESSION['visit']))
     {
             echo "This is the first time you're visiting this server";
             $_SESSION['visit'] = 0;
     }
     else
             echo "Your number of visits: ".$_SESSION['visit'];             

     $_SESSION['visit']++;              

     echo "\nServer IP: ".$_SERVER['SERVER_ADDR'];
     echo "\nClient IP: ".$_SERVER['REMOTE_ADDR'];
     echo "\nX-Forwarded-for: ".$_SERVER['HTTP_X_FORWARDED_FOR']."\n";
     print_r($_COOKIE);
?>
```

Vamos a añadir las opciones de persistencia de conexiones HTTP (sticky cookies) al fichero de configuración:
Contenido a incluir: (añadidos marcados con <- aquí):
```shell
#### Modificamos el fichero /etc/haproxy/haproxy.cfg ####
global
    daemon
    maxconn 256
    user    haproxy
    group   haproxy
    log     127.0.0.1       local0
    log     127.0.0.1       local1  notice     
defaults
    mode    http
    log     global
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms        
listen granja_cda 
    bind 192.168.1.117:80 #aquí pon la dirección ip del balanceador
    mode http
    stats enable
    stats auth  cda:cda
    balance roundrobin
    #server uno 10.10.10.11:80 maxconn 128
    #server dos 10.10.10.22:80 maxconn 128
    cookie PHPSESSID prefix                               # <- aquí
    server uno 10.10.10.11:80 cookie EL_UNO maxconn 128   # <- aquí
    server dos 10.10.10.22:80 cookie EL_DOS maxconn 128   # <- aquí
```

El parámetro cookie especifica el nombre de la cookie que se usa como identificador único de la sesión del cliente (en el caso de aplicaciones web PHP se suele utilizar por defecto el nombre PHPSESSID). Para cada “servidor real” se especifica una etiqueta identificativa exclusiva mediante el parámetro cookie. Con esa información HAproxy reescribirá las cabeceras HTTP de peticiones y respuestas para seguir la pista de las sesiones establecidas en cada “servidor real” usando el nombre de cookie especificado (PHPSESSID):

* conexión cliente -> balanceador HAproxy : cookie original + etiqueta de servidor

* conexión balanceador HAproxy -> servidor : cookie original

Reiniciamos el balanceador y realizamos las siguientes acciones:

* Desde el navegador web acceder varias veces a la URL **http://172.22.x.x/sesion.php** (comprobar el incremento del contador [variable de sesión])

![incremento de sesion](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/balanceador/incremento_sesion.png)

* Acceder la misma URL desde el navegador en modo texto lynx (o desde una pestaña de “incógnito” del Navegador para forzar la creación de una nueva sesión):

![nueva sesion desde privado](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/balanceador/primera_sesion.png)