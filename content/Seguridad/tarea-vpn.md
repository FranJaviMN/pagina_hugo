---
title: "Tarea Vpn"
date: 2021-02-10T14:20:10+01:00
draft: true
---

# VPN con OpenVPN

En esta parte del tema vamos a trabajar con **Redes Privadas Virtuales** que es una red privada que usa una red pública(normalmente Internet) para conectar sucursales o usuarios.

Existen tres tipos de **VPN** distintas:

* Conexiones de clientes de acceso remoto: Dan solución a los trabajadores móviles que deben poder usar los recursos de lared interna desde localizaciones remotas

* Comunicación site-to-site: Dos o más redes locales son unidas a través de una VPN formando una intranet extendida. Oficinas remotas.

* Acceso controlado dentro de una red local: Usado en muchas ocasiones para proteger la red wifi

## Clientes de acceso remoto

![imagen vpn acceso remoto](https://openvpn.net/wp-content/uploads/vpn_server_resources/Secure-Remote-Access.svg)

## Site-To-Site

![imagen vpn site-to-site](https://openvpn.net/wp-content/uploads/vpn_server_resources/layer-3-routing-diagram.png)

# OpenVPN

OpenVPN es una herramienta de conectividad basada en software libre: SSL (Secure Sockets Layer), VPN Virtual Private Network (red virtual privada). OpenVPN ofrece conectividad punto-a-punto con validación jerárquica de usuarios y host conectados remotamente. Resulta una muy buena opción en tecnologías Wi-Fi (redes inalámbricas IEEE 802.11) y soporta una amplia configuración, entre ellas balanceo de cargas. Está publicado bajo la licencia GPL, de software libre.

## Sus caracteristicas principales son:

* El componente principal es el driver tun/tap utilizado para simular interfaces dered, que se encarga de levantar el túnel y encapsular los paquetes a través del enlace virtual.

* Encriptación y autenticación con OpenSSL

* Utiliza un único puerto TCP o UDP→fácil para firewalls

* Multiplataforma→misma herramienta funcionando sobre distintos SO vs implementaciones diferentes de un mismo estándar en distintas arquitecturas

* Compresión de datos LZO

Aunque tambien podemos encontrar algunos problemas que tiene esta tecnología:

* No es compatible con IPSec, el estándar para soluciones VPN

* Comunidad no muy amplia

* Faltan dispositivos con clientes OpenVPN integrados

## Modos de funcionamiento

* **Modo Tunel**: Emplea el driver tun y es utilizado para crear túneles virtuales operando con el protocolo IP

* **Modo Puente**: Hace uso del dirver **tap** y es empleado para túneles que encapsulan directamente paquetes Ethernet. Se recomienda en las siguientes situaciones:
    * La VPN necesita encapsular protocolos no-IP.
    * Se ejecutan aplicaciones que necesitan network broadcasts.
    * No se cuenta con un servidor Samba y se necesita que los usuarios puedan navegar por los ficheros compartidos.


## Autenticación

La autenticación de los extremos remotos de una conexión SSL/TLS está basada en el modelo de claves asimétricas RSA. 

Los participantes intercambian sus claves públicas a través de certificados digitales X.509, que han sido firmados previamente por una Autoridad de Certificación en la que se confía.

# Tarea 1

Para esta tarea vamos a usar la receta *heat* [siguiente](https://fp.josedomingo.org/seguridadgs/u04/escenario_vpn.yaml). En esta tarea vas a realizar el segundo ejercicio del tema:

Configura una conexión VPN de acceso remoto entre dos equipos:

* Uno de los dos equipos (el que actuará como servidor) estará conectado a dos redes

* Para la autenticación de los extremos se usarán obligatoriamente certificados digitales, que se generarán utilizando openssl y se almacenarán en el directorio /etc/openvpn, junto con los parámetros Diffie-Helman y el certificado de la propia Autoridad de Certificación.

* Se utilizarán direcciones de la red 10.99.99.0/24 para las direcciones virtuales de la VPN. La dirección 10.99.99.1 se asignará al servidor VPN.

* Los ficheros de configuración del servidor y del cliente se crearán en el directorio /etc/openvpn de cada máquina, y se llamarán servidor.conf y cliente.conf respectivamente. La configuración establecida debe cumplir los siguientes aspectos:
    * El demonio openvpn se manejará con systemctl.
    * Se debe configurar para que la comunicación esté comprimida.
    * La asignación de direcciones IP será dinámica.
    * Existirá un fichero de log en el equipo.
    * Se mandarán a los clientes las rutas necesarias.
* Tras el establecimiento de la VPN, la máquina cliente debe ser capaz de acceder a una máquina que esté en la otra red a la que está conectado el servidor.

* Instala el complemento de VPN en networkmanager y configura el cliente de forma gráfica desde este complemento.


## Creación del escenario

Para ello, como indicamos en el apartado, vamos a usar una receta *heat* para nuestro **openstack** donde vamos a tener dos maquinas, una de ellas actuara como servidor VPN y la otra maquina, que es de la red interna, es a la que queremos acceder y tener la posibilidad de conectividad con la dicha maquina. Cuando tengamos el escenario ya creado nos debe de quedar algo como lo siguiente:

![topologia de escenario vpn](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/vpn/topologia-escenario-vpn.png)

Una vez ya tenemos el escenario creado solo nos queda la parte de configuración de nuestro servidor VPN, en este caso va a ser la maquina que se encuentra conectada a dos redes distintas y nosotros queremos tener conectividad con la red interna que tiene un direccionamiento **192.168.100.0/24**

## Configuración de VPN en el servidor

Lo primero que debemos de hacer es entrar en nuestro servidor de VPN y ahi debemos de instalar el paquete llamado **openvpn** y tambien el paquete **openssl** que es que contiene el software que vamos a usar para crear nuestra vpn y los certificados que vamos a necesitar:
```shell
#### Instalamos el paquete openvpn ####
debian@vpn-server:~$ sudo apt update && sudo apt install openvpn openssl -y

#### Comprobamos la version del paquete ####
debian@vpn-server:~$ sudo openvpn --version
OpenVPN 2.4.7 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Feb 20 2019
```

Una vez tengamos el paquete vamos a activar el *bit de forwarding* para que asi nuestra VPN funcione correctamente, ademas, para establecer la VPN hay que arrancar OpenVPN en ambos extremos, tenemos que configurar **openvpn** para que lea los ficheros *\*.conf* del directorio */etc/openvpn*, para ello, editamos el fichero /etc/default/openvpn y descomentamos la línea *AUTOSTART*:
```shell
#### Activamos el bit de forwarding ####
root@vpn-server:~# echo 1 > /proc/sys/net/ipv4/ip_forward

#### Descomentamos la linea AUTOSTART ####
...
AUTOSTART="all"
...
```

### Configuracion del servidor vpn

Para ello en nuestro servidor debemos de instalar el paquete de **git** ya que vamos a usar unos scripts que nos ofrece openvpn llamado **easy-rsa** por lo que realizamos las siguientes accione:
```shell
#### Instalamos git ####
debian@vpn-server:~$ sudo apt install git

#### Clonamos el repositorio de easy-rsa ####
debian@vpn-server:~$ git clone https://github.com/OpenVPN/easy-rsa
```

Nos genera un directorio llamado **easy-rsa**, entramos en dicho directorio y lo primero que debemos de hacer es declarar unas variables en el fichero llamado **vars.example** al cual debemos de cambiar el nombre a **vars**:
```shell
#### Le cambiamos el nombre ####
debian@vpn-server:~/easy-rsa/easyrsa3$ mv vars.example vars

#### Le añadimos las siguientes variables ####
...
set_var EASYRSA_REQ_COUNTRY     "ES"
set_var EASYRSA_REQ_PROVINCE    "Sevilla"
set_var EASYRSA_REQ_CITY        "Alcala"
set_var EASYRSA_REQ_ORG         "Francis Corp"
set_var EASYRSA_REQ_EMAIL       "fran@iesgn.org"
set_var EASYRSA_REQ_OU          "iesgn.org"
...
```

Una vez hecho lo anterior, debemos de usar el script de **easyrsa**, lo primero va a ser generar directorio llamado **pki** donde se almaceran todas la claves y certificados:
```shell
#### Lo ejecutamos con la opcion init-pki ####
debian@vpn-server:~/easy-rsa/easyrsa3$ ./easyrsa init-pki

Note: using Easy-RSA configuration from: /home/debian/easy-rsa/easyrsa3/vars

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /home/debian/easy-rsa/easyrsa3/pki
```

### Generar credenciales de CA

Para ello vamos a seguir trabajando con el script **easyrsa**, esta vez vamos a generar las credenciales de nuestro CA con la opción **build-ca**:
```shell
#### Ejecutamos el script con la opcion build-ca ####
debian@vpn-server:~/easy-rsa/easyrsa3$ ./easyrsa build-ca

Note: using Easy-RSA configuration from: /home/debian/easy-rsa/easyrsa3/vars
Using SSL: openssl OpenSSL 1.1.1d  10 Sep 2019

Enter New CA Key Passphrase: 
Re-Enter New CA Key Passphrase: 
Generating RSA private key, 2048 bit long modulus (2 primes)
......................................................................................+++++
..........................................+++++
e is 65537 (0x010001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:servidor-vpn

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/home/debian/easy-rsa/easyrsa3/pki/ca.crt
```

Durante este proceso nos pedirá una clave, que será la frase de paso de la clave privada de la CA. Clave que nos pedirá en el siguiente paso para generar las credenciales del servidor.

### Credenciales del servidor 

El próximo parámetro que utilizaremos con el script será build-server-full seguido del nombre que queramos que tengan los ficheros. Primero nos pedirá una frase de paso para proteger la clave privada del servidor, y luego nos pedirá la frase de paso de la CA para que pueda firmar dicha clave.
```shell
#### Usamo el parametro build-server-full con el nombre servidor ####
debian@vpn-server:~/easy-rsa/easyrsa3$ ./easyrsa build-server-full servidor

Note: using Easy-RSA configuration from: /home/debian/easy-rsa/easyrsa3/vars
Using SSL: openssl OpenSSL 1.1.1d  10 Sep 2019
Generating a RSA private key
.........+++++
............................+++++
writing new private key to '/home/debian/easy-rsa/easyrsa3/pki/easy-rsa-2261.vUP5jk/tmp.2pqACL'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
Using configuration from /home/debian/easy-rsa/easyrsa3/pki/easy-rsa-2261.vUP5jk/tmp.bw70r3
Enter pass phrase for /home/debian/easy-rsa/easyrsa3/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'servidor'
Certificate is to be certified until May 22 12:54:38 2023 GMT (825 days)

Write out database with 1 new entries
Data Base Updated
```

### Credenciales del cliente

En mi caso mi cliente se va a llamar *cliente*, y la opcion que debemos de usar con el script es **build-client-full**:
```shell
#### Generamos las credenciales del cliente ####
debian@vpn-server:~/easy-rsa/easyrsa3$ ./easyrsa build-client-full cliente

Note: using Easy-RSA configuration from: /home/debian/easy-rsa/easyrsa3/vars
Using SSL: openssl OpenSSL 1.1.1d  10 Sep 2019
Generating a RSA private key
............................................................................+++++
...+++++
writing new private key to '/home/debian/easy-rsa/easyrsa3/pki/easy-rsa-2468.oV3MDT/tmp.62prMH'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
Using configuration from /home/debian/easy-rsa/easyrsa3/pki/easy-rsa-2468.oV3MDT/tmp.xzMYdy
Enter pass phrase for /home/debian/easy-rsa/easyrsa3/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'cliente'
Certificate is to be certified until May 22 12:58:22 2023 GMT (825 days)

Write out database with 1 new entries
Data Base Updated
```

### Generar clave Diffie Hellman

Para ello, lo primero que vamos a hacer es explicar que son los parametros de **Diffie-Hellman**:

Diffie-Hellman es un protocolo criptografico, es un protocolo de establecimiento de claves entre partes que no han tenido contacto previo, utilizando un canal inseguro y de manera anónima (no autenticada).

Se emplea generalmente como medio para acordar claves simétricas que serán empleadas para el cifrado de una sesión (establecer clave de sesión). Siendo no autenticado, sin embargo, provee las bases para varios protocolos autenticados. 

El sistema se basa en la idea de que dos interlocutores pueden generar conjuntamente una clave compartida sin que un intruso, que esté escuchando las comunicaciones, pueda llegar a obtenerla. 

Para ello vamos a seguir usando el script de **easyrsa** y esta vez con la opcion **gen-dh** y debemos de tener en cuenta que es un algoritmo muy fuerte por lo que tardara un poco:
```shell
#### Generamos la clave Diffie Hellman ####
debian@vpn-server:~/easy-rsa/easyrsa3$ ./easyrsa gen-dh

Note: using Easy-RSA configuration from: /home/debian/easy-rsa/easyrsa3/vars
Using SSL: openssl OpenSSL 1.1.1d  10 Sep 2019
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
.....................................................................................................................................+............................................+.....................................................................................................................................................................+................................................................................+.....+........................................................................................................
```

### Configuracion de OpenVPN

Lo primero que debemos de hacer es llevarnos los fichero de clave y certificados al directorio **/etc/openvpn**, en mi caso los fichero que debo mover son **ca.crt, ca.key, dh.pem, server.crt y server.key**:
```shell
#### Estructura de pki ####
debian@vpn-server:~/easy-rsa/easyrsa3/pki$ tree
.
├── ca.crt
├── certs_by_serial
│   └── ...
├── dh.pem
├── index.txt
├── index.txt.attr
├── index.txt.attr.old
├── index.txt.old
├── issued
│   ├── ...
│   └── servidor.crt
├── openssl-easyrsa.cnf
├── private
│   ├── ca.key
│   ├── ...
│   └── servidor.key
├── renewed
│   └── ...
├── reqs
│   └── ...
├── revoked
│   └── ...
├── safessl-easyrsa.cnf
├── serial
└── serial.old

#### Contenido del directorio /etc/openvpn ####
debian@vpn-server:/etc/openvpn$ ls
ca.crt  ca.key  client  dh.pem  server  servidor.crt  servidor.key  update-resolv-conf
```

Ahora que ya tenemos los ficheros en el directorio que nosotros necesitamos vamos a pasar a crear el fichero de configuracion de openvpn llamado **servidor.conf**, y debe de contener el siguiente contenido:
```shell
#### Contenido del fichero /etc/openvpn/servidor.conf ####
#Dispositivo de túnel
dev tun
#Protocolo de red
proto tcp
#Direcciones IP virtuales
server 10.99.99.0 255.255.255.0 
#subred local
push "route 192.168.100.0 255.255.255.0"
#Rol de servidor
tls-server
#Parámetros Diffie-Hellman
dh /etc/openvpn/dh.pem
#Certificado de la CA
ca /etc/openvpn/ca.crt
#Certificado local
cert /etc/openvpn/servidor.crt
#Clave privada local
key /etc/openvpn/servidor.key
#Activar la compresión LZO
comp-lzo
#Detectar caídas de la conexión
keepalive 10 60
#Nivel de información
verb 3
# Contraseña del certificado del servidor
askpass contraseña.txt

#### Reiniciamos el servicio y vemos el nuevo dispositivo tun0 ####
debian@vpn-server:~$ sudo systemctl restart openvpn

debian@vpn-server:~$ ip a show tun0
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
    link/none 
    inet 10.99.99.1 peer 10.99.99.2/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::96a4:ea7d:b8f:d7d/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
```

## Configuración del cliente VPN

Debemos de seguir los pasos que hemos seguido con el servidor, pero solo la instalacion de **openvpn** ya que las claves y certificados ya los hemos generado en el servidor, una vez instalado y configurado como en el servidor, podemos seguir

Para ello debemos de pasar desde el servido vpn a nuestro cliente las credenciales que habiamos generado con el nombre **cliente**, por lo que mediante la utilidad de **scp** debemos de pasarlo al cliente y tambien debemos de pasar el certificado de nuestra CA:
```shell
debian@vpn-server:~/easy-rsa/easyrsa3/pki$ scp ca.crt debian@10.0.0.17:/home/debian/
debian@vpn-server:~/easy-rsa/easyrsa3/pki$ scp issued/cliente.crt debian@10.0.0.17:/home/debian/
debian@vpn-server:~/easy-rsa/easyrsa3/pki$ scp private/cliente.key debian@10.0.0.17:/home/debian/
```

Una vez hecho vamos a moverlos a **/etc/openvpn/** y generamos el fichero **cliente.conf**:
```shell
#### Contenido del fichero /etc/openvpn/cliente.conf ####
#Dispositivo de túnel
dev tun

#IP del servidor
remote 10.0.0.15 

#Direcciones IP virtuales
ifconfig 10.99.99.0 255.255.255.0
pull

# Protocolo tcp
proto tcp-client 

#Rol de cliente
tls-client

#Certificado de la CA
ca /etc/openvpn/ca.crt

#Certificado cliente
cert /etc/openvpn/cliente.crt

#Clave privada cliente
key /etc/openvpn/cliente.key

#Activar la compresión LZO
comp-lzo

#Detectar caídas de la conexión
keepalive 10 60

#Nivel de la información
verb 3

# Contraseña del certificado del cliente
askpass contraseña.txt
```

# VPN Site-to-Site

A diferencia del escenario anterior, una VPN sitio a sitio se basa en un concepto algo distinto. Mientras que en el primer caso, hemos conectado una máquina a una red remota, en este caso vamos a conectar dos redes remotas, por lo que técnicamente, estaremos fusionando las dos redes a nivel de aplicación. En este nuevo escenario, tendremos dos máquinas en redes locales diferentes; por un lado la 192.168.100.0/24 y por otro lado la 192.168.200.0/24. A su vez, cada una de estas máquinas estará compartiendo la red con un servidor que serán los que gestionen la conexión de la VPN, que tendrán una red común, la 172.22.0.0/16. 

Para ello, en nuestra maquina servidor vamos a tener el siguiente fichero:
```shell
#### Fichero de configuracion en la maquina servidor para site-to-site ####
#Dispositivo de túnel
dev tun

#Encaminamiento
ifconfig 10.99.99.1 10.99.99.2
route 192.168.200.0 255.255.255.0

#Rol de servidor
tls-server
#Parámetros Diffie-Hellman
dh /etc/openvpn/dh.pem
#Certificado de la CA
ca /etc/openvpn/ca.crt
#Certificado local
cert /etc/openvpn/servidor.crt
#Clave privada local
key /etc/openvpn/servidor.key
#Activar la compresión LZO
comp-lzo
#Detectar caídas de la conexión
keepalive 10 60
#Nivel de información
verb 3
# Contraseña del certificado del servidor
askpass contraseña.txt
```

Como vemos, respecto al anterior fichero de configuración, las lineas que han cambiado han sido las siguientes:
```shell
# CLIENTE-SERVIDOR
#Direcciones IP virtuales
server 10.99.99.0 255.255.255.0 

#subred local
push "route 192.168.100.0 255.255.255.0"

# SITE-TO-SITE
# Encaminamiento
ifconfig 10.99.99.1 10.99.99.2
route 192.168.200.0 255.255.255.0
```

Y ahora en la parte de nuestro cliente creamos un nuevo fichero en el cual vamos a tener el siguiente contenido:
```shell
#### Fichero de sit-to-site vpn ####
#Dispositivo de túnel
dev tun

#IP del servidor
remote 10.0.0.15 

#Encaminamiento
ifconfig 10.99.99.2 10.99.99.1

# Subred remota
route 192.168.100.0 255.255.255.0

#Rol de cliente
tls-client

#Certificado de la CA
ca /etc/openvpn/ca.crt

#Certificado cliente
cert /etc/openvpn/cliente.crt

#Clave privada cliente
key /etc/openvpn/cliente.key

#Activar la compresión LZO
comp-lzo

#Detectar caídas de la conexión
keepalive 10 60

#Nivel de la información
verb 3

# Contraseña del certificado del cliente
askpass contraseña.txt
```

Una vez definidos, modificamos el fichero /etc/default/openvpn si hemos creado nuevos ficheros .conf para definir esta nueva conexión, y posteriormente reiniciamos los servicios. Primero el servicio del servidor, y luego el del cliente. Debemos recordar activar el bit de forward en ambas máquinas, para que puedan “pasar” el tráfico sus respectivas redes locales.

![site-to-site](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/vpn/site-to-site-vpn.png)

Asi, si hacemos ping a las redes internas veremos que tenemos ping:
```shell
#### Desde el cliente al servidor ####
ebian@cliente-vpn:/etc/openvpn$ ping 192.168.100.2
PING 192.168.100.2 (192.168.100.2) 56(84) bytes of data.
64 bytes from 192.168.100.2: icmp_seq=1 ttl=64 time=0.868 ms
64 bytes from 192.168.100.2: icmp_seq=2 ttl=64 time=1.25 ms
64 bytes from 192.168.100.2: icmp_seq=3 ttl=64 time=1.24 ms
^C
--- 192.168.100.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 5ms
rtt min/avg/max/mdev = 0.868/1.117/1.249/0.180 ms

#### Desde el servidor al cliente ####
debian@vpn-server:/etc/openvpn$ ping 192.168.200.11
PING 192.168.200.11 (192.168.200.11) 56(84) bytes of data.
64 bytes from 192.168.200.11: icmp_seq=1 ttl=64 time=0.860 ms
64 bytes from 192.168.200.11: icmp_seq=2 ttl=64 time=1.16 ms
64 bytes from 192.168.200.11: icmp_seq=3 ttl=64 time=1.09 ms
^C
--- 192.168.200.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 5ms
rtt min/avg/max/mdev = 0.860/1.036/1.160/0.127 ms
```