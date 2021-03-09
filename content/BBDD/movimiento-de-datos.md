---
title: "Movimiento De Datos"
date: 2021-03-09T12:28:51+01:00
draft: true
---

1. Realiza una exportación del esquema de SCOTT usando la consola Enterprise Manager, con las siguientes condiciones:
* Exporta tanto la estructura de las tablas como los datos de las mismas
* Excluye la tabla BONUS y los departamentos sin empleados.
* Realiza una estimación previa del tamaño necesario para el fichero de exportación.
* Programa la operación para dentro de 5 minutos.
* Genera un archivo de log en el directorio raíz.

Realiza ahora la operación con Oracle Data Pump.

2. Importa el fichero obtenido anteriormente usando Enterprise Manager pero en un usuario distinto de la misma base de datos.

3. Realiza una exportación de la estructura de todas las tablas de la base de datos usando el comando expdp de Oracle Data Pump probando todas las posibles opciones que ofrece dicho comando y documentándolas adecuadamente.

4. Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con MySQL desde línea de comandos, documentando el proceso.

Primero tenemos que agregar unas tablas a la base de datos que queremos exportar para realizar la prueba.

```sql
MariaDB [(none)]> create database export;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> use export
Database changed

MariaDB [export]> show tables
    -> ;
+------------------+
| Tables_in_export |
+------------------+
| depart           |
+------------------+
1 row in set (0.000 sec)
```

Realizaremos la exportacion:
```shell
debian@pruebas-auditorias:~$ sudo mysqldump -u root -p export > export.sql
Enter password: 

debian@pruebas-auditorias:~$ ls
export.sql
```

Tendremos que crear la nueva base de datos para la importacion:
```sql
MariaDB [(none)]> create database import;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> Bye
```

Y ahora procedemos a importar los datos:
```shell
debian@pruebas-auditorias:~$ sudo mysql -u root -p import < export.sql 
Enter password: 

debian@pruebas-auditorias:~$ sudo mysql -u root -p
Enter password: 

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 41
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use import
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [import]> show tables;
+------------------+
| Tables_in_import |
+------------------+
| depart           |
+------------------+
1 row in set (0.000 sec)
```

5. Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con Postgres desde línea de comandos, documentando el proceso.

Para ello creamos una base de datos y la poblamos de datos para el ejemplo:
```sql
postgres=# create database export;
CREATE DATABASE

postgres=# \c export 
You are now connected to database "export" as user "postgres".

export=# \d
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | depart | table | postgres
(1 row)
```

Una vez hecho esto vamos a usar la herramienta llamada **pg_dump**:
```shell
postgres@postgres-audit:~$ pg_dump -U postgres export > export.sql

postgres@postgres-audit:~$ ls
11  export.sql
```

Ahora, si queremos importar la copia debemos de usar el comando **psql** desde el usuario **postgres**:
```shell
postgres@postgres-audit:~$ psql -U postgres < export.sql 
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
COPY 4
ALTER TABLE
```


6. Exporta los documentos de una colección de MongoDB que cumplan una determinada condición e impórtalos en otra base de datos.





7. SQL\*Loader es una herramienta que sirve para cargar grandes volúmenes de datos en una instancia de ORACLE. Exportad los datos de uno de vuestros proyectos de 1º desde Postgres a  texto plano con delimitadores y emplead SQL\*Loader para realizar el proceso de carga de dichos datos a una instancia ORACLE. Debéis explicar los distintos ficheros de configuración y de log que tiene SQL*Loader


# Fuentes

Importar y exportar en mysql - https://www.stackscale.com/es/blog/importar-exportar-bases-datos-mysql-mariadb/

Importar y exportar en postgresql - https://www.linuxito.com/programacion/333-como-exportar-y-restaurar-bases-de-datos-postgres