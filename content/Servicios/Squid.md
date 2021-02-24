---
title: "Squid"
date: 2021-02-24T19:25:28+01:00
draft: true
---

# Proxy

En esta práctica vamos a instalar un proxy squid para configurar nuestro cliente para que acceda a internet por medio de este proxy.

Vamos a usar el fichero Vagrantfile para crear el escenario: un ordenador llamado proxy donde instalaremos squid y un cliente interno (cliente_int).
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|
  config.vm.define :proxy do |proxy|
      proxy.vm.box = "debian/buster64"
      proxy.vm.hostname = "proxy"
      proxy.vm.network :private_network, ip: "10.0.0.10", virtualbox__intnet: "red_privada1"
      proxy.vm.network :private_network, ip: "192.168.200.10"
    end
    config.vm.define :cliente_int do |cliente_int|
      cliente_int.vm.box = "debian/buster64"
      cliente_int.vm.network :private_network, ip: "10.0.0.11",virtualbox__intnet: "red_privada1"
    end
    
  end
```

## ¿Que es un Proxy?

Un proxy es un servidor que ofrece, a nivel de aplicación, la posibilidad de conectar dos redes. Habitualmente se utiliza para hacer de intermediario entre una red local y el propio internet. Como conseguimos que todo el tráfico de nuestra red pase por un solo nodo, podemos aprovechar para aplicar filtros, situar una caché en esta máquina, etc. Durante bastante tiempo los proxys han generado bastante controversia, ya que coarta (en función del nivel de los filtros de búsquedas que apliquemos) la libertad de los usuarios.

No obstante en diversas ocasiones se ha utilizado principalmente como caché, cuyo fin era simplemente mejorar el rendimiento de la navegación web. Con la proliferación de las páginas dinámicas, cada vez tiene menos sentido la caché tanto de los servidores proxy como la de los navegadores.

## Tarea 1

Instala squid en la máquina squid y configúralo para que permita conexiones desde la red donde este tu ordenador.

Para ello, en nuestra maquina **proxy** vamos a instalar el proxy **squid**, para ello vamos a usar el gestor de paquetes **apt** y vamos a instalarlo desde repositorios:
```shell
#### Instalamos el paquete ####
vagrant@proxy:~$ sudo apt update && sudo apt install squid3 -y

#### Comprobamos su version ####
vagrant@proxy:~$ sudo squid3 -v
Squid Cache: Version 4.6
Service Name: squid
Debian linux
...
```

Ahora debemos de dirigirnos a nuestro equipo **cliente_int** en el cual debemos de cambiar los enrutamientos para que asi pueda pasar por nuestra maquina **proxy**, para ello debemos de indicar los siguientes comandos en nuestro **client_int**:
```shell
#### Cambiamos el enrutamiento ####
vagrant@buster:~$ sudo ip r del default

vagrant@buster:~$ sudo ip r add default via 10.0.0.10 dev eth1

#### Comprobamos que su enrutamiento es a traves de 10.0.0.10 ####
vagrant@buster:~$ ip r
default via 10.0.0.10 dev eth1 
...
```

Ahora que ya tenemos nuestro equipio y el cliente conectados a nuestra maquina con el proxy, vamos a proceder a configurar **squid**, para ello debemos de irnos al fichero **/etc/squid/squid.conf** el cual es preferible hacer una copia de seguridad antes de empezar:
```shell
#### Creamos una copia de seguridad ####
vagrant@proxy:~$ sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.old

#### Vaciamos el fichero para configurarlo ####
root@proxy:~# echo "" > /etc/squid/squid.conf

#### Dentro del fichero /etc/squid/squid.conf ####
##############################################################################
acl localnet src 10.0.0.0/24            # Red interna de virtualbox (cliente_int)
acl localnet src 192.168.200.0/24         # Red del adaptador puente (Mi ordenador)

# Puertos permitidos
acl SSL_ports port 443		# https
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 1025-65535  # unregistered ports
acl CONNECT method CONNECT

http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access deny manager

include /etc/squid/conf.d/*

http_access allow localhost
http_access allow localnet

http_access deny all

http_port 8080

coredump_dir /var/spool/squid
##############################################################################
```

Con esta configuración estamos creando una ACL llamada localnet, que está definida por las dos redes definidas en el fichero *Vagrantfile*. De esta forma solo permitimos el acceso a través del proxy a las redes **10.0.0.0/24** y **192.168.200.0/24**, además de uso exclusivo de los puertos registrados 443, 80, y 21.

## Tarea 3

Configura squid para que pueda ser utilizado desde el cliente interno. En el cliente interno configura el proxy desde la línea de comandos (con una variable de entorno). Fíjate que no hemos puesto ninguna regla SNAT y podemos navegar (protocolo HTTP), pero no podemos hacer ping o utilizar otro servicio.

En este caso, buscando en la documentación he descubierto que la variable de entorno que debemos de definir en nuestro equipo **client_int** es una variable llamada **http_proxy**, por lo que ahi debemos de indicarle la ip y el puerto de nuestro servidor proxy, por lo que debe de quedarnos algo como lo siguiente:
```shell
#### Añadimos la variable de entorno ####
vagrant@buster:~$ export http_proxy=http://10.0.0.10:8080/

#### Comprobamos que podemos navegar ####
vagrant@buster:~$ wget www.google.com
--2021-02-23 11:58:45--  http://www.google.com/
Connecting to 10.0.0.10:8080... connected.
Proxy request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘index.html.1’

index.html.1                              [ <=>                                                                   ]  13.56K  --.-KB/s    in 0.02s   

#### Comprobamos ahora el log de acceso ####
vagrant@proxy:~$ sudo tail -f /var/log/squid/access.log 
1614080580.375    204 10.0.0.11 TCP_MISS/200 14721 GET http://www.google.es/ - HIER_DIRECT/142.250.185.3 text/html
1614081498.752    390 10.0.0.11 TCP_MISS/301 366 GET http://www.franjavimn.onrender.com/ - HIER_DIRECT/34.83.64.96 text/html
1614081525.240    350 10.0.0.11 TCP_MISS/200 14677 GET http://www.google.com/ - HIER_DIRECT/142.250.185.4 text/html
1614081533.220    132 10.0.0.11 TCP_MISS/503 4157 GET http://www.yputube.com/ - HIER_NONE/- text/html
```

## Tarea 4 y 5

Con squid podemos filtrar el acceso por url o dominios, realiza las configuraciones necesarias para implementar un filtro que funcione como lista negra (todo el acceso es permitido menos las url o dominios que indiquemos en un fichero.)

Realiza las configuraciones necesarias para implementar un filtro que funcione como lista blanca (todo el acceso es denegado menos las url o dominios que indiquemos en un fichero.)

Para gestionar el acceso a internet utilizaremos listas blancas y negras. Una lista blanca se encarga de permitir solo el acceso a los dominios que figuran en ella y deniegan todo lo demás. Por el contrario, una lista negra deniega los dominios que contiene y permite el acceso a todo lo demás.

### Lista negra

Para ello vamos a irnos a nuestro fichero de configuracion **/etc/squid/squid.conf** y ahi vamos a definir una nueva acl en la que le vamos a indicar el fichero donde vamos a tener los dominios que no vamos a dejar que se puedan visitar con nuestr proxy, por lo que procedemos a configurar:
```shell
#### Añadimos la nueva acl a nuestro fichero ####
# Lista negra
acl blacklist dstdomain "/etc/squid/blacklist.txt"
http_access deny blacklist

#### Creamos el fichero blacklist.txt ####
vagrant@proxy:~$ sudo nano /etc/squid/blacklist.txt

www.google.es
```

De esta forma, si intentamos entrar en www.google.es usando nuestro proxy, veremos que no nos dejara acceder:
```shell
vagrant@buster:~$ wget www.google.es
--2021-02-23 12:43:10--  http://www.google.es/
Connecting to 10.0.0.10:8080... connected.
Proxy request sent, awaiting response... 403 Forbidden
2021-02-23 12:43:10 ERROR 403: Forbidden.

#### Log de squid ####
...
1614084189.853      3 10.0.0.11 TCP_DENIED/403 3937 GET http://www.google.es/ - HIER_NONE/- text/html
...
```

### Lista blanca

En este caso vamos a crear una lista blanca y el proceso de creación de esta es muy similar a la lista negra. Lo primero que debemos de hacer es difinir una nueva acl en la que le vamos a indicar nuestra lista blanca, para ello usamos las siguientes opciones en el fichero **/etc/squid/squid.conf**:
```shell
#### Añadimos las siguientes lineas ####
# Lista blanca
acl whitelist dstdomain "/etc/squid/whitelist.txt"
http_access deny !whitelist

#### Creamos el fichero whitelist.txt ####
vagrant@proxy:~$ sudo nano /etc/squid/whitelist.txt

www.google.com

#### Cargamos la nueva configuracion ####
vagrant@proxy:~$ sudo squid -k reconfigure
```

Ahora vamos a probar a intentar entrar en **www.google.es** y en **www.google.com** y en otras paginas y veremos que solo nos dejara en los dominios que tebfamos en nuestra lista blanca:
```shell
#### Probamos a www.google.es ####
vagrant@buster:~$ wget www.google.es
--2021-02-23 12:47:57--  http://www.google.es/
Connecting to 10.0.0.10:8080... connected.
Proxy request sent, awaiting response... 403 Forbidden
2021-02-23 12:47:57 ERROR 403: Forbidden.

#### Probamos a www.google.com ####
vagrant@buster:~$ wget www.google.com
--2021-02-23 12:47:59--  http://www.google.com/
Connecting to 10.0.0.10:8080... connected.
Proxy request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘index.html.4’

index.html.4                              [ <=>                                                                   ]  13.60K  --.-KB/s    in 0.02s   

2021-02-23 12:48:00 (779 KB/s) - ‘index.html.4’ saved [13930]

#### Vemos los logs de squid ####
...
1614084479.703    334 10.0.0.11 TCP_MISS/200 14725 GET http://www.google.com/ - HIER_DIRECT/142.250.185.4 text/html
1614084483.414      0 10.0.0.11 TCP_DENIED/403 3937 GET http://www.google.uk/ - HIER_NONE/- text/html
1614084486.536      0 10.0.0.11 TCP_DENIED/403 3937 GET http://www.google.it/ - HIER_NONE/- text/html
1614084491.461      0 10.0.0.11 TCP_DENIED/403 3937 GET http://www.google.jp/ - HIER_NONE/- text/html
...
```