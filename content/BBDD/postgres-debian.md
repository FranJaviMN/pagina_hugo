---
title: "Postgres Debian"
date: 2020-12-09T13:45:06+01:00
draft: true
---

# Instalacion de un servidor Postgres

Lo primero que debemos de tener es una maquina virtual en la cual vamos a realizar esta instalacion, en mi caso voy a usar una maquina con un sistema debian 10, creada mediante vagrant:
```ruby
# Fichero vagrant
Vagrant.configure("2") do |config|
  config.vm.define :postgres do |postgres|
    postgres.vm.box = "debian/buster64"
    postgres.vm.hostname = "postgres"
    postgres.vm.network :public_network, :bridge=>"wlp2s0"
  end
end
```

Para poder instalar el servidor de postgreSQL vamos a instalar el paquete llamado **postgresql** el cual, al instalarlo y ver su version vemos que actualmente tenemos la version **11.9**. 
```shell
vagrant@postgres:~$ sudo apt install postgresql
```

Seguido de esto nos habra creado un nuevo usuario llamado **postgres** que es el que vamos a usar para poder usar la base de datos, para ello lo importante es cambiarle la contraseña a dicho usuario y, una vez le hayamos cambiado la contraseña entramos en dicho usuario e introducimos el comando **psql**:
```shell
#### Cambiamos la contraseña de postgres ####
root@postgres:~# passwd postgres
New password: 
Retype new password: 
passwd: password updated successfully

#### Entramos en el usuario ####
vagrant@postgres:~$ su postgres
Password: 

#### Una vez dentro ejecutamos psql ####
postgres@postgres:~$ psql
psql (11.9 (Debian 11.9-0+deb10u1))
Type "help" for help.

postgres=# 
```

Hecho esta ya estaremos en el interprete de comando de postgres, ahora lo que vamos a hacer es crear un nuevo usuario con una nueva base de datos que poblaremos con algunas tablas mas adelante:
```shell
#### Creamos el nuevo usuario llamado fran con contraseña fran ####
postgres=# CREATE USER cliente with PASSWORD 'cliente';

#### COnsultamos la tabla pg_user para ver nuestro usuario ####
postgres=# select * from pg_user;
 usename  | usesysid | usecreatedb | usesuper | userepl | usebypassrls |  passwd  | valuntil | useconfig 
----------+----------+-------------+----------+---------+--------------+----------+----------+-----------
 postgres |       10 | t           | t        | t       | t            | ******** |          | 
 cliente  |    16386 | f           | f        | f       | f            | ******** |          | 

(2 rows)
```

Ya tenemos nuestro usuario, ahora solo debemos de crear en ella una nueva base de datos a la que le asociaremos el usuario que hemos creado posteriormente, para ello debemos de realizar los siguientes comando en el interprete **psql**:
```shell
#### Creamos la base de datos ####
postgres=# CREATE DATABASE prueba_1;
CREATE DATABASE

#### Le damos los privilegios al usuario fran sobre la base de datos prueba_1 ####
postgres=# GRANT ALL PRIVILEGES ON DATABASE prueba_1 TO cliente;
GRANT
```
Una vez tengamos esto vamos a conectarnos a esa base de datos con el usuario **cliente** y una vez dentro usaremos el siguiente script con las tablas y datos creados en la base de datos:
```shell
#### Entramos en la base de datos con el usuario cliente ####
postgres@postgres:~$ psql -d prueba_1 -U cliente

#### Usamos el siguiente script ####
create table depart
(
dept_no integer,
dnombre varchar(20),
loc     varchar(20),
primary key (dept_no)
);

insert into depart
values ('10','CONTABILIDAD','SEVILLA');
insert into depart
values ('20','INVESTIGACION','MADRID');
insert into depart
values ('30','VENTAS','BARCELONA');
insert into depart
values ('40','PRODUCCION','BILBAO');


create table emple
(
emp_no    integer,
apellidos varchar(20),
oficio    varchar(20),
dir       integer,
fecha_alt date,
salario   integer,
comision  integer,
dept_no   integer,
primary key (emp_no),
foreign key (dept_no) references depart (dept_no)
);

insert into emple (emp_no, apellidos, oficio, dir, fecha_alt, salario, dept_no)
values ('7369','SANCHEZ','EMPLEADO','7902','17/12/1980','104000','20');
insert into emple
values ('7499','ARROYO','VENDEDOR','7698','20/02/1980','208000','39000','30');
insert into emple
values ('7521','SALA','VENDEDOR','7698','22/02/1981','162500','162500','30');
insert into emple (emp_no, apellidos, oficio, dir, fecha_alt, salario, dept_no)
values ('7566','JIMENEZ','DIRECTOR','7839','02/04/1981','386750','20');
insert into emple
values ('7654','MARTIN','VENDEDOR','7698','29/09/1981','162500','182000','30');
insert into emple(emp_no, apellidos, oficio, dir, fecha_alt, salario, dept_no)
values ('7698','NEGRO','DIRECTOR','7839','01/05/1981','370500','30');
insert into emple(emp_no, apellidos, oficio, dir, fecha_alt, salario, dept_no)
values ('7788','GIL','ANALISTA','7566','09/11/1981','390000','20');
insert into emple(emp_no, apellidos, oficio, fecha_alt, salario, dept_no)
values ('7839','REY','PRESIDENTE','17/11/1981','650000','10');
insert into emple
values ('7844','TOVAR','VENDEDOR','7698','08/09/1981','195000','0','30');
insert into emple(emp_no, apellidos, oficio, dir, fecha_alt, salario, dept_no)
values ('7876','ALONSO','EMPLEADO','7788','23/09/1981','143000','20');
insert into emple(emp_no, apellidos, oficio, dir, fecha_alt, salario, dept_no)
values ('7900','JIMENO','EMPLEADO','7698','03/12/1981','1235000','30');
insert into emple(emp_no, apellidos, oficio, dir, fecha_alt, salario, dept_no)
values ('7902','FERNANDEZ','ANALISTA','7566','03/12/1981','390000','20');
insert into emple(emp_no, apellidos, oficio, dir, fecha_alt, salario, dept_no)
values ('7934','MUÑOZ','EMPLEADO','7782','23/01/1982','169000','10');


create table herramientas
(
descripcion	varchar(20),
estanteria	int,
unidades	int,
constraint pk_herramientas primary key (descripcion)
);


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
```

Llegados a este punto ya tenemos nuestro usuario listo con nuestra base de datos preparada.

## Configurar PostgreSQL para acceso remoto o conexiones entrantes a través de la red

Nosotros necesitamos desde nuestro cliente pueda tener acceso a nuestro sistema gestor de base de datos, pudiendo ser tan versátil para él como privilegios se le hayan otorgado al usuario con el que realiza la conexión.

Para poder realizar esta accion es necesario realizar unas modificaciones en los ficheros de configuración del motor de base de datos.

### Fichero postgresql.conf

El fichero de configuración de PostgreSQL se encuentra en la ruta /etc/postgresql/9.4/main/postgresql.conf En él podemos establecer ajustes como la ubicación del resto de ficheros de configuración, las conexiones y autenticaciones, los recursos de los que dispondrá PostgreSQL…

EN el fichero debemos de buscar una linea como **listen_addresses = 'localhost'** la cual debemos de cambiar para poder hacer que nuestro cliente pueda tener acceso. En mi caso puede entrar cualquier ip a este servidor postgres, por lo que lo cambiaremos de la siguiente forma:
```shell
#### Linea cambiada ####
# - Connection Settings -

listen_addresses = '*'                  # what IP address(es) to listen on;

...
```

Ya tendriamos este aspecto resuelto, ahora debemos de confirgurar otro fichero.

### Fichero de configuración pg_hba.conf

También es necesario realizar cambios en el fichero de configuración pg_hba.conf, que es el fichero de configuración de autenticaciones de clientes PostgreSQL. Este se ubica en /etc/postgresql/9.4/main/pg_hba.conf

En él buscamos la línea que está bajo el comentario que indica la configuración de conexiones locales con IPv4 (IPv4 local connections) para reemplazar 127.0.0.1/32 por all

```shell
#### Cambiamos la linea ####
# IPv4 local connections:
host    all             all             all                     md5

#### Reiniciamos el servicio ####
vagrant@postgres:~$ sudo service postgresql restart
```

Una vez hecho esto debemos de reiniciar el servicio de postgres y probar desde un cliente si podemos entrar.

## Conexión desde un cliente a un servidor PostgreSQL remoto

Para ello, en nuestro cliente debemos de instalar el paquete llamado **postgresql-client** para poder usar el servidor postgres remoto.
```shell
#### Instalamos el cliente ####
francisco@debian10:~$ sudo apt install postgresql-client

#### Nos conectamos al servidor remoto ####
francisco@debian10:~$ psql -h 192.168.1.120 -U cliente -d prueba_1
Contraseña para usuario cliente:
```

Una vez hecho lo anterior ya estaremos en nuestro interprete de psql en nuestro cliente.

![prueba funciona](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/postgres/cliente-postgres.png)


# Instalación de una herramienta de administración web para Postgres y prueba desde un cliente remoto.

En mi caso la herramienta seleccionada para este ejercicio va a ser **phpPgAdmin**. phpPgAdmin es una herramienta web que esta creada en **php** y su finalidad es la de administrar de manera mas intuitiva y de forma grafica un servidor postgreSQL con sus respectivas limitaciones.

Lo primero que debemos de hacer es instalar el paquete de **apache2** que es el servidor web que vamos a usar para servir la aplicacion de **phpPgAdmin**. Posteriormente debemos de instalar dos paquetes mas, en mi caso como estoy usando la version de php mas nueva que hay en repositorios que es la version **php7.3**.
```shell
#### Instalamos apache2 ####
vagrant@postgres:~$ sudo apt install apache2

#### Instalamos phpPgAdmin ####
vagrant@postgres:~$ sudo apt install phppgadmin php7.3-pgsql
```

Una vez instalado se configura un alias por defecto en **/phppgadmin**, por lo que, si desde nuestro cliente intentamos entrar en dicho alias mediante la ip de nuestro servidor, veremos el siguiente mensaje de error:

![error postgres](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/postgres/error-postgres.png)

Esto se debe a que debemos de modificar unos parametros en la configuracion de nuestra programa en php.

## Accediendo remotamente a phpPgAdmin

Para poder acceder debemos de dirgirnos primero al fichero **phppgadmin.conf** que se encuentra en el directorio **/etc/apache2/conf-available/**, una vez alli debemos de buscar una linea que tenga de contenido *Require local* la cual debemos de comentar para poder seguir con el ejercicio:
```shell
#### Modificamos el fichero /etc/apache2/conf-available/phppgadmin.conf ####
vagrant@postgres:~$ sudo nano /etc/apache2/conf-available/phppgadmin.conf

...
# Only allow connections from localhost:
#Require local
...
```

Vemos que hemos comentado la linea *Require local* ya que, como bien indica, solo permite el acceso local a nuestra herramienta web de postgres, por lo que podemos comentarla o cambiar su valor.

Ahora si reiniciamos el servidio de apache2 e intentamos entrar de nuevo a nuestra herramienta web veremos que en este caso si nos dejara, apareciendo una ventana como la siguiente:

![ventana phpPgAdmin](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/postgres/phpPgAdmin.png)