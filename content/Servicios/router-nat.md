---
title: "Router Nat"
date: 2020-10-07T19:37:24+02:00
draft: true
---

# Crear router-NAT con vagrant

![Prueba](https://fp.josedomingo.org/serviciosgs/u01/img/router.png "router-nat")

Queremos automatizar la creación de la siguiente infraestructura usando Vagrant, el esquema que queremos desarrollar, que vemos en la imagen, tiene las siguientes características:

* El escenario debe tener 2 maquinas:

    * router, que está conectada a una red pública y a una red privada. La interfaz de red en la red privada se configura con la IP 10.0.0.1.

    * cliente: Esta máquina está conectada a la misma red privada que la máquina anterior, en este caso su direccionamiento es 10.0.0.2.

* La máquina router debe acceder a internet por eth1. eth0 sólo se utiliza para acceder a la máquina con vagrant ssh.

* La máquina cliente debe tener a cceso a internet. Para ello debe salir por eth1 y la máquina router debe estar configurada para enrutar las peticiones de cliente. del mismo modo, eth0 sólo se utiliza para acceder con vagrant ssh. Debes pensar que configuración debe tener la máquina cilente: puerta de enlace, configuración dns,…

## Creacion del escenario en Vagrant.

Lo primero que debemos hacer es la creacion de nuestro entorno de trabajo, en este caso consta de dos maquinas. Una maquina llamada *router* y otra maquina llamada *cliente*.

A la hora de crear un escenario con vagrant nos dirigimos al directorio donde vayamos a tener nuestro escenario y ejecutamos:

```shell
vagrant init
```

De esta forma nos generara un fichero llamado ***Vagrantfile*** que es donde nosotros vamos a indicarle a vagrant la configuracion de nuestro escenario. Si nosotros editamos el fichero nos debe de quedar una estructura como la siguiente:

```ruby
Vagrant.configure("2") do |config|

  config.vm.define :router do |router|
    router.vm.box = "debian/buster64"
    router.vm.hostname = "router"
    router.vm.network :private_network, ip: "10.0.0.1",
    virtualbox__intnet: "red1"
    router.vm.network :public_network, :bridge=>"interfaz de red propia"
  end
  config.vm.define :cliente do |cliente|
    cliente.vm.box = "debian/buster64"
    cliente.vm.hostname = "cliente"
    cliente.vm.network :private_network, ip: "10.0.0.2",
    virtualbox__intnet: "red1"
  end
```

Para ver mas información de como configurar el fichero [Vagrantfile](https://www.vagrantup.com/docs)

Una vez configurado el fichero ***Vagrantfile*** tendremos que levantar las maquinas, para ello usamos el comando:

```shell
vagrant up
```

## Configuracion de las maquinas

Una vez hayamos creado el escenario propuesto vamos a pasar a la configuración de nuestras maquinas. Para poder entrar en nuestras maquinas creadas vamos a usar el comando:

```shell
vagrant ssh (nombre de la maquina)
```

### Maquina router

Primero vamos a pasar a configurar la maquina *router*. Como podremos observar si realizamos el comando ***ip a*** vemos que tenemos 3 interfaces de red: eth0, eth1 y eth2. La interfaz *eth0* se crea automaticamente y es la que nos da servicio a internet, mientras que las interfaces *eth1* y *eth2* son las que nosotros le hemos indicado que creara en el fichero ***Vagarantfile***. En mi caso la interfaz de red *eth1* corresponde a la red privada y la interfaz *eth2* corresponde a la red publica en modo puente.

Lo primero que debemos hacer es hacer que nuestra maquina salga a internet por la interfaz puente que le hemos indicado en el fichero de configuración, para ello debemos de realizar el siguiente comando en la maquina *router*:

```shell
ip route replace default via 192.168.1.1
```

Con ese comando le indicamos a la maquina que su ruta por defecto debe de ser la del modo puente, de este modo conseguimos que nuestra maquina salga a internet por *eth2* y la interfaz *eth0* solo la usaremos para conectarnos con el comando ***vagrant ssh***

Una vez hecho este comando necesitamos añadirle las reglas necesarias para que nuestra maquina actue como un *router-nat* y pueda enrutar correctamente las peticiones que le vamos a hacer a traves de la maquina *cliente*, para ello usaremos la siguiente regla de *iptables*:

```shell
iptables -t nat -A POSTROUTING -s 10.0.0.1/24 -o eth2 -j MASQUERADE

####Donde podemos ver los parametros siguientes:####

-t nat: Le indicamos que la regla es para la tabla nat

-A POSTROUTING: Añade (Add) una regla a la cadena POSTROUTING

-s 10.0.0.1/24: Se aplica a los paquetes que tengan como dirección origen (source) la 10.0.0.1/24

-o eth2: Se aplica a los paquetes que salgan (out-interface) por eth2

MASQUERADE: Cuando la dirección IP pública que sustituye a la IP origen es dinámica, caso bastante habitual en conexiones a Internet domésticas.
```

Una vez hecho esto debemos de hacer un ultimo paso que es activar ***el bit de FORWARD*** ya que, en principio un equipo con GNU/Linux no permite que pasen paquetes de una interfaz de red a otra, para que se permita esto y por tanto pueda funcionar el equipo como router-nat debemos de activarlo.

Podemos hacerlo para que solo se active hasta que el equipo se apaga, una vez se vuelva a encender debemos de activarlo otra vez o podemos hacerlo de forma permanente:

```shell

####Solo hasta que se apague el equipo####
echo 1 > /proc/sys/net/ipv4/ip_forward

####De forma permanente####
Nos dirigimos al fichero /etc/sysctl.conf y buscamos una linea como: net.ipv4.ip_forward=1
```

Ya tendriamos nuestra maquina *router* lista para actuar como router-nat.

### Maquina cliente

En nuestra maquina cliente debemos de tener dos interfaces de red: eth0 y eth1. *eth0* es la interfaz que crea por defecto y nos da acceso a internet, pero nosotros queremos que tenga acceso a internet mediante nuestro *router-nat* que hemos estado configurando. Para hacer que nuestra maquina *cliente* tenga internet mediante el *router* debemos de hacer el siguiente comando:

```shell
ip route replace default via 10.0.0.1
```

Asi cambiaremos la ruta por defecto por la que tenemos asignada en nuestro router, asi, si nosotros hacemos ping a google o hacemos un *apt update* veremos que tenemos internet y este servicio a internet nos lo esta ofreciendo nuestro *router*.

