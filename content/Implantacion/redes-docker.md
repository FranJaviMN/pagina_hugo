---
title: "Redes Docker"
date: 2021-02-09T13:22:13+01:00
draft: true
---

# Redes en Docker

Aunque hasta ahora no lo hemos tenido en cuenta, cada vez que creamos un contenedor, esté se conecta a una red virtual y docker hace una configuración del sistema (usando interfaces puente e iptables) para que la máquina tenga una ip interna, tenga acceso al exterior, **podamos mapear (DNAT)** puertos,…)

```shell
debian@omega:~$ docker run -it --rm debian bash -c "ip a"
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
22: eth0@if23: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

Como vemos en el ejemplo anterior, podemos usar un contenedor para que ejecute un comando y cuando lo ejecute, se borra ese contenedor. Como vemos, por defecto, el contenedor docker tiene un ip del rango 172.17.0.0/16 ademas podemos ver que a creado una red del tipo *bridge* que es donde se conectan los contenedores por defecto:
```shell
debian@omega:~$ ip a show docker0 
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:4c:54:e0:83 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:4cff:fe54:e083/64 scope link 
       valid_lft forever preferred_lft forever
```

Ademas podemos ver que tambien se han creado distintas reglas en *iptables* para crear el **DNAT** en los contenedores, para ver esas reglas debemos de usar el comando **iptables -L -n** e **iptables -L -n - t nat**

## Tipos de redes en Docker

Cuando nosotros instalamos Docker tenemos una serie de redes ya definidas, para poder ver esas redes que ya tenemos definidas solo debemos de usar el comando **docker network ls**
```shell
debian@omega:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
39aa24802bd8        bridge              bridge              local
62096e07c04b        host                host                local
213740da8bd0        none                null                local
```

Como vemos tenemos 3 redes ya definidas al instalar docker y vemos, como habiamos hablado en el apartado anterior, que tenemos la red de tipo *Bridge* llamada **bridge** en la cual se conectan por defecto todos lo contenedores que nosotros vayamos creando. Las redes de tipo *bridge* nos va a permitir realizar las siguientes acciones:

* Aislar los distintos contenedores que tengo en distintas subredes docker, de tal manera que desde cada una de las subredes solo podremos acceder a los equipos de esa misma subred.

* Aislar los contenedores del acceso exterior.

* Publicar servicios que tengamos en los contenedores mediante redirecciones que docker implementará con las pertinentes reglas de iptables.

![imagen sobre bridge](https://docs.docker.com/engine/tutorials/bridge1.png)

Ahora bien, como vemos tambien tenemos una red de tipo *host* con el nombre **host** que, como su propio nombre indica, al usar esta red el contenedor creado va a usar el direccionamiento que tenga nuestra maquina en ese momento por lo que va a tomar un direccionamiento **DHCP** de nuestra red ademas de que los puertos son directamente accesibles desde nuestra maquina:
```shell
#### Creamos un contenedor con la red host ####
debian@omega:~$ docker run -d --name prueba-host --network host httpd:2.4
```

Y por ultimo tambien tenemos otra red del tipo *none* con el nombre **none** la cual no configurará ninguna IP para el contenedor y no tiene acceso a la red externa ni a otros contenedores. Tiene la dirección loopback y se puede usar para ejecutar trabajos por lotes.

## Gestionar las redes en Docker

Tenemos que hacer una diferenciación entre dos tipos de redes bridged:

* Tenemos que hacer una diferenciación entre dos tipos de redes bridged

* Y las redes “bridged” definidas por el usuario.

Esta red “bridged” por defecto, que es la usada por defecto por los contenedores, se diferencia en varios aspectos de las redes “bridged” que creamos nosotros. Estos aspectos son los siguientes:

* Las redes que nosotros definamos proporcionan resolución DNS entre los contenedores, cosa que la red por defecto no hace a no ser que usemos opciones que ya se consideran “deprectated(obsoleta)” (--link).

* Puedo conectar en caliente a los contenedores redes “bridged” definidas por el usuario. Si uso la red por defecto tengo que parar previamente el contenedor

* Me permite gestionar de manera más segura el aislamiento de los contenedores, ya que si no indico una red al arrancar un contenedor éste se incluye en la red por defecto donde pueden convivir servicios que no tengan nada que ver

* Tengo más control sobre la configuración de las redes si las defino yo. Los contenedores de la red por defecto comparten todos la misma configuración de red (MTU, reglas ip tables etc…).

* **Los contenedores dentro de la red “bridge” comparten todos ciertas variables de entorno lo que puede provocar ciertos conflictos.**

En resumen, **Es importante que nuestro contenedores en producción se estén ejecutando sobre una red definida por el usuario.**

Para la gestión de redes en Docker tenemos diversos comandos que podemos usar para ello:

* docker network ls: Nos permite hacer un listado de las redes que tenemos creadas
```shell
debian@omega:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
39aa24802bd8        bridge              bridge              local
62096e07c04b        host                host                local
213740da8bd0        none                null                local
```

* docker network create: Creacion de una red en docker, por ejemplo:
   * docker network create red1
   * docker network create -d bridge --subnet 172.24.0.0/16 --gateway 172.24.0.1 red2

* docker network rm/prune: Borrar redes. Teniendo en cuenta que no puedo borrar una red que tenga contenedores que la estén usando. deberé primero borrar los contenedores o desconectar la red.

* docker network inspect: Nos da información de la red.

Ahora como ejemplo vamos a crear una nueva red de tipo *bridged* la cual va a tener el direccionamiento **192.168.100.0/24** y como puerta de entrada la ip **192.168.100.1** y la vamos a llamar *red_prueba1*:
```shell
#### Creamos la nueva con la información del ejemplo ####
debian@omega:~$ docker network create -d bridge --subnet 192.168.100.0/24 --gateway 192.168.100.1 red_prueba1
835aad1ce96c0fe7cc603acac9646a62fd7529209fb5b517ad7f49bb4db88f5c

#### Listamos las redes y buscamos nuestra nueva red ####
debian@omega:~$ docker network ls | grep red_prueba1
835aad1ce96c        red_prueba1         bridge              local

#### Vamos a ver informacion respecto a esa red ####
debian@omega:~$ docker network inspect red_prueba1
[
    {
        "Name": "red_prueba1",
        "Id": "835aad1ce96c0fe7cc603acac9646a62fd7529209fb5b517ad7f49bb4db88f5c",
        "Created": "2021-02-08T08:44:32.948922711Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.100.0/24",
                    "Gateway": "192.168.100.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

#### Por ultimo vamos a borrar esa red ####
debian@omega:~$ docker network rm red_prueba1
red_prueba1
```

## Asociación de redes a los contenedores

Imaginemos que he creados dos redes definidas por el usuario:
```shell 
debian@omega:~$ docker network create --subnet 172.28.0.0/16 --gateway 172.28.0.1 red1
4347de46ffe40411d303bd056a5384ac925bc232fc90c04c8747a4d478f8d306

debian@omega:~$ docker network create red2
0d5418612c7832ec3855e3949bb04f9651761f6147fd163e38a0b5742d225f7c
```

En un primer momento vamos a trabajar solo con la *red1*, por lo que vamos a crear un nuevo contenedor:
```shell
debian@omega:~$ docker run -d --name my-apache-app --network red1 -p 8080:80 httpd:2.4
```

Vamos a comprobar la resolución DNS que tiene nuestra *red1* creando un contenedor interactivo para poder comprobarlo:
```shell
#### Creamos el contenedor ####
debian@omega:~$ docker run -it --name prueba_dns --network red1 debian bash
root@02acd60f5507:/#

#### Instalamos los paquetes necesarios ####
root@02acd60f5507:/# apt update && apt install dnsutils -y

#### Hacemos una consulta dns con dig ####
root@02acd60f5507:/# dig my-apache-app

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> my-apache-app
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49802
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;my-apache-app.			IN	A

;; ANSWER SECTION:
my-apache-app.		600	IN	A	172.28.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Mon Feb 08 08:54:07 UTC 2021
;; MSG SIZE  rcvd: 60
```

Como podemos ver, al consultar en nuestro DNS sobre el nombre **my-apache-app**, vemos que nos responde con la ip del contenedor que hemos creado con ese nombre. Tanto al crear un contenedor con el flag --network, como con la instrucción docker network connect, podemos usar algunos otros flags:
* --dns: para establecer unos servidores DNS predeterminados.

* --ip6: para establecer la dirección de red ipv6

* --hostname o -h: para establecer el nombre de host del contenedor. Si no lo establezco será el ID del mismo.

## Ejercicios 

En este ejercicio vamos a instalar una aplicación wordpress y una base de dataos mariadb, por lo que vamos a necesitar dos contendores, uno para wordpress y otro para mariadb. Los dos contenedores deben de estar en la misma red y deben de tener acceso por nombre, es decir, resolución DNS por lo que vamos a necesitar una nueva red que debemos de crear. Por lo que vamos a empezar creando nuestra nueva red:
```shell
#### Creamos una nueva red llamada red_wordpress ####
debian@omega:~$ docker network create -d bridge red_wordpress
72286220eb797320840226accfc0e0f35fc8c16e065f14d34e210c5e91c1f348

#### Creamos los contenedores en la red_wordpress ####
debian@omega:~$ docker run -d --name servidor_mysql --network red_wordpress -v /home/debian/red_wordpress/mysql/:/var/lib/mysql -e MYSQL_DATABASE=bd_wordpress -e MYSQL_USER=user_wordpress -e MYSQL_PASSWORD=passuser -e MYSQL_ROOT_PASSWORD=passroot mariadb

debian@omega:~$ docker run -d --name servidor_wordpress --network red_wordpress -v /home/debian/red_wordpress/wordpress/:/var/www/html/wp-content -e WORDPRESS_DB_HOST=servidor_mysql -e WORDPRESS_DB_USER=user_wordpress -e WORDPRESS_DB_PASSWORD=passuser -e WORDPRESS_DB_NAME=bd_wordpress -p 8080:80 wordpress

#### Vemos que este corriendo correctamente ####
debian@omega:~$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
b547c614ad5b        wordpress           "docker-entrypoint.s…"   3 seconds ago       Up 1 second         0.0.0.0:8080->80/tcp   servidor_wordpress
f836f37bacea        mariadb             "docker-entrypoint.s…"   10 seconds ago      Up 8 seconds        3306/tcp               servidor_mysql
```

Ahora si nos dirigirmos a nuestro navegador entrando en **http://nuestra-ip:8080** veremos que entramos en los ultimos pasos para completar wordpress, los rellenamos y ya tendriamos listo nuestro wordpress, que esta aislado de las otras redes en una red propia junto a una base de datos mariadb

![wordpress en red propia](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/docker/wordpress-red-docker.png)