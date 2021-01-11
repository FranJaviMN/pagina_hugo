---
title: "Https Quijote"
date: 2021-01-11T18:00:52+01:00
draft: true
---

# Solicitar certificado wildcard con openssl

Para esta practica vamos a solicitar un certificado wildcard para nuestra dominio llamado *franjavier.gonzalonazareno.org en el que vamos a tener nuestro escenario de openstack.

## Creacion de clave privada y certificado

Para poder realizar esta practica vamos a irnos a nuestra maquina llamada **freston** que es donde vamos a usar nuestro certificado wildcard principalmente, en esta maquina vamos a crear una clave privada y luego generamos el certificado indicandole el directorio donde esta la clave:
```shell
#### Generamos la clave privada ####
root@freston:~# openssl genrsa 4096 > /etc/ssl/private/wildcard.key
Generating RSA private key, 4096 bit long modulus (2 primes)
..............++++
...............................................................++++
e is 65537 (0x010001)

#### Generamos el certificado wildcard ####
root@freston:~# openssl req -new -key /etc/ssl/private/wildcard.key -out /root/wildcard-franjavier.csr
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
Common Name (e.g. server FQDN or YOUR name) []:*.franjavier.gonzalonazareno.org
```

Una vez hecho debemos enviarlo a la autoridad certificadora. Una vez lo tengamos firmado debemos de tener un fichero .crt y otro fichero que es el de la autoridad certificadora, es decir, gonzalonazareno.crt.

Los fichero debemos de tenerlos en los siguientes directorios:
* wildcard-franjavier.crt -> /etc/ssl/certs/
* gonzalonazareno.crt -> /etc/ssl/certs/
* wildcard.key -> /etc/ssl/private/

De esta forma ya tendremos nuestros certificaodos en nuestra maquina y podremos empezar a usar el certificado wildcard.

# https en quijote

Ahora lo que vamos a hacer es usar la clave privada y el certificado firmado **.crt** y nos lo vamos a llevar a la maquina de quijote, una vez lo tengamos alli vamos a mover la clave privada llamada **wildcard.key** al directorio **/etc/pki/tls/private/** y el certificado firmado lo movemos a **/etc/pki/tls/certs/**, de tal forma que debemos de tenerlo de las siguiente forma:
```shell
#### Clave privada ####
[centos@quijote private]$ ls -la
total 12
drwxr-xr-x. 2 root root   66 Dec 17 23:45 .
drwxr-xr-x. 5 root root  104 Dec 17 23:46 ..
-rw-------. 1 root root 1704 Jan  8 09:14 localhost.key
-rw-------. 1 root root 3243 Nov 19 12:52 postfix.key
-rw-------. 1 root root 3243 Jan  8 09:02 wildcard.key

#### Certificados ####
[centos@quijote certs]$ ls -la
total 20
drwxr-xr-x. 2 root root   125 Dec 17 23:45 .
drwxr-xr-x. 5 root root   104 Dec 17 23:46 ..
lrwxrwxrwx. 1 root root    49 Nov 19 12:49 ca-bundle.crt -> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
lrwxrwxrwx. 1 root root    55 Nov 19 12:49 ca-bundle.trust.crt -> /etc/pki/ca-trust/extracted/openssl/ca-bundle.trust.crt
-rw-r--r--. 1 root root  3766 Jan  8 09:14 localhost.crt
-rw-r--r--. 1 root root  2216 Nov 19 12:52 postfix.pem
-rw-r--r--. 1 root root 10015 Jan  8 08:59 wildcard-franjavier.crt
```

Una vez hecho esto debemos de instalar el mod de https para apache, el paquete en cuestion se llama **mod_ssl**:
```shell
#### Instalamos mod_ssl ####
[centos@quijote ~]$ sudo dnf install mod_ssl

#### Recargamos el servicio de httpd ####
[centos@quijote ~]$ sudo systemctl reload httpd
```

Una vez hecho esto debemos de dirigirnos a nuestra carpeta de **sites-availables** y ahi debemos de crear un nuevo fichero llamado, en mi caso, **gonzalonazareno-https.conf** que tiene el siguiente contenido:
```shell
<VirtualHost *:443>
    ServerName www.franjavier.gonzalonazareno.org
    DocumentRoot /var/www/gonzalonazareno/html
    ErrorLog /var/www/gonzalonazareno/log/error.log
    CustomLog /var/www/gonzalonazareno/log/requests.log combined

    <Proxy "unix:/run/php-fpm/www.sock|fcgi://php-fpm">
    ProxySet disablereuse=off
    </Proxy>
    <FilesMatch \.php$>
           SetHandler proxy:fcgi://php-fpm
    </FilesMatch>

    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/wildcard-franjavier.crt
    SSLCertificateKeyFile /etc/pki/tls/private/wildcard.key

</VirtualHost>
```

En el que vemos como indicamos donde se encuentran los fichero necesarios para poder usar https. Cuando tengamos este paso listo nos dirigimos a nuestro antiguo fichero de http llamado **gonzalonazareno.conf** le vamos a añadir la siguiente linea:
```shell
#### Redireccion hacia https ####
...
Redirect permanent / https://www.franjavier.gonzalonazareno.org
...
```

Asi ya tendremos listo nuestro sistema para que siempre nos muestra la pagina con https, ahora solo queda deshabilitar SElinux para que no nos de problemas a la hora de usar ssl en nuestro apache, para ello nos dirigimos al fichero **/etc/selinux/config**:
```shell
#### Deshabilitamos SElinux ####

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are pr$
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

#### Reiniciamos nuestra maquina ####
[centos@quijote ~]$ sudo reboot
```

Cuando lo tengamos solo nos queda recargar el servicio de httpd y ya tendriamos listo nuestro sitio web con https activado, debemos de tener en cuenta que, a la hora de entrar en la pagina, si no tenemos el certificado de la autoridad certificadora de gonzalonazareno en nuestro navegador nos saldra un pagina en la que nos indica que el sitio no es seguro, asi que debemos de instalarlo para poder evitar este problema, una vez hecho solo entramos y vemos is ha funcionado:

![pagina con https](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/https-quijote/https-quijote.png)