---
title: "Vpn Openstack"
date: 2020-10-30T13:21:46+01:00
draft: true
---

# Configuración de cliente OpenVPN con certificados X.509

En este caso vamos a configurar una VPN con el cliente **OpenVPN** para poder conectarnos a la maquina virtual **sputnik.gonzalonazareno.org**, los pasos a seguir son los siguientes:

1. Disponer de un certificado x509 para nuestro equipo firmado por la CA Gonzalo Nazareno
2. Configurar OpenVPN para que se conecte a sputnik.

## Creacion de clave privada y solicitud de firma.

Lo primero que vamos a crear va a ser una clave privada con la herramienta **openssl**, en este caso una clave privada RSA de 4096 bits. Tenemos que tener en cuenta que esta acciones las vamos a realizar como root. Despues de crear nuestra clave privada vamos a generar un fichero .csr el cual debe de ser firmado por la autoridad de nuestro instituto Gonzalo Nazareno.

1. Creamos nuestra clave privada desde el usuario **root**:
```shell
root@debian10:~# openssl genrsa 4096 > /etc/ssl/private/nombredemimaquina.key
Generating RSA private key, 4096 bit long modulus (2 primes)
..................................................................................................................................................................................++++
..............................................................++++
e is 65537 (0x010001)
```

2. Generamos el fichero csr que debe de ser firmado por CA del Gonzalo Nazareno:
```shell
root@debian10:~# openssl req -new -key /etc/ssl/private/nombredemimaquina.key -out /root/nombredemimaquina.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:Sevilla
Locality Name (eg, city) []:Dos Hermanas
Organization Name (eg, company) [Internet Widgits Pty Ltd]:IES Gonzalo Nazareno
Organizational Unit Name (eg, section) []:Informatica
Common Name (e.g. server FQDN or YOUR name) []:nombremimaquina-francisco.martin
```

De esta forma ya tendremos la clave privada y el fichero csr, dicho fichero csr debe de ser firmado. Para poder entragar nuestro fichero csr debemos de enviarlo por este [enlace](https://dit.gonzalonazareno.org/gestiona/) en el apartado de de Utilidades -> Certifcados.

## Configuración de OpenVPN

Una vez hayamos obtenido el certificado debemos de configurar nuestro OpenVPN, para ello debemos de dirigirnos(como root) al directorio **/etc/openvpn** y en dicho directorio generamos un fichero de extensión **.conf** en el cual debemos de tener las siguientes lineas:
```shell
dev tun
remote sputnik.gonzalonazareno.org
ifconfig 172.23.0.0 255.255.255.0
pull
proto tcp-client
tls-client
remote-cert-tls server
ca /etc/ssl/certs/gonzalonazareno.pem <- La ruta al certificado de la CA Gonzalo Nazareno
cert /etc/openvpn/nombremimaquina-francisco.martin.crt <- La ruta al certificado CRT firmado que nos han devuelto
key /etc/ssl/private/nombredemimaquina.key <- La ruta a la clave privada con permisos 600
comp-lzo
keepalive 10 60
log /var/log/openvpn-sputnik.log
verb 1
```
Donde podemos ver lo siguiente:
* gonzalonazareno.pem: Este certificado lo podemos encontrar [en este enlace](https://dit.gonzalonazareno.org/gestiona/info/documentacion/ca).
* nombremimaquina.francisco.martin.crt: Certificado firmado por la CA Gonzalo Nazareno.
* nombredemimaquina.key: Es la clave privada que habiamos generado anteriormente.

Una vez creado el fichero vamos a reiniciar el servicio de OpenVPN y comprobamos que tenemos las rutas de enlace y veremos una linea como la siguiente:
```shell
root@debian10:~# ip r
...
172.22.0.0/16 via 172.23.0.37 dev tun0 
...
```

Y si vemos el documento de log de OpenVPN en **/var/log/openvpn.log**:
```shell
root@debian10:~# cat /var/log/openvpn-sputnik.log 
Fri Oct 30 13:10:22 2020 OpenVPN 2.4.7 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Feb 20 2019
Fri Oct 30 13:10:22 2020 library versions: OpenSSL 1.1.1d  10 Sep 2019, LZO 2.10
Fri Oct 30 13:10:22 2020 WARNING: using --pull/--client and --ifconfig together is probably not what you want
Fri Oct 30 13:10:22 2020 TCP/UDP: Preserving recently used remote address: [AF_INET]92.222.86.77:1194
Fri Oct 30 13:10:22 2020 Attempting to establish TCP connection with [AF_INET]92.222.86.77:1194 [nonblock]
Fri Oct 30 13:10:23 2020 TCP connection established with [AF_INET]92.222.86.77:1194
Fri Oct 30 13:10:23 2020 TCP_CLIENT link local: (not bound)
Fri Oct 30 13:10:23 2020 TCP_CLIENT link remote: [AF_INET]92.222.86.77:1194
Fri Oct 30 13:10:24 2020 [sputnik.gonzalonazareno.org] Peer Connection Initiated with [AF_INET]92.222.86.77:1194
Fri Oct 30 13:10:26 2020 TUN/TAP device tun0 opened
Fri Oct 30 13:10:26 2020 /sbin/ip link set dev tun0 up mtu 1500
Fri Oct 30 13:10:26 2020 /sbin/ip addr add dev tun0 local 172.23.0.38 peer 172.23.0.37
Fri Oct 30 13:10:26 2020 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
Fri Oct 30 13:10:26 2020 Initialization Sequence Completed
```

**IMPORTANTE:** Si queremos que este servicio solo se active manualmente debemos de usar el siguiente comando con el que dehabilitalemos que inicie al encender el ordenador:
```shell
root@debian10:~# systemctl disable openvpn.service
Synchronizing state of openvpn.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install disable openvpn
```

Tambien es recomendable actualizar nuestro fichero **/etc/hosts** y añadir la resolucion estatica de las siguientes ip:
```shell
...
172.22.222.1    jupiter
172.22.0.1       macaco
...
```

Comprobamos que podemos hacer ping a **172.22.0.1**:
```shell
root@debian10:~# ping 172.22.0.1
PING 172.22.0.1 (172.22.0.1) 56(84) bytes of data.
64 bytes from 172.22.0.1: icmp_seq=1 ttl=63 time=157 ms
64 bytes from 172.22.0.1: icmp_seq=2 ttl=63 time=107 ms
64 bytes from 172.22.0.1: icmp_seq=3 ttl=63 time=104 ms
64 bytes from 172.22.0.1: icmp_seq=4 ttl=63 time=108 ms
64 bytes from 172.22.0.1: icmp_seq=5 ttl=63 time=96.10 ms
```