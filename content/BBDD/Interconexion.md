---
title: "Interconexion"
date: 2021-02-08T18:19:40+01:00
draft: true
---

# Interconexión de Servidores de Bases de Datos

Las interconexiones de servidores de bases de datos son operaciones que pueden ser muy útiles en diferentes contextos. Básicamente, se trata de acceder a datos que no están almacenados en nuestra base de datos, pudiendo combinarlos con los que ya tenemos.

En esta práctica veremos varias formas de crear un enlace entre distintos servidores de bases de datos. Se pide los tres siguientes puntos:

1. Realizar un enlace entre dos servidores de bases de datos ORACLE, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.

2. Realizar un enlace entre dos servidores de bases de datos Postgres, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.

3. Realizar un enlace entre un servidor ORACLE y otro Postgres o MySQL empleando Heterogeneus Services, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.

Los servidores que se vayan a enlacer entre si deben de estar en maquinas distintas.

# Enlace entre servidores oracle

Para ello vamos a usar dos maquinas creadas en un escenario de **Vagrant** en el cual hemos usado el siguiente fichero de **Vagrantfile**
```ruby
Vagrant.configure("2") do |config|
    config.vm.define "oracle1" do |oracle1|
        config.vm.provider "virtualbox" do |vb|
            vb.name = "oracle1"
            vb.memory = 2048
            vb.cpus = 1
        end
        oracle1.vm.box = "neko-neko/centos6-oracle-11g-XE"
        oracle1.vm.hostname = "oracle1"
        oracle1.vm.network "public_network",
            bridge: "wlp2s0", 
            use_dhcp_assigned_default_route: true
    end
    config.vm.define "oracle2" do |oracle2|
        config.vm.provider "virtualbox" do |vb|
            vb.name = "oracle2"
            vb.memory = 2048
            vb.cpus = 1
        end
        oracle2.vm.box = "neko-neko/centos6-oracle-11g-XE"
        oracle2.vm.hostname = "oracle2"
        oracle2.vm.network "public_network",
            bridge: "wlp2s0",
            use_dhcp_assigned_default_route: true
    end
end
```

Donde vemos que las maquinas estan basadas en un sistema **centOS 6**. Una vez tengamos el escenario nos metemos en las maquinas y vamos a empezar con la configuracion de las maquinas para la interconexión:

Antes ejecutamos los siguientes comandos como **root** para cambiar el idioma y diversos parametros en ambas maquinas:
```shell
[root@oracle1 ~]# sed -ri 's/NLS_LANG=.*/NLS_LANG="SPANISH_SPAIN.AL32UTF8"/g' /etc/profile.d/oracle_env.sh
[root@oracle1 ~]# export NLS_LANG='SPANISH_SPAIN.AL32UTF8'

[root@oracle1 ~]# cat <<EOF>>$ORACLE_HOME/sqlplus/admin/glogin.sql
> alter session set NLS_DATE_FORMAT = 'DD-MM-YYYY HH24:mi:ss';
> set sqlprompt "_DATE _USER'@'_CONNECT_IDENTIFIER 'SQL'> "
> set pagesize 2000 linesize 100
> set serveroutput on
> EOF
```

## Maquina oracle1: Fichero listener.ora

Lo primero será modificar el fichero de configuración listener.ora. En este fichero podremos configurar el interlocutor de oracle(listener) el cual se encarga de aceptar peticiones remotas desde la red(TCP). Este fichero se encuentra en el directorio **$ORACLE_HOME/network/admin/listener.ora** y debemos de tener el siguiente contenido en el fichero:
```shell
[root@oracle1 ~]# cat /u01/app/oracle/product/11.2.0/xe/network/admin/listener.ora 
# listener.ora Network Configuration File:
LISTENER =
    (DESCRIPTION_LIST =
        (DESCRIPTION =
            (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
        )
    )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (SID_NAME = XE)
      (GLOBAL_DBNAME = XE)
      (ORACLE_HOME = /u01/app/oracle/product/11.2.0/xe)
    )
  )

DEFAULT_SERVICE_LISTENER = (XE)
```

Donde:
* LISTENER: Es dónde especificamos los protocolos,IPs o nombres(HOST), puertos y etc. desde los que se podrán conectar remotamente a este servidor. Usamos *localhost* para que cualquier maquina de la red pueda conectarse
* SID_LIST_LISTENER: Son los nombre de los servicios de escucha dónde especificamos los nombres de las instancias y directorio de las bases de datos.
* DEFAULT_SERVICE_LISTENER: Por defecto el servicio de escucha se llama "XE".

Una vez hecho reiniciamos los servicios del **listener** y comprobamos su estado:
```shell
#### Reiniciamos el servicio llamado lsnrctl ####
[root@oracle1 ~]# lsnrctl stop && lsnrctl start

#### Comprobamos el servicio ####
[root@oracle1 ~]# lsnrctl status

LSNRCTL for Linux: Version 11.2.0.2.0 - Production on 04-FEB-2021 18:10:51

Copyright (c) 1991, 2011, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 11.2.0.2.0 - Production
Start Date                04-FEB-2021 17:54:33
Uptime                    0 days 0 hr. 16 min. 18 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Default Service           XE
Listener Parameter File   /u01/app/oracle/product/11.2.0/xe/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/oracle1/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=oracle1)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=oracle1)(PORT=8080))(Presentation=HTTP)(Session=RAW))
Services Summary...
Service "XE" has 2 instance(s).
  Instance "XE", status UNKNOWN, has 1 handler(s) for this service...
  Instance "XE", status READY, has 1 handler(s) for this service...
Service "XEXDB" has 1 instance(s).
  Instance "XE", status READY, has 1 handler(s) for this service...
The command completed successfully
```

## Maquina oracle2: Fichero tnsname.ora

Este apartado es para el cliente desde el cual nos conectaremos.

Primero haremos un ping para ver si tenemos conexion con la maquina **oracle1**:
```shell
#### Comprobamos ping ####
[root@oracle2 ~]# ping 192.168.1.121
PING 192.168.1.121 (192.168.1.121) 56(84) bytes of data.
64 bytes from 192.168.1.121: icmp_seq=1 ttl=64 time=0.476 ms
64 bytes from 192.168.1.121: icmp_seq=2 ttl=64 time=0.410 ms
64 bytes from 192.168.1.121: icmp_seq=3 ttl=64 time=0.496 ms
64 bytes from 192.168.1.121: icmp_seq=4 ttl=64 time=0.295 ms
^C
--- 192.168.1.121 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3369ms
```

Ahora vamos a hacer una prueba de conexion a la base de datos de nuestro maquina **oracle1**:
```shell
[root@oracle2 ~]# rlwrap sqlplus system/vagrant@//192.168.1.121:1521/XE

SQL*Plus: Release 11.2.0.2.0 Production on Jue Feb 4 18:26:39 2021

Copyright (c) 1982, 2011, Oracle.  All rights reserved.

ERROR:
ORA-28002: the password will expire within 7 days

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

Session altered.

04-02-2021 18:26:39 SYSTEM@//192.168.1.121:1521/XE SQL> 
```

Ahora para realizar la interconexión del servidor necesitamos configurar el fichero tnsnames.ora. En este fichero podemos configurar y "mapear" los nombres de los servicios que está escuchando el servidor. Lo encontraremos en el directorio **$ORACLE_HOME/network/admin/tnsnames.ora**
```shell
# tnsnames.ora Network Configuration File:

tns_ora1 =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.1.121)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = XE)
    )
  )
```

Y comprobamos mediante el comando **tnsping** si tenemos conexion mediante el siguiente comando:
```shell
[root@oracle2 ~]# tnsping tns_ora1

TNS Ping Utility for Linux: Version 11.2.0.2.0 - Production on 04-FEB-2021 18:30:27

Copyright (c) 1997, 2011, Oracle.  All rights reserved.

Used parameter files:

Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.1.121)(PORT = 1521)) (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = XE)))
OK (10 msec)
```

## Configuración de interconexión mediante enlace de base de datos

Ahora necesitamos configurar dicho enlace en el cliente desde "sqlplus".
```shell
#### Creamos el link de la base de datos ####
04-02-2021 18:35:51 SYSTEM@XE SQL> CREATE DATABASE LINK link_ora1 CONNECT TO scott IDENTIFIED BY tiger USING 'tns_ora1';

Database link created.

#### Consultamos la base de datos de oracle1 ####

04-02-2021 18:47:07 SYSTEM@XE SQL> select * from scott.emp@link_ora1;

     EMPNO ENAME      JOB	       MGR HIREDATE		      SAL	COMM	 DEPTNO
---------- ---------- --------- ---------- ------------------- ---------- ---------- ----------
      7369 SMITH      CLERK	      7902 17-12-1980 00:00:00	      800		     20
      7499 ALLEN      SALESMAN	      7698 20-02-1981 00:00:00	     1600	 300	     30
      7521 WARD       SALESMAN	      7698 22-02-1981 00:00:00	     1250	 500	     30
```




# Enlace entre servidores Postgres

Para ello vamos a tener a nuestra disposición des maquinas en **openstack** que las vamos a llamas **interconexion1 e interconexion2** en las cuales vamos a realizar las pruebas de enlace entre dos bases de datos postgres, para ello, lo primero que debemos de hacer es instalar en ambas maquinas la base de datos postgres:
```shell
#### Maquina de interconexion1 ####
debian@interconexion-1:~$ sudo apt update && sudo apt install postgresql -y

#### Maquina de interconexion2 ####
debian@interconexion-2:~$ sudo apt update && sudo apt install postgresql -y
```

Ahora que tenemos ambas maquinas con postgresql instalado, vamos a proceder a realizar los cambios pertinentes para poder hacer que ambas bases de datos esten enlazadas entre si. Asi que ahora vamos a configurar los servidores para que permitan las conexiones remotas, debemos de tener en cuenta que esta configuración se debe de hacer en ambas maquinas.

Primero vamos al fichero **/etc/postgresql/11/main/postgresql.conf** en el cual debemos de indicar que debe de escuchar en un rango de red que nosotros le indiquemos, en nuestro caso vamos a permitir cualquier rango de red, asi debe de quedar dicho fichero en **AMBAS MAQUINAS**:
```shell
#### Fichero postgresql.conf en la maquina interconexion1 ####
# - Connection Settings -

listen_addresses = '*'

#### Fichero postgresql.conf en la maquina interconexion2 ####
# - Connection Settings -

listen_addresses = '*'
```

Despues de esta modificacion debemos de dirigirnos al fichero **/etc/postgresql/11/main/pg_hba.conf** en el cual vamos a indicar que permita la conexión remota desde cualquier dirección ip de la red, introduciendo la siguiente línea que aparece a continuación en **AMBAS MAQUINAS**:
```shell
#### Fichero pg_hba.conf en interconexion1 ####
# Permitimos la conexion remota
host    all             all             0.0.0.0/0               md5

#### Fichero pg_hba.conf en interconexion2 ####
# Permitimos la conexion remota
host    all             all             0.0.0.0/0               md5
```

Una vez hecho estos dos pasos en **AMBAS MAQUINAS** debemos de reiniciar los servicios de postgresql en nuestra maquina mediante el siguiente comando:
```shell
#### Reiniciamos en ambas maquinas ####
debian@interconexion-1:~$ sudo systemctl restart postgresql

debian@interconexion-2:~$ sudo systemctl restart postgresql
```

# Poblando ambas bases de datos

Ahora que tenemos habilitado la conexion remota, debemos de poblar nuestras bases de datos con algunos datos para poder realizar las pruebas de enlace entre las bases de datos mas adelante, por ello, en cada maquina vamos a crear un nuevo usuario, una base de datos y le daremos todos los privilegios a dicho usuario sobre la base de datos que hayamos creado, por lo que vamos a proceder a su creacion:

## Maquina interconexion1

En esta maquina vamos a tener creado un usuario llamado **usuario1** el cual va a tener todos los privilegios sobre la base de datos llamada **base_prueba1**, para ello debemos de seguir los siguientes pasos:
```shell
#### Nos conectamos a la base de datos postgresql ####
debian@interconexion-1:~$ sudo su - postgres
postgres@interconexion-1:~$ psql

#### Creamos el nuevo usuario ####
postgres=# create user usuario1 with password 'usuario1';
CREATE ROLE

#### Creamos la nueva base de datos ####
postgres=# create database base_prueba1;
CREATE DATABASE

#### Le damos a usuario1 los privilegios sobre base_prueba1 ####
postgres=# grant all privileges on database base_prueba1 to usuario1;
GRANT
```

Una vez hayamos realizado estos pasos, salimos y nos conectamos con el usuario que hemos creado para asi poder crear una tabla nueva en la base de datos nueva:
```shell
#### Nos conectamos con el usuario creado ####
postgres@interconexion-1:~$ psql -h localhost -U usuario1 -W -d base_prueba1
Password: 
...
base_prueba1=>

#### Creamos la siguiente tabla ####
create table depart
(
    dept_no integer,
    dnombre varchar(20),
    loc     varchar(20),
    primary key (dept_no)
);

#### Le insertamos los valores a la tabla ####
insert into depart
values ('10','CONTABILIDAD','SEVILLA');
insert into depart
values ('20','INVESTIGACION','MADRID');
insert into depart
values ('30','VENTAS','BARCELONA');
insert into depart
values ('40','PRODUCCION','BILBAO');

#### Comprobamos la tabla ####
base_prueba1=> select * from depart;
 dept_no |    dnombre    |    loc    
---------+---------------+-----------
      10 | CONTABILIDAD  | SEVILLA
      20 | INVESTIGACION | MADRID
      30 | VENTAS        | BARCELONA
      40 | PRODUCCION    | BILBAO
(4 rows)
```

Ya tenemos nuestra maquina **interconexion1** configurada, ahora debemos de hacer la misma configuracion pero para la maquina **interconexion2**

## Maquina interconexion2

En esta maquina vamos a seguir el mismo procedimiento que hemos hecho en la maquina **interconexion1** y es el de crear un nuevo usuario, una nueva base de datos y en ella crear una tabla y poblarla:
```shell
#### Creamos el nuevo usuario ####
postgres=# create user usuario2 with password 'usuario2';
CREATE ROLE

#### Creamos la nueva base de datos ####
postgres=# create database base_prueba2;
CREATE DATABASE

#### Le damos a usuario2 los privilegios en la base de datos base_prueba2 ####
postgres=# grant all privileges on database base_prueba2 to usuario2;
GRANT
```

Una vez hayamos realizado estos pasos, salimos y nos conectamos con el usuario que hemos creado para asi poder crear una tabla nueva en la base de datos nueva:
```shell
#### Nos conectamos a la nueva base de datos con el nuevo usuario ####
postgres@interconexion-2:~$ psql -h localhost -U usuario2 -W -d base_prueba2
Password:
...
base_prueba2=>

#### Creamos la siguiente tabla ####
create table herramientas
(
    descripcion	varchar(20),
    estanteria	int,
    unidades	int,
    constraint pk_herramientas primary key (descripcion)
);

#### Le insertamos los siguiente valores ####
insert into herramientas
values('Alicates',1,10);
insert into herramientas
values('Cortador',4,5);
insert into herramientas
values('Destornillador',3,7);
insert into herramientas
values('Escafina',6,5);
insert into herramientas
values('Guantes',2,7);
insert into herramientas
values('Lima',6,10);
insert into herramientas
values('Martillo',3,10);
insert into herramientas
values('Metro',5,15);
insert into herramientas
values('Sierra',4,5);
insert into herramientas
values('Soldador',1,15);

#### Realizamoz una consulta para comprobarlo ####
base_prueba2=> select * from herramientas;
  descripcion   | estanteria | unidades 
----------------+------------+----------
 Alicates       |          1 |       10
 Cortador       |          4 |        5
 Destornillador |          3 |        7
 Escafina       |          6 |        5
 Guantes        |          2 |        7
 Lima           |          6 |       10
 Martillo       |          3 |       10
 Metro          |          5 |       15
 Sierra         |          4 |        5
 Soldador       |          1 |       15
(10 rows)
```

Cuando tengamos estos pasos ya realizados vamos a proceder a la interconexion entre la maquina1 entre la maquina2 y luego entre la maquina2 y la maquina1.

# Enlace entre maquina1 y maquina2

Para conectar la máquina1 con la máquina2 tendremos que dirigirnos primero a la máquina1 y estando en ella nos conectaremos con un usuario que tenga privilegios de crear un link (enlace) y procederemos a crearlo. Nos conectamos a la base de datos con el usuario con privilegios y creamos el link.
```shell
#### Nos conectamos al usuario postgres ####
postgres@interconexion-1:~$ psql -d base_prueba1
psql (11.9 (Debian 11.9-0+deb10u1))
Type "help" for help.

base_prueba1=# 

#### Creamos un link que lo llamaremos dblink ####
base_prueba1=# create extension dblink;
CREATE EXTENSION
```

Una vez hecho esto, nos conectamos con el usuario **usuario1**, cuando estemos conectados lo que realizaremos es una consulta al link creado anteriormente al cual vamos a indicarle los parametros para poder acceder a la base de datos de la otra máquina, datos como el nombre de la base de datos, la ip de la máquina, el nombre de usuario con su contraseña, seguido de la consulta select que se le realiza a una tabla y ademas de ello le asignamos los nombres a los campos a mostrar.
```shell
#### Hacemos la consulta a la base de datos remota ####
base_prueba1=> select * from dblink
('dbname=base_prueba2 host=10.0.0.15 user=usuario2 password=usuario2', 'select * from herramientas') as herramientas
(descripcion varchar, estanteria int, unidades int);

descripcion   | estanteria | unidades 
----------------+------------+----------
 Alicates       |          1 |       10
 Cortador       |          4 |        5
 Destornillador |          3 |        7
 Escafina       |          6 |        5
 Guantes        |          2 |        7
 Lima           |          6 |       10
 Martillo       |          3 |       10
 Metro          |          5 |       15
 Sierra         |          4 |        5
 Soldador       |          1 |       15
(10 rows)
```

# Enlace entre maquina2 y maquina1

Ahora debemos de hacer lo mismo pero a la inversa, es decir, debemos de crear link de la maquina2 a la maquina1:
```shell
#### Con el usuario postgres creamos el link ####
postgres@interconexion-2:~$ psql -d base_prueba2
psql (11.9 (Debian 11.9-0+deb10u1))
Type "help" for help.

base_prueba2=# 

#### Creamos el link que lo llamaremos dblink ####
base_prueba2=# create extension dblink;
CREATE EXTENSION
```

Nos conectaremos con el usuario que hemos creado anteriormente y conectados a el, podremos realizar una consulta a la tabla de la máquina1 introduciendo el mismo comando introducido anteriormente pero adaptandolo a este caso.
```shell
#### Hacemos la consulta a la base de datos remota ####
base_prueba2=# select * from dblink
('dbname=base_prueba1 host=10.0.0.11 user=usuario1 password=usuario1', 'select * from depart') as depart
(dept_no integer, dnombre varchar, loc varchar);
 dept_no |    dnombre    |    loc    
---------+---------------+-----------
      10 | CONTABILIDAD  | SEVILLA
      20 | INVESTIGACION | MADRID
      30 | VENTAS        | BARCELONA
      40 | PRODUCCION    | BILBAO
(4 rows)
```