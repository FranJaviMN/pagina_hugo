---
title: "Usuarios ACL Grupos"
date: 2020-12-21T13:21:30+01:00
draft: true
---

# Usuarios, grupos y ACLs en LDAP

En este documento vamos a proceder a la creacion de un escenario de ldap en el que vamos a tener los siguientes puntos a completar:

* Crea 10 usuarios con los nombres que prefieras en LDAP, esos usuarios deben ser objetos de los tipos posixAccount e inetOrgPerson. Estos usuarios tendrán un atributo userPassword.

* Crea 3 grupos en LDAP dentro de una unidad organizativa diferente que sean objetos del tipo groupOfNames. Estos grupos serán: comercial, almacen y admin

* Añade usuarios que pertenezcan a:
    * Solo al grupo comercial
    * Solo al grupo almacen
    * Al grupo comercial y almacen
    * Al grupo admin y comercial
    * Solo al grupo admin

* Modifica OpenLDAP apropiadamente para que se pueda obtener los grupos a los que pertenece cada usuario a través del atributo "memberOf".

* Crea las ACLs necesarias para que los usuarios del grupo almacen puedan ver todos los atributos de todos los usuarios pero solo puedan modificar las suyas.

* Crea las ACLs necesarias para que los usuarios del grupo admin puedan ver y modificar cualquier atributo de cualquier objeto.

## Crea 10 usuarios con los nombres que prefieras en LDAP

En este ejercicio vamos a usar un script que, mediante un fichero csv nos va a generar un fichero ldif con las 10 personas que queremos añadir, para ello debemos de, o bien crear nuestro propio fichero csv o podemos busacrlo en github. En mi caso voy a coger un fichero csv del siguiente [repositorio](https://github.com/HaykInanc/personCSV/blob/master/perosons.csv) de los cuales vamos a coger solo 10 personas y vamos a cambiar el genero por la contraseña, quedando el fichero de la siguiente forma:
```csv
Lanita,Maraga,lmaraga0@bloglines.com,
Joleen,Lamburne,jlamburne1@mapy.cz,
Joseph,Ranahan,jranahan2@taobao.com,
Deva,Blaker,dblaker3@bandcamp.com,
Irving,Picott,ipicott4@cbsnews.com,
Garrott,Fieldgate,gfieldgate5@nba.com,
Willie,Gee,wgee6@telegraph.co.uk,
Sully,,scolqueran7@ed.gov,
Jason,Kidstone,jkidstone8@wp.com,
Bobinette,Hernik,bhernik9@creativecommons.org,
```

Para generar las claves aleatorias he usado el comando **pwgen** que esta en el paquete con el mismo nombre que el binario. Lo hemos hecho con el siguiente script en **bash** y lo hemos ejecutado en modo superusuario para asi generar un fichero con la contraseña aleatoria y la contraseña cifrada:
```bash
#!/bin/bash

# El siguiente script nos genera un fichero con las claves codificadas junto a la clave en claro para poder usarlas en ldap.
# Nos movemos a la carpeta del usuario root para poder tener los ficheros protegidos
cd /root/
# Generamos el fichero con las claves en claro, que posteriormente sera eliminado dicho fichero por seguridad
for i in {1..10}
do
        echo $(pwgen -cn 8 1) >> passwd.txt
done
# Generamos un fichero con las claves y su codificacion
while IFS= read -r pass
do
        echo "$pass,$(slappasswd -s $pass)" >> shapasswd.txt
done < passwd.txt
# Borramos el fichero con las contraseñas en claro
rm passwd.txt
```

Si ejecutamos el script, veremos que en el directorio de root tenemos un fichero con la contraseña en claro y codificada para poder introducir los usuarios a nuestro ldap. El fichero .csv se nos quedaria de la siguiente forma:
```shell
Lanita,Maraga,lmaraga0@bloglines.com,{SSHA}jjEEhPdIjcLJHC+VWA+U05lvEYJQwVoV
Joleen,Lamburne,jlamburne1@mapy.cz,{SSHA}2s8JtaEp8v+bXfN598MlG2uEm+Yf4N4Z
Joseph,Ranahan,jranahan2@taobao.com,{SSHA}S02IXcYNs/wU054fCsnvcTsIFDfKGzpu
Deva,Blaker,dblaker3@bandcamp.com,{SSHA}uUsheb7UZ/0EtMpXR02L7TPOn27LKsPg
Irving,Picott,ipicott4@cbsnews.com,{SSHA}MKmeM70l2fTa03Nb7+KRL6Tm+BV6GBRS
Garrott,Fieldgate,gfieldgate5@nba.com,{SSHA}2IAjgCgQfJg1vT8k5qEASBrRKwAJG0Yu
Willie,Gee,wgee6@telegraph.co.uk,{SSHA}Um7G3DheRZQRWmEJXCpQtHwJtZAihDIq
Sully,,scolqueran7@ed.gov,{SSHA}q106tZuNnwgWY24/PNaPOAWqUIGfP8KN
Jason,Kidstone,jkidstone8@wp.com,{SSHA}IEvD4Fek71iVjQ/XeRiRuRPuTu2TprIi
Bobinette,Hernik,bhernik9@creativecommons.org,{SSHA}bxtPJ5LJz0JWA0/XGhc+970so1BvmsZr
```

Ahora vamos a generar un fichero .ldif a partir de este fichero .csv para poder introducirlo en nuestro ldap y asi poder tener usuarios en nuestro ldap, para ello usamos el script siguiente:
```bash
#!/bin/bash

# Vamos a contar la lineas de nuestro nuevo fichero .csv para poder saber cuantas vueltas debe hacer el while que vamos a implementar, aunque
#no haria falta ya que sabemos que va a tener 10 lineas. Donde tenemos persons.csv ponemos nuestro fichero .csv

num_lineas_csv=`cat persons.csv | wc -l`
# Para poder poblar nuestro ldap con usuarios debemos de tener en cuenta que deben de tener un uid y un gid los cuales vamos a iniciar en 2000 y 2500.

uid=2000
gid=2500
# Iniciamos el while donde, mientras haya usuarios en nuestro fichero seguira dentro del while, cuando ya no tenga mas usuarios saldra de él.

while [ $num_lineas_csv -gt 0 ]
do
# Guardamos los elementos del fichero csv en variables para poder ir creando nuestro fichero .ldif
	nombre=`cat persons.csv | head -$num_lineas_csv | tail -1 | cut -d "," -f1`
	apellidos=`cat persons.csv | head -$num_lineas_csv | tail -1 | cut -d "," -f2`
	correo=`cat persons.csv | head -$num_lineas_csv | tail -1 | cut -d "," -f3`
	password=`cat persons.csv | head -$num_lineas_csv | tail -1 | cut -d "," -f4`
# Generamos ahora si el fichero .ldif

	echo "dn: uid=$apellidos,ou=Personas,dc=gonzalonazareno,dc=org" >> personas.ldif
	echo "objectClass: inetOrgPerson" >> personas.ldif
	echo "objectClass: posixAccount" >> personas.ldif
	echo "objectClass: top" >> personas.ldif
	echo "uid:" $apellidos >> personas.ldif
	echo "gidNumber:" $gid >> personas.ldif
	echo "uidNumber:" $uid >> personas.ldif
	echo "homeDirectory:" /home/$nombre.$apellidos >> personas.ldif
	echo "loginShell:" /bin/bash >> personas.ldif
	echo "description: Elementos para $nombre $apellidos" >> personas.ldif
	echo "sn:" $apellidos >> personas.ldif
	echo "givenName:" $nombre >> personas.ldif
	echo "cn:" $nombre $apellidos >> personas.ldif
	echo "mail:" $correo >> personas.ldif
	echo "userPassword:" $password >> personas.ldif
	echo "" >> personas.ldif
# Usamos un contador para ir restando los usuarios y sumando el uid.

	let num_lineas_csv=num_lineas_csv-1
	let uid=uid+1
done
```

Despues de ejecutar nuestro script nos va a generar un fichero .ldif el cual, mediante el siguiente comando vamos a usar para poder meter todos los usuarios en ldap:
```shell
ldapadd -x -D cn=admin,dc=franjavier,dc=gonzalonazareno,dc=org -W -f personas.ldif
Enter LDAP Password: 
adding new entry "uid=Goldberg,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org"

adding new entry "uid=Gee,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org"

adding new entry "uid=Fieldgate,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org"

adding new entry "uid=Picott,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org"
...
```

De esta forma ya tendremos todos estos usuarios añadidos a nuestra unidad de **Personas**.

## Crea 3 grupos en LDAP dentro de una unidad organizativa diferente que sean objetos del tipo groupOfNames. Estos grupos serán: comercial, almacen y admin

Añade usuarios que pertenezcan a:
* Solo al grupo comercial:

* Solo al grupo almacen: 

* Al grupo comercial y almacen:

* Al grupo admin y comercial:

* Solo al grupo admin:

Para ello vamos a tener el siguiente fichero .ldif el cual contiene los datos para la creacion de los grupos:
```ldif
dn: cn=comercial,ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: comercial
member: uid=Hernik,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Kidstone,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Gee,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Blaker,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Ranahan,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org

dn: cn=almacen,ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: almacen
member: uid=Kidstone,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Goldberg,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Blaker,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Lamburne,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Maraga,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org

dn: cn=admin,ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: admin
member: uid=Hernik,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Fieldgate,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Picott,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
member: uid=Ranahan,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
```

Y este fichero lo añadimos a nuestro ldap con el siguiente comando:
```shell
debian@freston:~/LDAP/grupos-usuarios-acls/grupos$ ldapadd -x -D cn=admin,dc=franjavier,dc=gonzalonazareno,dc=org -W -f grupos.ldif
```

## Modifica OpenLDAP apropiadamente para que se pueda obtener los grupos a los que pertenece cada usuario a través del atributo "memberOf".

Para ello debemos de crear un nuevo fichero .ldif en el cual debemos de tener lo siguiente:
```ldif
Fichero de member_of.ldif
dn: cn=module,cn=config
cn: module
objectClass: olcModuleList
objectclass: top
olcModuleLoad: memberof.la
olcModulePath: /usr/lib/ldap

Fichero memberof_config.ldif
dn: olcOverlay={0}memberof,olcDatabase={1}mdb,cn=config
objectClass: olcConfig
objectClass: olcMemberOf
objectClass: olcOverlayConfig
objectClass: top
olcOverlay: memberof
olcMemberOfDangling: ignore
olcMemberOfRefInt: TRUE
olcMemberOfGroupOC: groupOfNames
olcMemberOfMemberAD: member
olcMemberOfMemberOfAD: memberOf

Fichero refint.ldif -> Fichero de relacion entre ellos
dn: cn=module,cn=config
cn: module
objectclass: olcModuleList
objectclass: top
olcmoduleload: refint.la
olcmodulepath: /usr/lib/ldap

dn: olcOverlay={1}refint,olcDatabase={1}mdb,cn=config
objectClass: olcConfig
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
objectClass: top
olcOverlay: {1}refint
olcRefintAttribute: memberof member manager owner
```

Y a continuacion vamos a ir añadiendo cada uno de ellos de la siguiente forma:
```shell
#### Añadimos el fichero de member_of.ldif ####
debian@freston:~/LDAP/grupos-usuarios-acls/member_of$ sudo ldapadd -Y EXTERNAL -H ldapi:/// -f member_of.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=module,cn=config"

#### Añadimos el fichero de memberof_config.ldif ####
debian@freston:~/LDAP/grupos-usuarios-acls/member_of$ sudo ldapadd -Y EXTERNAL -H ldapi:/// -f memberof_config.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "olcOverlay={0}memberof,olcDatabase={1}mdb,cn=config"

#### Añadimos la relacion entre ellos ####
debian@freston:~/LDAP/grupos-usuarios-acls/member_of$ sudo ldapadd -Y EXTERNAL -H ldapi:/// -f refint.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=module,cn=config"

adding new entry "olcOverlay={1}refint,olcDatabase={1}mdb,cn=config"
```

Una vez hecho esto debemos de borrar todos los grupos que hemos creado antes, por lo que debemos de usar el comando **ldapdelete** e ir borrando los grupos uno a uno, en este caso los tres que habiamos creado:
```shell
#### Borramos el grupo admin ####
debian@freston:~$ sudo ldapdelete -x -D "cn=admin,dc=franjavier,dc=gonzalonazareno,dc=org" 'cn=admin,ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org' -W

#### Borramos el grupo comercial ####
debian@freston:~$ sudo ldapdelete -x -D "cn=admin,dc=franjavier,dc=gonzalonazareno,dc=org" 'cn=comercial,ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org' -W

#### Borramos el grupo almacen ####
debian@freston:~$ sudo ldapdelete -x -D "cn=admin,dc=franjavier,dc=gonzalonazareno,dc=org" 'cn=almacen,ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org' -W
```

Y ahora que ya tenemos borrados los grupos, volvemos a añadirlos para que asi podamos consultar a traves de *member of*:
```shell
#### Añadimos los grupos de nuevo ####
debian@freston:~/LDAP/grupos-usuarios-acls/grupos$ ldapadd -x -D cn=admin,dc=franjavier,dc=gonzalonazareno,dc=org -W -f grupos.ldif
```

Y ahora, a la buscar los usuarios de nuestros ldap, por ejemplo, mediante su uid, veremos que nos saldra su informacion y si al final de la busqueda le añadimos la opcion **member of** veremos que tambien nos va a decir a que grupos pertenece el usuario por el que estamos realizando la busqueda, de tal forma que si, por ejemplo, en mi ldap tengo un usuario con un uid=Hernik y quiero saber a que grupos pertenece, es tan sencillo como realizar la siguiente consulta:

```shell
debian@freston:~$ ldapsearch -LL -Y EXTERNAL -H ldapi:/// "(uid=Hernik)" -b dc=franjavier,dc=gonzalonazareno,dc=org memberOf
SASL/EXTERNAL authentication started
SASL username: gidNumber=1000+uidNumber=1000,cn=peercred,cn=external,cn=auth
SASL SSF: 0
version: 1

dn: uid=Hernik,ou=Personas,dc=franjavier,dc=gonzalonazareno,dc=org
memberOf: cn=comercial,ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org
memberOf: cn=admin,ou=Grupos,dc=franjavier,dc=gonzalonazareno,dc=org
```

Como vemos nos indica que el usuario con uid=Hernik pertenece a los grupos *comercial* y *admin*.