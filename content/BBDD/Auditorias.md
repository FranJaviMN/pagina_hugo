---
title: "Auditorias"
date: 2021-03-04T12:55:26+01:00
draft: true
---

# Auditoría de Bases de Datos 

1. Activa desde SQL\*Plus la auditoría de los intentos de acceso fallidos al sistema. Comprueba su funcionamiento

Para activar la auditoría de los intentos de acceso fallidos primero tenemos que ver los parámetros de auditoría que tenemos en nuestra base de datos:
```sql
SQL> SHOW PARAMETER AUDIT

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      C:\APP\FRANJ\ADMIN\ORCL\ADUMP
audit_sys_operations                 boolean     FALSE
audit_trail                          string      DB
```

Una vez hecho esto y visto los parametros vamos a activar el parametro *audit_trail*:
```sql
SQL> ALTER SYSTEM SET audit_trail=db scope=spfile;

Sistema modificado.
```

Una vez hechos los cambios debemos de ejecutarlos por lo que debemos de reiniciar para que estos se vean reflejados:
```sql
SQL> SHUTDOWN
```

Y una vez reiniciada vamos a ejecutar una nueva auditoria:
```sql
SQL> AUDIT CREATE SESSION WHENEVER NOT SUCCESSFUL;

AuditorÝa terminada correctamente.

#### Vemos las auditorias activas ####
SQL> SELECT * FROM dba_priv_audit_opts;
```

Una vez hecho esto debemos de salir y hacer pruebas, para ello vamos a fallar la contraseña para iniciar sesion para que asi haya datos en las auditorias:
```sql
SQL> SELECT os_username, username, extended_timestamp, action_name, returncode
  2  FROM dba_audit_session;

OS_USERNAME
--------------------------------------------------------------------------------
USERNAME
------------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
ACTION_NAME                  RETURNCODE
---------------------------- ----------
DESKTOP-78T4DF0\franj
SYSTEM
02/03/21 18:03:00,221000 +01:00
LOGOFF                                0


OS_USERNAME
--------------------------------------------------------------------------------
USERNAME
------------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
ACTION_NAME                  RETURNCODE
---------------------------- ----------
DESKTOP-78T4DF0\franj
SYSTEM
02/03/21 18:07:31,196000 +01:00
LOGOFF                                0

#### Desactivamos la auditoria creada ####
SQL> NOAUDIT CREATE SESSION WHENEVER NOT SUCCESSFUL;

No auditorÝa terminada correctamente.
```


2. Realiza un procedimiento en PL/SQL que te muestre los accesos fallidos junto con el motivo de los mismos, transformando el código de error almacenado en un mensaje de texto comprensible.

3. Activa la auditoría de las operaciones DML realizadas por SCOTT. Comprueba su funcionamiento.

Para ello, primero, debemos de activar una nueva auditoria:
```sql
SQL> AUDIT INSERT TABLE, UPDATE TABLE, DELETE TABLE BY SCOTT BY ACCESS;

AuditorÝa terminada correctamente.
```

Una vez hecho, vamos a realizar unas accionas para comprobar que funciona correctamente:
```sql
#### Realizamos las pruebas necesarias ####
SQL> INSERT INTO dept VALUES(50,'RRHH','ALCALA');

1 fila creada.

SQL> UPDATE dept SET loc='Marchena' WHERE deptno=50;

1 fila actualizada.

SQL> DELETE FROM dept WHERE deptno=50;

1 fila suprimida.

#### Comprobamos las acciones que ha realizado el usuario SCOTT ####
SQL> SELECT obj_name, action_name, timestamp
  2  FROM dba_audit_object
  3  WHERE username='SCOTT';

OBJ_NAME
--------------------------------------------------------------------------------
ACTION_NAME                  TIMESTAM
---------------------------- --------
SDO_GEOR_DDL__TABLE$$
DELETE                       02/03/21

DEPT
DELETE                       02/03/21

DEPT
UPDATE                       02/03/21


OBJ_NAME
--------------------------------------------------------------------------------
ACTION_NAME                  TIMESTAM
---------------------------- --------
DEPT
INSERT                       02/03/21

DEPT
INSERT                       02/03/21
```

4. Realiza una auditoría de grano fino para almacenar información sobre la inserción de empleados del departamento 10 en la tabla emp de scott.

Lo primero que tenemos que hacer es crear el procedimiento para controlar la inserción de los empleados del departamento:
```sql
SQL> BEGIN
  2      DBMS_FGA.ADD_POLICY (
  3          object_schema      =>  'SCOTT',
  4          object_name        =>  'EMP',
  5          policy_name        =>  'policy1',
  6          audit_condition    =>  'DEPTNO = 10',
  7          statement_types    =>  'INSERT'
  8      );
  9  END;
 10  /

Procedimiento PL/SQL terminado correctamente.
```

Y ahora consultamos la auditoria que hemos creado:
```sql
SQL> SELECT object_schema, object_name, policy_name, policy_text
  2  FROM dba_audit_policies;

OBJECT_SCHEMA                  OBJECT_NAME
------------------------------ ------------------------------
POLICY_NAME
------------------------------
POLICY_TEXT
--------------------------------------------------------------------------------
SCOTT                          EMP
POLICY1
DEPTNO = 10
```

5. Explica la diferencia entre auditar una operación by access o by session.

La diferencia es que "by access" realiza un registro por cada sentencia auditada y "by session" agrupa las sentencias por tipo en un registro por cada sesión iniciada.

Ahora vamos a ver un ejemplo de ambas operaciones:

* **By session**
```sql
SQL> SELECT owner, obj_name, action_name, timestamp, priv_used
  2  FROM dba_audit_object
  3  WHERE username='SYSTEM';

OWNER
------------------------------
OBJ_NAME
--------------------------------------------------------------------------------
ACTION_NAME                  TIMESTAM PRIV_USED
---------------------------- -------- ----------------------------------------
PUBLIC
PRODUCT_PROFILE
DROP PUBLIC SYNONYM          30/03/10

PUBLIC
PRODUCT_USER_PROFILE
DROP PUBLIC SYNONYM          30/03/10
```

* **By access**
```sql
SQL> select obj_name, action_name, timestamp
  2  from dba_audit_object
  3  where username='SCOTT';

OBJ_NAME
--------------------------------------------------------------------------------
ACTION_NAME                  TIMESTAM
---------------------------- --------
SDO_GEOR_DDL__TABLE$$
DELETE                       02/03/21

DEPT
DELETE                       02/03/21

DEPT
UPDATE                       02/03/21

```


6. Documenta las diferencias entre los valores db y db, extended del parámetro audit_trail de ORACLE. Demuéstralas poniendo un ejemplo de la información sobre una operación concreta recopilada con cada uno de ellos.

Vamos a ver las principales diferencias entre db y db, extended con el parámetro audit_trail.

* **db**: Activa la auditoría y los datos se almacenan en la tabla SYS.AUD$.

* **db,extended**: Aparte de almacenar los datos en la tabla SYS.AUD$, este escribirá los valores en las columnas SQLBIND y SQLTEXT de la misma tabla; SYS.AUD$.

Ahora vamos a ver un ejemplo:
```sql
#### Vemos el estado de las auditorias ####
SQL> show parameter audit;

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      C:\APP\FRANJ\ADMIN\ORCL\ADUMP
audit_sys_operations                 boolean     FALSE
audit_trail                          string      DB

#### Activamos db,extended ####
SQL> ALTER SYSTEM SET audit_trail = DB,EXTENDED SCOPE=SPFILE;

Sistema modificado.

#### Debemos de reiniciar para que se hagan los cambios ####
SQL> show parameter audit;

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      C:\APP\FRANJ\ADMIN\ORCL\ADUMP
audit_sys_operations                 boolean     FALSE
audit_trail                          string      DB, EXTENDED
```

7. Localiza en Enterprise Manager las posibilidades para realizar una auditoría e intenta repetir con dicha herramienta los apartados 1, 3 y 4.

8. Averigua si en Postgres se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.

PostgreSQL por defecto no incorpora una herramienta de auditorías, por lo que, si queremos conseguir realizar auditorías, tendremos que descargarnos una herramienta que ha desarrollado la comunidad, llamada Audit trigger 91plus.

Para ello, desde git, descargamos el repositorio con el fichero de la siguiente forma:
```shell
debian@postgres-audit:~$ wget https://raw.githubusercontent.com/2ndQuadrant/audit-trigger/master/audit.sql
```

Y una vez descargado entramos en nuestro servidor y ejecutamos el siguiente comando:
```sql
postgres=# \i audit.sql 
CREATE EXTENSION
CREATE SCHEMA
REVOKE
COMMENT
CREATE TABLE
REVOKE
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
COMMENT
CREATE INDEX
CREATE INDEX
CREATE INDEX
CREATE FUNCTION
COMMENT
CREATE FUNCTION
COMMENT
CREATE FUNCTION
CREATE FUNCTION
COMMENT
CREATE VIEW
COMMENT
```

Y ahora solo nos queda activar la recolección de los datos de las auditorias, para ello solo debemos de ejecutar la siguiente sentencia:
```sql
postgres=# select audit.audit_table({nombre_de_la_tabla});
```


9. Averigua si en MySQL se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.

Para ello vamos a crear una nueva base de datos llamada **prueba** con una tabla llamada **depart**:
```sql
#### Creamos la base de datos ####
MariaDB [(none)]> create database prueba;
Query OK, 1 row affected (0.001 sec)

#### Creamos la tabla y le añadimos datos ####
MariaDB [prueba]> create table depart
    -> (
    -> dept_no integer,
    -> dnombre varchar(20),
    -> loc     varchar(20),
    -> primary key (dept_no)
    -> );
Query OK, 0 rows affected (0.103 sec)

MariaDB [prueba]> insert into depart
    -> values ('10','CONTABILIDAD','SEVILLA');
Query OK, 1 row affected (0.018 sec)

MariaDB [prueba]> insert into depart
    -> values ('20','INVESTIGACION','MADRID');
Query OK, 1 row affected (0.004 sec)

MariaDB [prueba]> insert into depart
    -> values ('30','VENTAS','BARCELONA');
Query OK, 1 row affected (0.003 sec)

MariaDB [prueba]> insert into depart
    -> values ('40','PRODUCCION','BILBAO');
Query OK, 1 row affected (0.002 sec)
```

Ahora debemos de crear una nueva base de datos que se va a llamar **auditorias**:
```sql
#### Creamos la nueva base de datos ####
MariaDB [prueba]> create database auditorias;
Query OK, 1 row affected (0.001 sec)

MariaDB [prueba]> use auditorias
Database changed

#### Creamos una nueva tabla para la salida de los trigger ####
MariaDB [auditorias]> CREATE TABLE accesos
    ->  (
    ->    codigo int(11) NOT NULL AUTO_INCREMENT,
    ->    usuario varchar(100),
    ->    fecha datetime,
    ->    PRIMARY KEY (`codigo`)
    ->  )
    ->  ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;
Query OK, 0 rows affected (0.021 sec)
```

Una vez tengamos la tabla vamos a crear el trigger que nos permite auditar los insert en la tabla **depart** de la base de datos **prueba**:
```sql
MariaDB [prueba]> DELIMITER $$
MariaDB [prueba]> CREATE TRIGGER auditar_insert 
    -> BEFORE INSERT ON prueba.depart
    -> FOR EACH ROW
    -> BEGIN
    ->   INSERT INTO auditorias.accesos (usuario, fecha)
    ->   values (CURRENT_USER(), NOW());
    -> END$$
Query OK, 0 rows affected (0.024 sec)
```

Una vez hecho el trigger vamos a añadir una nueva fila a nuestra tabla llamada **depart**:
```sql
MariaDB [prueba]> insert into depart values ('50','PRUEBA','ALCALA');
Query OK, 1 row affected (0.021 sec)
```

Hecho eso nos dirigimos a la base de datos **auditorias** y consultamos la tabla **accesos**:
```sql
MariaDB [prueba]> use auditorias
Database changed
MariaDB [auditorias]> select * from accesos;
    -> $$
+--------+----------------+---------------------+
| codigo | usuario        | fecha               |
+--------+----------------+---------------------+
|      1 | root@localhost | 2021-03-04 11:51:43 |
|      2 | root@localhost | 2021-03-04 11:51:58 |
+--------+----------------+---------------------+
2 rows in set (0.001 sec)
```

10. Averigua las posibilidades que ofrece MongoDB para auditar los cambios que va sufriendo un documento.

MongoDB si nos permite realizar ciertas auditorías para ver los permitidos pondremos el siguiente comando:

```shell
--auditFilter
```

Con el siguiente ejemplo de a continuacion vamos a auditar las acciones **createCollection** y **dropCollection**:
```shell
{ atype: { $in: [ "createCollection", "dropCollection" ] } }
```

Y debemos de poner el filtro entre comillas simples para pasar el documento como una cadena:
```shell
mongod --dbpath data/db --auditDestination file --auditFilter '{ atype: { $in: [ "createCollection", "dropCollection" ] } }' --auditFormat BSON --auditPath data/db/auditLog.bson
```

11. Averigua si en MongoDB se pueden auditar los accesos al sistema.

Si, leyendo la documentacion de MongoDB, es algo parecido a lo realizado anteriormente, solamente tendremos que cambiar los parametros:
```shell
{ atype: "authenticate", "param.db": "test" }
```

Al igual que antes podemos usarlo como una cadena, si lo ponemos entre comillas simples.
```shell
mongod --dbpath data/db --auth --auditDestination file --auditFilter '{ atype: "authenticate", "param.db": "test" }' --auditFormat BSON --auditPath data/db/auditLog.bson
```

# Fuentes

Documentacion de Mongodb -> https://docs.mongodb.com/manual/tutorial/configure-audit-filters/

Auditorias en Postgres -> https://usuarioperu.com/2018/07/23/auditoria-de-tablas-en-postgresql-i/

AUDIT_TRAIL -> https://docs.oracle.com/cd/B19306_01/server.102/b14237/initparams016.htm#REFRN10006

DBMS_FGA -> https://docs.oracle.com/database/121/ARPLS/d_fga.htm#ARPLS225

Auditorias en Mariadb -> https://costaricamakers.com/?p=2149