---
title: "Cortafuegos"
date: 2021-02-03T19:30:58+01:00
draft: true
---

# Cortafuegos

Vamos a construir un cortafuegos en dulcinea que nos permita controlar el tráfico de nuestra red. El cortafuegos que vamos a construir debe funcionar tras un reinicio.

## Política por defecto

La política por defecto que vamos a configurar en nuestro cortafuegos será de tipo DROP. 

Antes de activar dicha politica debemos de comprobar que nuestras reglas funcionan, si no nos quedaremos sin que funcionen algunos servicios o el ssh.

# NAT

1. Configura de manera adecuada las reglas NAT para que todas las máquinas de nuestra red tenga acceso al exterior.

Para ello vamos a usar una regla del tipo **POSTROUTING** ya que el trafico de las redes internas debe de pasar por la maquina **Dulcinea** y salir al exterior. Como tenemos dos redes distintas que estan conectadas a interfaces de red en Dulcinea distintas, debemos de añadir dos reglas, una por cada red interna que tengamos:
```shell
#### Regla para la red interna 10.0.1.0/24 ####
debian@dulcinea:~$ sudo iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o eth0 -j SNAT --to 10.0.0.13

#### Regla para la red DMZ 10.0.2.0/24 ####
debian@dulcinea:~$ sudo iptables -t nat -A POSTROUTING -s 10.0.2.0/24 -o eth0 -j SNAT --to 10.0.0.13
```

Asi, si nos vamos a las maquinas de las redes internas e intenamos hacer un ping a google, veremos que nos deja sin ningun tipo de problema:
```shell
#### Prueba para sancho ####
ubuntu@sancho:~$ ping www.google.es
PING www.google.es (172.217.17.3) 56(84) bytes of data.
64 bytes from mad07s09-in-f3.1e100.net (172.217.17.3): icmp_seq=1 ttl=112 time=402 ms
64 bytes from mad07s09-in-f3.1e100.net (172.217.17.3): icmp_seq=2 ttl=112 time=172 ms
--- www.google.es ping statistics ---
3 packets transmitted, 2 received, 33.3333% packet loss, time 5800ms
rtt min/avg/max/mdev = 172.431/287.242/402.054/114.811 ms

#### Prueba para Freston ####
debian@freston:~$ ping www.google.es
PING www.google.es (172.217.17.3) 56(84) bytes of data.
64 bytes from mad07s09-in-f3.1e100.net (172.217.17.3): icmp_seq=1 ttl=112 time=263 ms
64 bytes from mad07s09-in-f3.1e100.net (172.217.17.3): icmp_seq=2 ttl=112 time=389 ms
64 bytes from mad07s09-in-f3.1e100.net (172.217.17.3): icmp_seq=3 ttl=112 time=162 ms
64 bytes from mad07s09-in-f3.1e100.net (172.217.17.3): icmp_seq=4 ttl=112 time=194 ms
--- www.google.es ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 8ms
rtt min/avg/max/mdev = 162.013/251.945/389.442/87.342 ms

#### Prueba para Quijote ####
[centos@quijote ~]$ ping www.google.es
PING www.google.es (172.217.17.3) 56(84) bytes of data.
64 bytes from 172.217.17.3 (172.217.17.3): icmp_seq=1 ttl=112 time=406 ms
64 bytes from 172.217.17.3 (172.217.17.3): icmp_seq=2 ttl=112 time=190 ms
64 bytes from 172.217.17.3 (172.217.17.3): icmp_seq=3 ttl=112 time=241 ms
64 bytes from 172.217.17.3 (172.217.17.3): icmp_seq=4 ttl=112 time=439 ms
--- www.google.es ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 459ms
rtt min/avg/max/mdev = 190.320/318.934/438.598/105.590 ms
```

2. Configura de manera adecuada todas las reglas NAT necesarias para que los servicios expuestos al exterior sean accesibles.

Para ello, los servicios que tenemos expuestos al exterior son 2, nuestro DNS que se encuentra en la maquina Freston y nuestro sitio web, que se encuentra en la maquina Quijote por lo que tenemos que añadir de nuevo reglas del tipo **PREROUTING**.

## Maquina Freston

Para nuestra maquina freston, que es la que contiene el servidor DNS, vamos a añadirle 2 reglas, una que sera el LOG de la regla para ver que se cumple con exito y la otra para hacer que las maquinas puedan usar el dns de freston:
```shell
#### Creamos la regla del LOG ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -p udp --dport 53 -i eth0 -j LOG --log-prefix 'Regla DNS correcta ' --log-level 4

#### Le añadimos la regla de DNS ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -p udp --dport 53 -i eth0 -j DNAT --to 10.0.1.11:53
```
Si hacemos una consulta desde el exterior, es decir, por nuestra pagina web, veremos que en nuestro log de **/var/log/kern.log** veremos que tendremos una linea donde tendra un mensaje de exito que habiamos puesto:
```shell
#### Mensaje del log del kernel ####
Jan 13 13:54:20 dulcinea kernel: [2515708.564525] Regla DNS correcta IN=eth0 OUT= MAC=fa:16:3e:88:a0:2b:fa:16:3e:c5:ac:f0:08:00 SRC=172.22.4.54 DST=10.0.0.13 LEN=68 TOS=0x00 PREC=0x00 TTL=63 ID=59300 PROTO=UDP SPT=41332 DPT=53 LEN=48
```

## Maquina Quijote

En nuestra maquina quijote tenemos el servidor web, que esta escuchando en el puerto 80 y en el puerto 443, por lo que tendremos que añadir 2 reglas, una por cada puerto en el que escucha nuestra maquina por lo que solo debemos de añadir una regla de log porque tenemos una redireccion hacia https que es el puerto 443:
```shell
#### Regla sobre el LOG ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -i eth0 -j LOG --log-prefix 'HTTPS EXITO ' --log-level 4

#### Regla sobre el puerto 80 ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -i eth0 -j DNAT --to 10.0.2.3:80

#### Regla sobre el puerto 443 ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -i eth0 -j DNAT --to 10.0.2.3:443
```

Asi, si desde nuestro nevegador entramos en la pagina www.franjavier.gonzalonazareno.org o a nuestra ip flotante 172.22.200.155 veremos que en el log del kernel nos aparecera el siguiente mensaje:
```shell
#### Mensaje de exito ####
Jan 13 14:05:28 dulcinea kernel: [2516376.594103] HTTPS EXITO IN=eth0 OUT= MAC=fa:16:3e:88:a0:2b:fa:16:3e:c5:ac:f0:08:00 SRC=172.22.4.54 DST=10.0.0.13 LEN=60 TOS=0x00 PREC=0x00 TTL=63 ID=58187 DF PROTO=TCP SPT=33012 DPT=443 WINDOW=64240 RES=0x00 SYN URGP=0
```

# SSH

1. Podemos acceder por ssh a todas las máquinas.

En estas reglas, en total, seria 4 reglas que tenemos que añadir las cuales nos permitirian hacer ssh entre la red interna con la red DMZ y dulcinea con la red interna y la red DMZ

## SSH Desde Dulcinea

Lo primero que vamos a hacer es crear las dos reglas que necesitamos para poder conectarnos por ssh desde Dulcinea a las demas maquinas de la red interna y la red DMZ por lo que debemos de crear reglas **INPUT** y reglas **OUTPUT** para poder hacerlo. Seguimos el mismo metodo que antes, ponemos primero la regla con el LOG para comprobar que funciona correctamente y luego la regla en si:
```shell
#### Regla LOG para la red interna 10.0.1.0/24 ####
debian@dulcinea:~$ sudo iptables -A INPUT -i eth1 -s 10.0.1.0/24 -p tcp --sport 22 -j LOG --log-prefix 'SSH RED INTERNA EXITO ' --log-level 4

#### Par de reglas para ssh con la red interna 10.0.1.0/24 ####
debian@dulcinea:~$ sudo iptables -A INPUT -i eth1 -s 10.0.1.0/24 -p tcp --sport 22 -j ACCEPT

debian@dulcinea:~$ sudo iptables -A OUTPUT -o eth1 -d 10.0.1.0/24 -p tcp --dport 22 -j ACCEPT
```

Con estas reglas ya tenemos listo el acceso por ssh desde dulcinea a la red interna 10.0.1.0/24 y ademas nos guarda un log en el que nos dice que hemos tenido exito en la conexión:
```shell
#### Log con exito ####
Jan 13 14:37:17 dulcinea kernel: [2518284.976984] SSH RED INTERNA EXITO IN=eth1 OUT= MAC=fa:16:3e:d7:a2:7a:fa:16:3e:1a:e5:bb:08:00 SRC=10.0.1.11 DST=10.0.1.9 LEN=52 TOS=0x10 PREC=0x00 TTL=64 ID=51083 DF PROTO=TCP SPT=22 DPT=57006 WINDOW=1002 RES=0x00 ACK FIN URGP=0
```

Una vez hecho esto hacemos lo mismo pero esta vez con la red DMZ 10.0.2.0/24:
```shell
#### Regla LOG para la red DMZ 10.0.2.0/24 ####
debian@dulcinea:~$ sudo iptables -A INPUT -i eth2 -s 10.0.2.0/24 -p tcp --sport 22 -j LOG --log-prefix 'SSH RED DMZ EXITO ' --log-level 4

#### Par de reglas para ssh con la red DMZ 10.0.2.0/24 ####
debian@dulcinea:~$ sudo iptables -A INPUT -i eth2 -s 10.0.2.0/24 -p tcp --sport 22 -j ACCEPT

debian@dulcinea:~$ sudo iptables -A OUTPUT -o eth2 -d 10.0.2.0/24 -p tcp --dport 22 -j ACCEPT
```

Con estas reglas ya tendriamos conexion hacia la red DMZ:
```shell
#### Log con exito ####
Jan 13 18:23:29 dulcinea kernel: [2531858.117376] SSH RED DMZ EXITO IN=eth2 OUT= MAC=fa:16:3e:7a:bf:43:fa:16:3e:dc:d9:55:08:00 SRC=10.0.2.3 DST=10.0.2.11 LEN=52 TOS=0x08 PREC=0x40 TTL=64 ID=51084 DF PROTO=TCP SPT=22 DPT=42602 WINDOW=826 RES=0x00 ACK FIN URGP=0
```

## SSH desde la red interna

Para ello debemos de tener en cuenta que debemos de crear una regla de iptables en dulcinea del tipo **PREROUTING** la cual hace que el trafico pase a traves de la maquina dulcinea hasta la red DMZ, para ello vamos a usar las siguientes reglas:
```shell
#### Regla LOG para el trafico de la red interna a la red DMZ ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -i eth1 -s 10.0.1.0/24 -p tcp --dport 22 -j LOG --log-prefix 'DE RED INTERNA A DMZ EXITO ' --log-level 4

#### Regla para ssh de red interna con DMZ ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -i eth1 -s 10.0.1.0/24 -p tcp --dport 22 -j DNAT --to 10.0.2.3:22

#### Comprobacion de log ####
Jan 13 18:41:23 dulcinea kernel: [2532932.166102] DE RED INTERNA A DMZ EXITO IN=eth1 OUT= MAC=fa:16:3e:d7:a2:7a:fa:16:3e:1a:e5:bb:08:00 SRC=10.0.1.11 DST=10.0.2.3 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=35702 DF PROTO=TCP SPT=57116 DPT=22 WINDOW=64240 RES=0x00 SYN URGP=0 
```

## SHH desde la red DMZ

Para ello hacemos como en el apartado anterior pero esta vez cambiando las redes y las interfaces:
```shell
#### Regla LOG para el trafico de la red DMZ a la red interna ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -i eth2 -s 10.0.2.0/24 -p tcp --dport 22 -j LOG --log-prefix 'DE RED DMZ A RED INTERNA EXITO ' --log-level 4

#### Regla para ssh desde DMZ a red interna ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -i eth2 -s 10.0.2.0/24 -p tcp --dport 22 -j DNAT --to 10.0.1.8:22

debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -i eth2 -s 10.0.2.0/24 -p tcp --dport 22 -j DNAT --to 10.0.1.11:22

#### Comprobacion del log ####
Jan 13 18:51:38 dulcinea kernel: [2533547.522213] DE REd DMZ A RED INTERNA EXITIN=eth2 OUT= MAC=fa:16:3e:7a:bf:43:fa:16:3e:dc:d9:55:08:00 SRC=10.0.2.3 DST=10.0.1.8 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=24877 DF PROTO=TCP SPT=55076 DPT=22 WINDOW=26730 RES=0x00 SYN URGP=0 
```

2. Podemos hacer ssh al exterior desde todas las maquinas

Para ello vamos a permitir el trafico por el puerto 22 de nuestra maquina Dulcinea, para ello, primero lo haremos con la maquina dulcinea y luego con las maquinas de las dos redes internas:

## Maquina Dulcinea

Para ello vamos a usar el siguiente comando:
```shell
#### Log de ssh ####
debian@dulcinea:~$ sudo iptables -A INPUT -i eth0 -p tcp --sport 22 -j LOG --log-prefix 'DE DULCINEA A EXTERIOR ' --log-level 4

#### Reglas de entrada y salida para Dulcinea ####
debian@dulcinea:~$ sudo iptables -A INPUT -i eth0 -p tcp --sport 22 -j ACCEPT

debian@dulcinea:~$ sudo iptables -A OUTPUT -o eth0 -p tcp --dport 22 -j ACCEPT

#### Resultado del log ####
Jan 28 17:18:09 dulcinea kernel: [3824005.988438] DE DULCINEA A EXTERIOR IN=eth0 OUT= MAC=fa:16:3e:88:a0:2b:fa:16:3e:c5:ac:f0:08:00 SRC=172.23.0.30 DST=10.0.0.13 LEN=52 TOS=0x10 PREC=0x00 TTL=61 ID=52062 DF PROTO=TCP SPT=22 DPT=42510 WINDOW=501 RES=0x00 ACK URGP=0
```

# Bases de datos

Para ello debemos de saber que el puerto que usa nuestra base de datos para conectarse es el puerto **3306** ya que estamos usando una base de datos mariadb. Lo primero es tener en el fichero **/etc/mysql/mariadb.conf.d/50-server.cnf** la linea que dice *port = 3306* descomentada, la linea *skip-external-locking* comentada y la linea **bind-address  = 0.0.0.0*.

Cuando tengamos todo listo debemos de añadir una regla que permita la conexion desde la red 10.0.2.0/24 a la ip de la maquina 10.0.1.8 por el puerto 3306
```shell
#### Regla de log ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -i eth2 -s 10.0.2.0/24 -p tcp --dport 3306 -j LOG --log-prefix 'CONEXION BD EXITO ' --log-level 4

#### Regla para lo conexion desde la red DMZ ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -s 10.0.2.3 -i eth2 -p tcp --dport 3306 -j DNAT --to 10.0.1.8:3306
```


# Web

1. Las páginas web de quijote (80, 443) pueden ser accedidas desde todas las máquinas de nuestra red y desde el exterior.

Para ello vamos a usar dos reglas, una por cada puerto que deben de ser redirigidos hacia los puerto 80 y 443 de la maquina de Quijote, que es donde tenemos nuestro servidor web funcionando, por lo que vamos a realizar las siguientes reglas:
```shell
#### Regla para el puerto 80 ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -i eth0 -j DNAT --to 10.0.2.3:80

#### Reglas para control de log del puerto 443 ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -i eth0 -j LOG --log-prefix 'HTTPS EXITO ' --log-level 4

#### Regla para el puerto 443 ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -i eth0 -j DNAT --to 10.0.2.3:443
```

# Mas servicios

## Correos

Para ello vamos a usar el servidor de correos llamado postfix el cual usa el puerto 25/tcp, como este servidor de correos lo vamos a tener corriendo en la maquina freston debemos de inidicarle, al igual que con los servicios web, la ip interna de la maquina junto al puerto por el que escucha por lo que usamos la siguiente regla de iptables:
```shell
#### Regla de iptable para correos ####
debian@dulcinea:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 25 -i eth0 -j DNAT --to 10.0.1.11:25
```

Donde vemos que, todo el paquete que llegue a la maquina dulcina con el puerto 25/tcp y por la interfaz eth0, sea redirigido hacia la ip de freston junto a su puerto.
