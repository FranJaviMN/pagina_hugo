---
title: "Dhcp Ejercicio"
date: 2020-10-06T19:05:29+02:00
draft: true
---

# Ejercicio: Instalación y configuración del servidor dhcp en linux
En las siguientes tareas vamos ver como podemos instalar y configurar un servidor dhcp en una maquina vagrant para que este proporcione direcciones dinamicas a otras maquinas clientes.

## Instalción del servidor isc-dhcp-server

Para poder hacer una instalacion del servidor isc-dhcp-server en nuestro servidor, para ello nos vamos a la linea de comandos y escribimos:

```shell
vagrant@servidor:~$ sudo apt install isc-dhcp-server
```

Con ese comando vamos a hacer la instalacion de nuestro servidor. Veremos que cuando estamos instalado este servidor dhcp nos saldra unos errores pero no debemos tenerlos en cuenta ya que estos errores se deben a que nuestro servidor dhcp no esta configurado.

## Configuración del servidor isc-dhcp-server

Lo primero que debemos de hacer es indicar a nuestro servidor que interfaz de red por la que va a trabajar dhcp, para ello debemos de irnos al siguiente fichero de configuración:

```shell
vagrant@servidor:~$ sudo nano /etc/default/isc-dhcp-server
```

Cuando estamos en el fichero nos fijamos que hay una linea en la que pone **INTERFACESv4=""**. Nosotros nos vamos a esta linea y le indicamos la interfaz que va a usar nuestro servidor dhcp, en este caso la mia es la eth1:

```shell
INTERFACESv4="eth1"
```

Ahora nos dirigimos al fichero de configuracion de dhcp que es ***/etc/dhcp/dhcpd.conf***. En dicho fichero debemos de poner la configuracion dhcp que nosotros necesitemos en nuestro servidor, todos los parametros que se pueden especificar son:

* Subnet: Especifican rangos de direcciones IPs que serán cedidas a los clientes que lo soliciten.

* max-lease-time: Tiempo de la concesión de la dirección IP

* default-lease-time: Tiempo de renovación de la concesión

* option routers: Indicamos la dirección red de la puerta de enlace que se utiliza para salir a internet.

* option domain-name-servers: Se pone las direcciones IP de los servidores DNS que va a utilizar el cliente.

* option domain­-name: Nombre del dominio que se manda al cliente.

* option subnet­mask: Subred enviada a los clientes.

* option broadcast-­address: Dirección de difusión de la red.

Al indicar una sección subnet tenemos que indicar la dirección de la red y la mascara de red y entre llaves podemos poner los siguientes parámetros:

* range: Indicamos el rango de direcciones IP que vamos a asignar.

* Algunos de los parámetros que hemos explicado en la sección principal.

Ejemplo de configuración de la sección subnet puede ser:

```shell
subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.100 192.168.100.110;
  option routers 192.168.100.1;
  option domain-name-servers 80.58.0.33, 80.58.32.9;
}
```

Reinciciamos el servidor dhcp:

```shell
vagrant@servidor:~$ sudo systemctl restart isc-dhcp-server.service
```

### Ejercicio 1
1. Configura el servidor dhcp con las siguientes características

    * Rango de direcciones a repartir: 192.168.0.100 - 192.168.0.110
    * Máscara de red: 255.255.255.0
    * Duración de la concesión: 1 hora
    * Puerta de enlace: 192.168.0.1
    * Servidores DNS: 8.8.8.8, 8.8.4.4

2. Configura los clientes para obtener direccionamiento dinámico. Comprueba las configuraciones de red que han tomado los clientes. Visualiza el fichero del servidor donde se guarda las configuraciones asignadas.

#### Tarea 1
Para poder realizar este ejercicio necesitaremos minimos dos maquinas para poder hacer un servidor y un cliente, para ello vamos a usar **vagrant** para la creacion de estas dos maquinas y usaremos el siguiente contenido en el ***Vagrantfile***:

```ruby
Vagrant.configure("2") do |config|
  config.vm.define :servidor do |servidor|
    servidor.vm.box = "debian/buster64"
    servidor.vm.hostname = "servidor"
    servidor.vm.network :private_network, ip: "192.168.0.1",
    virtualboxintnet: "red1"
  end
  config.vm.define :nodolan1 do |nodolan1|
    nodolan1.vm.box = "debian/buster64"
    nodolan1.vm.hostname = "nodolan1"
    nodolan1.vm.network :private_network, type: "dhcp",
    virtualboxintnet: "red1"
  end
end
```

Una vez tengamos ambas maquinas creadas nos vamos a la maquina "servidor" mediante **vagrant ssh servidor** y una vez dentro seguimos los pasos de instalacion del servidor dhcp hasta la parte donde añadimos la interfaz de red que vamos a usar, en este caso usaremos eth1 por lo que debemos de tener en el fichero ***/etc/default/isc-dhcp-server*** y tener el parametro **INTERFACESv4="eth1"**. 

Una vez tengamos este paso nos dirigimos al fichero de configuracion de dhcp y ponemos lo siguiente:

```shell
subnet 192.168.0.1 netmask 255.255.255.0 {
  range 192.168.0.100 192.168.0.110;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option routers 192.168.0.1;
  option broadcast-address 192.168.0.255;
  default-lease-time 600;
  max-lease-time 3600;
}
```

Una vez hecho este ultimo paso, reiniciamos nuestro servicio con:

```shell
vagrant@servidor:~$ sudo systemctl restart isc-dhcp-server.service

Y vemos el estado del servicio veremos que esta activo:

vagrant@servidor:~$ sudo systemctl status isc-dhcp-server.service
```

#### Tarea 2

Ahora que ya tenemos el servicio dhcp configurado y activado, si nos dirigimos a nuestra maquina cliente y hacemos ***vagrant@nodolan1:~$ ip a show dev eth1*** y veremos que tiene asignada una ip del rango que nosotros le hemos dicho en nuestro fichero de configuracion del servidor dhcp y tambien la direccion de broadcast que le hemos indicado.

A la hora de ver la configuracion que toma nuestro cliente se le añade a nuestro servidor unas concesiones, donde tenemos los datos de la maquina a la que se le a asignado la ip. En este fichero nos muestra el tiempo de concesion que tiene esa ip, y el tiempo de renovacion de esa concesion, que en este caso el tiempo de renovacion es de 10 min.

Para ver este fichero nos dirigimos al fichero ***/var/lib/dhcp/dhcpd.leases*** y ahi tendremos todas las concesiones a los clientes que estan cogiendo ip de nuestro servidor dhcp.

### Ejercicio 2

1. Crea en el servidor dhcp una sección HOST para conceder a un cliente una dirección IP determinada (por ejemplo la 192.168.0.105)

2. Obtén una nueva dirección IP en el cliente y comprueba que es la que has asignado por medio de la sección host.

**Realiza las siguientes comprobaciones**

Vamos a comprobar que ocurre con la configuración de los clientes en determinadas circunstancias, para ello vamos a poner un tiempo de concesión muy bajo.

1. Los clientes toman una configuración, y a continuación apagamos el servidor dhcp. ¿qué ocurre con el cliente windows? ¿Y con el cliente linux?

2. Los clientes toman una configuración, y a continuación cambiamos la configuración del servidor dhcp (por ejemplo el rango). ¿qué ocurre con el cliente windows? ¿Y con el cliente linux?

#### Tarea 1

En esta tarea debemos de hacer que una ip determinada (en este caso usaremos la 192.168.0.105) pueda ser asignada a un solo dispositivo y que dicho dispositivo al hacer una peticion dhcp a nuestro servidor este solo le asigne la ip que hemos mencionado anteriormente. 

Para realizar este proceso tenemos que indicar en el fichero de configuracion ***/etc/dhcp/dhcpd.conf*** y, esto es muy importante, para realizar esta reserva debemos de indicarla dentro de la subnet que tengamos definida, en este caso la subnet de la tarea anterior:

```shell
subnet 192.168.0.1 netmask 255.255.255.0 {
  range 192.168.0.100 192.168.0.110;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option routers 192.168.0.1;
  option broadcast-address 192.168.0.255;
  default-lease-time 600;
  max-lease-time 3600;

  host reserva-1 { #seccion host dentro de la subnet
        fixed-address 192.168.0.105; # La dirección IP que le vamos a asignar.
        hardware ethernet 08:00:27:38:ee:c6; # Es la dirección MAC de la tarjeta de red del host.
        }
}
```

De esta forma tendremos que, cuando nuestra maquina cliente haga una peticion dhcp a nuestro servidor se le asignara la ip que le hemos indicado en el fichero de configuración. Para que nos de la ip solo debemos de renovar nuestra concesion con ***dhclient*** y al hacer ***ip a show dev eth1*** veremos que nos han asignado la ip 192.168.0.105 .