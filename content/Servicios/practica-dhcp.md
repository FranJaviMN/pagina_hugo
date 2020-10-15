---
title: "Practica Dhcp"
date: 2020-10-10T12:40:00+02:00
draft: true
---

# Practica DHCP

## Teoria

* **Tarea 1 (1 punto)**: Lee el documento [Teoría: Servidor DHCP](https://fp.josedomingo.org/serviciosgs/u02/dhcp.html) y explica el funcionamiento del servidor DHCP resumido en este [gráfico](https://fp.josedomingo.org/serviciosgs/u02/img/dhcp.png).

## Preparación del escenario

Crea un escenario usando Vagrant que defina las siguientes máquinas:

* Servidor: Tiene dos tarjetas de red: una pública y una privada que se conectan a la red local.

* nodolan1: Un cliente conectado a la red local.

Instala un servidor dhcp en el ordenador “servidor” que de servicio a los ordenadores de red local, teniendo en cuenta que el tiempo de concesión sea 12 horas y que la red local tiene el direccionamiento *192.168.100.0/24*.

* **Tarea 2**: Entrega el fichero *Vagrantfile* que define el escenario.

* **Tarea 3 (2 puntos)**: Muestra el fichero de configuración del servidor, la lista de concesiones, la modificación en la configuración que has hecho en el cliente para que tome la configuración de forma automática y muestra la salida del comando ` ip address`.

* **Tarea 4 (1 puntos)**: Configura el servidor para que funcione como router y NAT, de esta forma los clientes tengan internet.

* **Tarea 5 (1 punto)**: Realizar una captura, desde el servidor usando *tcpdump*, de los cuatro paquetes que corresponden a una concesión: DISCOVER, OFFER, REQUEST, ACK.

## Funcionamiento del dhcp

Vamos a comprobar que ocurre con la configuración de los clientes en determinadas circunstancias, para ello vamos a poner un tiempo de concesión muy bajo.

* **Tarea 6 (1 punto)**: Los clientes toman una configuración, y a continuación apagamos el servidor dhcp. ¿qué ocurre con el cliente windows? ¿Y con el cliente linux?

* **Tarea 7 (1 punto)**: Los clientes toman una configuración, y a continuación cambiamos la configuración del servidor dhcp (por ejemplo el rango). ¿qué ocurriría con un cliente windows? ¿Y con el cliente linux?

## Reservas

Crea una reserva para el que el cliente tome siempre la dirección 192.168.100.100.

* **Tarea 8 (1 puntos)**: Indica las modificaciones realizadas en los ficheros de configuración y entrega una comprobación de que el cliente ha tomado esa dirección.

## Uso de varios ámbitos

Modifica el escenario Vagrant para añadir una nueva red local y un nuevo nodo:

* Servidor: En el servidor hay que crear una nueva interfaz

* nodolan2: Un cliente conectado a la segunda red local.

Configura el servidor dhcp en el ordenador “servidor” para que de servicio a los ordenadores de la nueva red local, teniendo en cuenta que el tiempo de concesión sea 24 horas y que la red local tiene el direccionamiento 192.168.200.0/24.

* **Tarea 9**: Entrega el nuevo fichero Vagrantfile que define el escenario.

* **Tarea 10 (1 punto)**: Explica las modificaciones que has hecho en los distintos ficheros de configuración. Entrega las comprobaciones necesarias de que los dos ámbitos están funcionando.

* **Tarea 11 (1 punto)**: Realiza las modificaciones necesarias para que los cliente de la segunda red local tengan acceso a internet. Entrega las comprobaciones necesarias.


### Tarea 1

![Imagen DHCP](https://fp.josedomingo.org/serviciosgs/u02/img/dhcp.png "Gráfico DHCP")

Podemos ver en este gráfico el funcionamiento del protocolo DHCP(Dynamic Host Configuration Protocol) que es una extensión de protocolo BOOTP que da más flexibilidad al administrar las direcciones IP. Este protocolo puede usarse para configurar dinámicamente los parámetros esenciales TCP/IP de los hosts (estaciones de trabajo y servidores) de una red.

Este servidor posee una lista de direcciones IP dinámicas y las va asignando a los clientes conforme estas van quedando libres, sabiendo en todo momento quién ha estado en posesión de esa IP, cuánto tiempo la ha tenido y a quién se la ha asignado después. Así los clientes de una red IP pueden conseguir sus parámetros de configuración automáticamente.

Como podemos ver en la imagen sobre el funcionamiento del protoco DHCP vemos que tiene distintos estados que vamos a pasar a explicar:

El cliente comienza con el estado de inicialización “INIT”, una vez que este inicializado este espera un tiempo aleatoria entre 1 y 10 segundos para evitar colisión con otros clientes y envia un “DHCPDISCOVER broadcast” 

Después de enviar este mensaje el cliente pasa a un estado SELECTING en el que deberá aceptar un paquete DHCPOFFER enviado por el servidor.

Una vez que recibe la atención de los servidores DHCP el cliente envia un mensaje “DHCPREQUEST” para elegir el servidor que le va a asignar una dirección IP y este le contestará con un “DHCPACK”.

En el caso de que se hubiera elegido una ip duplicada, el cliente envía un paquete DHCPDECLINE y vuelve al estado INIT.

Si el paquete DHPACK es aceptado, el cliente pasa al estado BOUND, en el que recibirá un T1,T2 y T3, donde:

* T1 es el temporizador de renovación de alquiler.

* T2 es el temporizador de reenganche.

* T3 es la Duración del alquiler

Una vez que T1 expira, el cliente pasa al estado RENEWING  en el que se negocia de nuevo la asignación de ip.

Si el servidor no renueva la ip, le enviará un mensaje DHCPNACK y el cliente volverá al estado INIT e intentará obtener una ip de nuevo.

Si el servidor renueva la ip, le enviará de nuevo un mensaje DHCPACK con nuevos valores en T1,T2 y T3. De nuevo el cliente pasa al estado BOUND. 

Si T2 expira mientras el cliente DHCP está esperando en el estado RENEWING, el cliente pasará al estado REBINDING en el que envia de nuevo un DHCPREQUEST a cualquier  servidor. Si un servidor contesta con un DHCPACK, el cliente renueva su ip y sus temporizadores y pasa al estado BOUND.

En el caso de que no hubiese respuesta de ningún servidor dhcp, el cliente pasa al estado INIT para buscar un nuevo servidor.




### Tarea 2

Como hemos leido en las tareas de esta practica, ahora nos toca montar el escenario en el que vamos a ver en funcionamiento el protocolo DHCP, para ello vamos a usar el siguiente fichero ***Vagrantfile:***

```ruby
Vagrant.configure("2") do |config|
  config.vm.define :servidor do |servidor|
    servidor.vm.box = "debian/buster64"
    servidor.vm.hostname = "servidor"
    servidor.vm.network :public_network, :bridge=>"wlp2s0"
    servidor.vm.network :private_network, ip: "192.168.100.1",
    virtualbox__intnet: "red1"
  end
  config.vm.define :nodolan1 do |nodolan1|
    nodolan1.vm.box = "debian/buster64"
    nodolan1.vm.hostname = "nodolan1"
    nodolan1.vm.network :private_network, type: "dhcp",
    virtualbox__intnet: "red1"
  end
end
```

Este seria nuestro fichero de ***Vagrantfile***. Ahora lo que debemos hacer es empezar a configurar nuestros servidor DHCP.


### Tarea 3

#### Servidor

Lo priemro que debemos hacer es instalar un servidor DHCP en nuestra maquina *servidor*, para ello vamos a usar el siguiente comando:

```shell
vagrant@servidor:~$ sudo apt install isc-dhcp-server
```

Con ese comando vamos a hacer la instalacion de nuestro servidor. Veremos que cuando estamos instalado este servidor dhcp nos saldra unos errores pero no debemos tenerlos en cuenta ya que estos errores se deben a que nuestro servidor dhcp no esta configurado.

Ahora que tenemos instalado nuestro servidor DHCP debemos de entrar en la configuracion para poder hacer que funcione correctamente. Primero debemos de ver cual va a ser la interfaz de red que va a usar nuestro *servidor* para dar servicio DHCP, para ello realizamos un ***ip a*** y vemos cual vamos a escoger, en mi caso va a ser la interfaz *eth2* ya que es la que tiene asignada la ip publica. Una vez hayamos visto cual es la que vamos a usar nos vamos al fichero de configuración ***/etc/default/isc-dhcp-server*** y buscamos la linea donde pone **INTERFACESv4=""** y ahi le indicamos nuestra interfaz, en mi caso seria:

```shell
INTERFACESv4="eth2"
```

Ahora nos dirigimos al fichero de configuracion de dhcp que es ***/etc/dhcp/dhcpd.conf***. En dicho fichero debemos de poner la configuracion dhcp que nosotros necesitemos en nuestro servidor. Hay muchos [parametros](https://franjavimn.onrender.com/servicios/dhcp-ejercicio/) que podemos contemplar a la hora de configurar nuestros servidor DHCP. En este caso vamos a usar la siguiente configuracion:

```shell
subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.100 192.168.100.110;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option routers 192.168.100.1;
  option broadcast-address 192.168.100.255;
  default-lease-time 43200;
  max-lease-time 43200;
}
```

Despues de haber configurado nuestro servidor lo que debemos hacer es reiniciar el servicio para que se ponga en funcionamiento, para ello usamos el siguiente comando:

```shell
vagrant@servidor:~$ sudo systemctl restart isc-dhcp-server.service -> Reiniciamos servicio

vagrant@servidor:~$ sudo systemctl status isc-dhcp-server.service -> Vemos el status del servicio
```

Y con este proceso nuestra maquina ya debe de poder dar servicio DHCP a los clientes de nuestra red interna.

#### nodolan1

Ahora que ya tenemos el servicio dhcp configurado y activado, si nos dirigimos a nuestra maquina cliente y hacemos ***vagrant@nodolan1:~$ ip a show dev eth1*** y veremos que tiene asignada una ip del rango que nosotros le hemos dicho en nuestro fichero de configuracion del servidor dhcp y tambien la direccion de broadcast que le hemos indicado.

```shell
####Salida del comando ip address####

3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:ec:34:83 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.101/24 brd 192.168.100.255 scope global dynamic eth1
       valid_lft 431sec preferred_lft 431sec
    inet6 fe80::a00:27ff:feec:3483/64 scope link 
       valid_lft forever preferred_lft forever
```

A la hora de ver la configuracion que toma nuestro cliente se le añade a nuestro servidor unas concesiones, donde tenemos los datos de la maquina a la que se le a asignado la ip. En este fichero nos muestra el tiempo de concesion que tiene esa ip que es de 12 horas, y el tiempo de renovacion de esa concesion, que en este caso el tiempo de renovacion es de 10 min.

Para ver este fichero nos dirigimos al fichero ***/var/lib/dhcp/dhcpd.leases*** de nuestra maquina *servidor* y ahi tendremos todas las concesiones a los clientes que estan cogiendo ip de nuestro servidor dhcp.


### Tarea 4

En esta parte tenemos que hacer que nuestro *servidor* actue como router-nat para que le de internet a nuestra maquina *cliente* y asi tenga internet, para ello nos dirigimos a nuestra maquina ***servidor*** primero y realizamos el comando siguiente:

```shell
vagrant@servidor:~$ sudo ip route replace default via 192.168.1.1 -> Cambiamos la ruta por defecto
```

Una vez hecho esto tendremos internet a traves de la interfaz *eth1* en vez de tener internet a traves de *eth0*. 

Una vez hecho este comando necesitamos añadirle las reglas necesarias para que nuestra maquina actue como un router-nat y pueda enrutar correctamente las peticiones que le vamos a hacer a traves de la maquina cliente, para ello usaremos la siguiente regla de iptables:

```shell
iptables -t nat -A POSTROUTING -s 192.168.100.1/24 -o eth1 -j MASQUERADE

####Donde podemos ver los parametros siguientes:####

-t nat: Le indicamos que la regla es para la tabla nat

-A POSTROUTING: Añade (Add) una regla a la cadena POSTROUTING

-s 192.168.100.1/24: Se aplica a los paquetes que tengan como dirección origen (source) la 192.168.100.1/24

-o eth1: Se aplica a los paquetes que salgan (out-interface) por eth1

MASQUERADE: Cuando la dirección IP pública que sustituye a la IP origen es dinámica, caso bastante habitual en conexiones a Internet domésticas.
```

Una vez hecho esto debemos de hacer un ultimo paso que es activar el bit de FORWARD ya que, en principio un equipo con GNU/Linux no permite que pasen paquetes de una interfaz de red a otra, para que se permita esto y por tanto pueda funcionar el equipo como router-nat debemos de activarlo.

Podemos hacerlo para que solo se active hasta que el equipo se apaga, una vez se vuelva a encender debemos de activarlo otra vez o podemos hacerlo de forma permanente:

```shell
####Solo hasta que se apague el equipo####
echo 1 > /proc/sys/net/ipv4/ip_forward

####De forma permanente####
Nos dirigimos al fichero /etc/sysctl.conf y buscamos una linea como: net.ipv4.ip_forward=1
```

Ahora vamos a nuestra maquina ***nodolan1*** y en ella tenemos que hacer que salga a internet a traves de la interfaz por la que esta recibiendo servicio DHCP, en este caso es *eth1*. Para hacer eso posible usamos el comando siguiente:

```shell
vagrant@nodolan1:~$ sudo ip route replace default via 192.168.100.1
```

Asi cambiaremos la ruta por defecto por la que tenemos asignada en nuestro *servidor*. 

Ahora si realizamos un ***apt update*** en nuestra maquina *nodolan1* veremos que lo hace sin problemas.

### Tarea 5

Ahora vamos a realizar una captura de los paquetes que corresponden a una concesión: *DISCOVER, OFFER, REQUEST, ACK*. Para ello vamos a usar *tcpdump* para la captura de los paquetes. Debemos asegurarnos que esta instalado, normalmente no lo esta y tendremos que instalarlo usando ***vagrant@servidor:~$ sudo apt install tcpdump*** y una vez instalado usamos el siguiente comando:

```shell
vagrant@servidor:~$ sudo tcpdump -i eth2 port 67 or port 68 -e -n -vv

#### Donde podemos ver: ####

-i: Escucha en la interfaz especificada

-e: Imprime el encabezado de nivel de enlace en cada línea de volcado.

-n: No convierte las direcciones de salida

-vv: Vemos la salida de la captura al completo.
```

Podemos ver el trafico capturado en el siguiente [fichero](https://github.com/FranJaviMN/elementos-grado/blob/main/Servicios/dhcp/captura-practica.txt).

### Tarea 6

A la hora de apagar nuestro servidor dhcp, si le cambiamos la concesion a un numero muy bajo veremos que en una maquina linux, al no encontrar el servidor dhcp y no poder recibir la respuesta por parte del servidor, si hacemos un ***ip a*** veremos que ya no tiene la asignacion ip que tenia antes de apagar el servidor.


### Tarea 7

En la siguiente tarea vamos a cambiar la configuracion de nuestro servidor dhcp, para ello nos dirigimos al fichero ***/etc/dhcp/dhcpd.conf*** y una vez ahi, cambiamos el rango de ip que vamos a servir, en este caso el fichero de configuracion quedaria asi:

```shell
subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.111 192.168.100.120;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option routers 192.168.100.1;
  option broadcast-address 192.168.100.255;
  default-lease-time 60;
  max-lease-time 43200;
}
```

Y si esperamos el tiempo 1 minuto y se acaba el tiempo de concesión veremos que la ip a camabiado a una del nuevo rango:

```shell
vagrant@nodolan1:~$ ip a show dev eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:a1:26:74 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.111/24 brd 192.168.100.255 scope global dynamic eth1
       valid_lft 188sec preferred_lft 188sec
    inet6 fe80::a00:27ff:fea1:2674/64 scope link 
       valid_lft forever preferred_lft forever
```

### Tarea 8

Ahoa vamos a añadir una reserva en nuestro servidor dhcp para que, cuando nosotros tengamos a nuestro cliente y su tiempo de concesion acabe, le de una ip que nosotros hemos reservado, para ello vamos a necesitar la MAC de la interfaz de red de nuestra maquina *cliente*, para ello usamos el comando ***ip link show dev eth1***:

```shell
vagrant@nodolan1:~$ ip link show dev eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:a1:26:74 brd ff:ff:ff:ff:ff:ff
```

En mi caso, la MAC es la **08:00:27:a1:26:74** .

El siguiente paso es dirigirnos a nuestra maquina servidor e ir al fichero de configuracion ***/etc/dhcp/dhcpd.conf*** y ahí tenemos que añadir el campo *"host"* dentro de los parametros que indicamos. Deberia de quedar de la siguiente forma:

```shell
subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.111 192.168.100.120;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option routers 192.168.100.1;
  option broadcast-address 192.168.100.255;
  default-lease-time 60;
  max-lease-time 43200;

  host reserva-1 { #seccion host dentro de la subnet.
        fixed-address 192.168.100.115; #La dirección IP que le vamos a asignar.
        hardware ethernet 08:00:27:a1:26:74; #Es la dirección MAC de la tarjeta de red de la maquina cliente.
        }
}
```

Asi pues, pasado el tiempo de concesion de 1 minuto, volvemos a ver la ip de nuestro cliente y veremos que le a asignado la ip que nosotros le habiamos reservado previamente en el servidor.

### Tarea 9

Para ver el fichero final de Vagrant, ver [aqui](https://github.com/FranJaviMN/elementos-grado/blob/main/Servicios/dhcp/Vagrantfile)

### Tarea 10

Ahora que ya tenemos las maquinas levantadas, solo debemos de entrar en la maquina servidor y hacer ***ip a*** y veremos que nos ha creado otra interfaz nueva, esta vez sera la interfaz *eth3* con la ip estatica que nosotros le hemos indicado.

Ahora debemos de configurar nuestra servidor para que pueda actuar como servidor dhcp en la nueva maquina que hemos añadido, para ello debemos de ir al fichero de configuración ***/etc/default/isc-dhcp-server*** y donde dice ***INTERFACESv4=""*** deben de estar las dos interfaces por la que nuestro servidor va a repartir las ip, en este caso debe de estar asi:

```shell
INTERFACESv4="eth2 eth3"
```

Una vez hecho esto nos dirigimos al fichero de configuración ***/etc/dhcp/dhcpd.conf*** y una vez estemos dentro debemos de añadir los parametros necesarios para que nuestro servidor dhcp pueda dar direccionamiento a nuestra nueva maquina *nodolan2*, para ello nuestro fichero debe de quedar asi:

```shell
#### Parametros de la maquina nodolan1 ####

subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.100 192.168.100.110;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option routers 192.168.100.1;
  option broadcast-address 192.168.100.255;
  default-lease-time 43200;
  max-lease-time 86400;
}

#### Parametros de nodolan2 ####

subnet 192.168.200.0 netmask 255.255.255.0 {
  range 192.168.200.100 192.168.200.110;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option routers 192.168.200.1;
  option broadcast-address 192.168.200.255;
  default-lease-time 43200;
  max-lease-time 86400;
}
```

Una vez hecho lo anterior debemos de reiniciar nuestro servicio en nuestra maquina *servidor* con los comandos ***systemctl restart isc-dhcp-server.service*** y para ver su status con el comando ***systemctl status isc-dhcp-server.service***. Hechoe sto si nos vamos a nuestra maquinas *nodolan1* y *nodolan2* y realizamos un ***dhclient*** y seguido hacemos ***ip a*** veremos que nuestro servidor nos ha asignado una ip del rango ip que nosotros le hemos indicado.

```shell

#### Nodolan1 ####

vagrant@nodolan1:~$ ip addr show eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:de:f9:23 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.105/24 brd 192.168.100.255 scope global dynamic eth1
       valid_lft 21391sec preferred_lft 21391sec
    inet6 fe80::a00:27ff:fede:f923/64 scope link 
       valid_lft forever preferred_lft forever

#### Nodolan2 ####

vagrant@nodolan2:~$ ip addr show eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:ef:86:7c brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.105/24 brd 192.168.200.255 scope global dynamic eth1
       valid_lft 21373sec preferred_lft 21373sec
    inet6 fe80::a00:27ff:feef:867c/64 scope link 
       valid_lft forever preferred_lft forever
```

### Tarea 11

Ahora lo que debemos de hacer es que nuestro servidor pueda dar servicio de router-nat al igual que hicimos en la **tarea 4**.

Primero configuramos el servidor con la siguiente lista de comandos:

```shell

#### Cambiamos la ruta por defecto ####

vagrant@servidor:~$ sudo ip route replace default via 192.168.1.1

#### Añadimos las reglas de iptables ####

vagrant@servidor:~$ iptables -t nat -A POSTROUTING -s 192.168.100.1/24 -o eth1 -j MASQUERADE -> para nodolan1

vagrant@servidor:~$ iptables -t nat -A POSTROUTING -s 192.168.200.1/24 -o eth1 -j MASQUERADE -> para nodolan2

#### Activamos el bit de forward ####

vagrant@servidor:~$ echo 1 > /proc/sys/net/ipv4/ip_forward
```

Ahoara nos dirigimos a nuestras maquinas *nodolan1* y *nodolan2*:

```shell

#### nodolan1 ####

vagrant@nodolan1:~$ sudo ip route replace default via 192.168.100.1

#### nodolan2 ####

vagrant@nodolan2:~$ sudo ip route replace default via 192.168.200.1
```

Despues de estas modificaciones si hacemos un *apt update* veremos que tendremos internet a traves de nuestra maquina *servidor*