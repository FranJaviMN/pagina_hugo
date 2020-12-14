---
title: "Ldap Instalacion"
date: 2020-12-14T11:52:37+01:00
draft: true
---

# Instalacion de ldap en la maquina freston

Para ello tenemos en nuestro escenario una maquina llamada freston con una ip privada **10.0.1.11** accesible desde dulcinea, en esta maquina vamos a instalar un servidor ldap y en el vamos a crear dos unidades organizativas, una para personas y otra para los grupos.

Importante recalcar que nuestra maquina freston tiene como hostname **freston** y como fqdn **freston.franjavier.gonzalonazareno.org** lo cual es muy importante a la hora de la resolucion dns.

## Instalacion del paquete slapd

Para ello vamos a instalar desde repositorios el paquete llamado **slapd** que es el que contiene  el servidor ldap:
```shell
#### Instalamos slapd ####
debian@freston:~$ sudo apt install slapd

#### Nos salta la ventana de contraseña y la introducimos ####
#### Comprobamos el servicio de slapd ####
debian@freston:~$ sudo systemctl status slapd.service 
● slapd.service - LSB: OpenLDAP standalone server (Lightweight Directory Access Protocol)
   Loaded: loaded (/etc/init.d/slapd; generated)
   Active: active (running) since Wed 2020-12-09 18:33:33 CET; 1min 0s ago
```

Tenemos que tener en cuenta que se levanta en el puerto 389/tcp. Ahora si queremos ver el contenido en bruto podemos usar el siguiente comando:
```shell
#### Mostramos todo el servidor ldap ####
debian@freston:~$ sudo slapcat 
dn: dc=franjavier,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: IESGN
dc: franjavier
...
```

Tambien podemos usar las herramientas de ldap en el paquete **ldap-utils**:
```shell
debian@freston:~$ sudo apt install ldap-utils
```

Pero a nosotros nos interesa añadir las entradas de forma manual para tener mas control sobre estas, por lo que ejecutamos el siguiente comando y seguimos los pasos que nos va indicando:
```shell
debian@freston:~$ sudo dpkg-reconfigure -plow slapd
```
Donde:
1. Nombre de dominio DNS: franjavier.gonzalonazareno.org
2. Nombre de la organización: IESGN
3. Contraseña del administrador de OpenLDAP: xxxxxxxx
4. Motor de base de datos: MDB (opción por defecto).
5. Marcamos que si se elimine el directorio cuando slapd se elimine (opción por defecto).
6. Seleccionamos que se remueva el antiguo directorio (opción por defecto).

## Crear unidades organizativas

Para ello vamos a usar los fichero .ldif el cual es un estandar para las configuracion de ldap. Se escribe como texto plano y en el se pueden añadir, modificar y eliminar registros que luego se cargan mediante un comando. Es el formato en el que se muestran las salidas de slapcat y cada elemento se separa con una linea en blando.

Podemos hacerlo de distinta forma, o bien podemos hacerlo a traves de un fichero unico o bien dividiendolo en varios fichero. En mi caso lo hare mediante el fichero **unidadesorganizativas.ldif**.

```shell
#### Creamos el fichero de unidadesorganizativas.ldif ####
dn: ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: organizationalUnit
ou: Personas

dn: ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: organizationalUnit
ou: Grupos

#### Lo añadimos ####
root@freston:~# ldapadd -x -D 'cn=admin,dc=franjavier,dc=gonzalonazareno,dc=org' -W -f /home/debian/fich_ldif/unidadesorganizativas.ldif
Enter LDAP Password: 
adding new entry "ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org"

adding new entry "ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org"

```

Donde podemos ver que en el fichero hemos creado dos objetos de la clase **organizationalUnit** para organizar los grupos y usuarios que vayamos añadiendo. El nombre que le he puesto es simple, personas para usuario y grupos para grupos.

Luego vemos el comando que hemos usado el cual vemos como indicamos las credenciales del admin, e indicamos el fichero que vamos a añadir.

Ahora si hacemos de nuevo una vista en bruto de todos nuestros elementos veremos dos nuevos:
```shell
#### Ejecutamos slapcat ####

# Unidad personas #
dn: ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: organizationalUnit
ou: Personas
...

# Unidad grupos #
dn: ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: organizationalUnit
ou: Grupos
```