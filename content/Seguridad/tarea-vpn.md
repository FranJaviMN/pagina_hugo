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

## Configuración de VPN

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

### Generar la la autoridad certificadora

El siguiente paso es configurar una autoridad certificadora (CA). Esta es la responsable de emitir y revocar certificados digitales y servirá para la confianza entre el equipo o equipos clientes y el servidor, no siendo posible la conexión mediante VPN si se carece de certificado válido.

OpenVPN permite la autentificación bidireccional mediante certificados, de manera que el cliente debe autenticar el certificado del servidor y éste debe autenticar el certificado del cliente.
