---
title: "Ldaps"
date: 2020-12-21T13:18:45+01:00
draft: true
---

# LDAPS

Para poder realizar este ejercicio necesitamos un certificado wildcard en nuestra maquina donde tenemos el servidor de ldap. Para ello debemos de solicitar el certificado a nuestra autoridad certificador bajo el dominio que nosotros necesitemos, en mi caso será **.franjavier.gonzalonazareno.org**

Una vez lo tengamos creado ya podremos seguir con el ejercicio sobre ldaps. Lo primero que debemos de hacer es crear unas acl para que nuestro usuario de **openldap** pueda acceder a los certificados y claves, para ello vamos a usar el paquete llamado **acl** y luego vamos a crear un total de 4 acl:
```shell
#### Instalamos acl ####
debian@freston:~$ sudo apt install acl

#### Creamos las 4 acls ####
debian@freston:~$ sudo setfacl -m u:openldap:r-x /etc/ssl/certs/
debian@freston:~$ sudo setfacl -m u:openldap:r-x /etc/ssl/certs/gonzalonazareno.crt 
debian@freston:~$ sudo setfacl -m u:openldap:r-x /etc/ssl/certs/wildcard-franjavier.crt
debian@freston:~$ sudo setfacl -m u:openldap:r-x /etc/ssl/private/
debian@freston:~$ sudo setfacl -m u:openldap:r-x /etc/ssl/private/wildcard.key
```

De esta forma nuestro usuario de openldap va a tener el acceso necesario a los ficheros y directorios que necesita para poder realizar la configuracion sobre ldaps.

Ahora debemos de crearnos un nuevo fichero .ldif en el que vamos a ir indicando cual es cada uno de los ficheros que tenemos en los dirertorios que indicamos en el ejercicio sobre el certificado de wildcard, por ello el fichero queda de la siguiente forma:
```shell
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ssl/certs/gonzalonazareno.crt
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ssl/certs/wildcard-franjavier.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ssl/private/wildcard.key
```
**ATENCION: DBEEMOS ASEGURARNOS BIEN QUE TENEMOS LOS PERMISOS NECESARIOS CON EL USUARIO openldap PARA PODER REALIZAR EL EJERCICIO, DE AHI QUE HAYAMOS USADO ACL PARA PODER HACERLO CORRECTAMENTE.**

Cuando lo tengamos debemos de añadir esta configuracion a nuestro ldap, para ello usamos el siguiente comando:
```shell
#### Añadimos la configuracion ####
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f ldaps.ldif

#### Comprobamos que este bien ####
debian@freston:~/LDAP/ldaps$ sudo slapcat -b "cn=config" | grep -E "olcTLS" 
olcTLSCACertificateFile:: L2V0Yy9zc2wvY2VydHMvZ29uemFsb25hemFyZW5vLmNydCA=
olcTLSCertificateFile: /etc/ssl/certs/wildcard-franjavier.crt
olcTLSCertificateKeyFile: /etc/ssl/private/wildcard.key

#### Comprobamos que los ficheros esten bien  ####
debian@freston:~/LDAP/ldaps$ sudo slaptest -u
config file testing succeeded
```

Despues de realizar esto debemos de dirigirnos al fichero de configuracion de ldap que se encuentra en **/etc/ldap/ldap.conf** y debemos de buscar una linea llamada **TLS_CACERT** y debemos de cambiarla por lo siguiente:
```shell
...
TLS_CACERT      /etc/ssl/certs/gonzalonazareno.crt
...
```

Y tambien debemos de añadirle que pueda usar ldaps, por lo que nos dirigimos al fichero **/etc/default/slapd** y buscamos la linea llamada **SLAPD_SERVICES** y le añadimos el siguiente contenido:
```shell
...
SLAPD_SERVICES="ldap:/// ldapi:/// ldaps:///"
...
```

Cuando lo tengamos reiniciamos el servicio de ldaps y ya tendriamos el uso de ldaps en nuestro equipo.
