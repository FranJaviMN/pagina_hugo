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

Lo primero que podemos hacer es ver las opciones que tenemos con el comando **expdp**:
```shell
PS C:\Users\franj> expdp HELP=Y

Export: Release 11.2.0.1.0 - Production on Mar Mar 9 12:42:34 2021

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.


La utilidad de exportaci¾n de pump de datos proporciona un mecanismo para transferir objetos de datos
entre bases de datos Oracle. La utilidad se llama con el siguiente comando:

   Ejemplo: expdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp

Puede controlar c¾mo se ejecuta la exportaci¾n introduciendo el comando 'expdp' seguido
por varios parßmetros. Para especificar los parßmetros, utilice palabras clave:

   Formato:  expdp KEYWORD=valor o KEYWORD=(valor1,valor2,...,valorN)
   Ejemplo: expdp scott/tiger DUMPFILE=scott.dmp DIRECTORY=dmpdir SCHEMAS=scott
               o TABLES=(T1:P1,T1:P2), si T1 es una tabla particionada

USERID debe ser el primer parßmetro de la lÝnea de comandos.
```

Y ahora vamos a ver la opciones que nos ofrece:
```shell
ATTACH
Conectar a un trabajo existente.
Por ejemplo, ATTACH=nombre_trabajo.

COMPRESSION
Reduce el tama±o de un archivo de volcado.
Los valores de palabras clave vßlidos son: ALL, DATA_ONLY, [METADATA_ONLY] y NONE.

CONTENT
Especifica los datos que se van a descargar.
Los valores de palabras clave vßlidos son: [ALL], DATA_ONLY y METADATA_ONLY.

DATA_OPTIONS
Indicadores de opciones de nivel de datos.
Los valores de palabra clave vßlidos son: XML_CLOBS.

DIRECTORY
Objeto de directorio que se va a utilizar para los archivos de volcado y log.

DUMPFILE
Lista de nombres de archivo de volcado de destino [expdat.dmp].
Por ejemplo, DUMPFILE=scott1.dmp, scott2.dmp, dmpdir:scott3.dmp.

ENCRYPTION
Cifra todo o parte del archivo de volcado.
Los valores de palabras clave vßlidos son: ALL, DATA_ONLY, ENCRYPTED_COLUMNS_ONLY, METADATA_ONLY y NONE.

ENCRYPTION_ALGORITHM
Especifica c¾mo se debe hacer el cifrado.
Los valores de palabras clave vßlidos son: [AES128], AES192 y AES256.

ENCRYPTION_MODE
MÚtodo para generar la clave de cifrado.
Los valores de palabras clave vßlidos son: DUAL, PASSWORD y [TRANSPARENT].

ENCRYPTION_PASSWORD
Clave de contrase±a para crear datos cifrados en un archivo de volcado.

ESTIMATE
Calcula estimaciones de trabajo.
Los valores de palabras clave vßlidos son: [BLOCKS] y STATISTICS.

ESTIMATE_ONLY
Calcula las estimaciones de trabajo sin realizar la exportaci¾n.

EXCLUDE
Excluye tipos de objeto especÝficos.
Por ejemplo, EXCLUDE=SCHEMA:"='HR'".

FILESIZE
Especifica el tama±o de cada archivo de volcado en unidades de bytes.

FLASHBACK_SCN
SCN utilizado para restablecer la instantßnea de sesi¾n.

FLASHBACK_TIME
Tiempo utilizado para buscar el valor de SCN correspondiente mßs cercano.

FULL
Exporta toda la base de datos [N].

HELP
Muestra mensajes de ayuda [N].

INCLUDE
Incluye tipos de objetos especÝficos.
Por ejemplo, INCLUDE=TABLE_DATA.

JOB_NAME
Nombre del trabajo de exportaci¾n que se va a crear.

LOGFILE
Especifica el nombre del archivo log [export.log].

NETWORK_LINK
Nombre del enlace de base de datos remota al sistema de origen.

NOLOGFILE
No se escribe el archivo log [N].

PARALLEL
Cambia el n·mero de workers activos para el trabajo actual.

PARFILE
Especifica el nombre del archivo de parßmetros.

QUERY
Clßusula de predicado utilizada para exportar un subjuego de una tabla.
Por ejemplo, QUERY=employees:"WHERE identificador_departamento > 10".

REMAP_DATA
Especifica una funci¾n de conversi¾n de datos.
Por ejemplo, REMAP_DATA=EMP.EMPNO:REMAPPKG.EMPNO.

REUSE_DUMPFILES
Sobrescribe el archivo de volcado de destino si existe (N).

SAMPLE
Porcentaje de datos para exportar.

SCHEMAS
Lista de esquemas que se van a exportar (esquema de conexi¾n).

SOURCE_EDITION
Edici¾n que se usarß para extraer metadatos.

STATUS
Estado del trabajo de frecuencia (seg) que se va a controlar donde
el valor por defecto (0) mostrarß el nuevo estado cuando estÚ disponible.

TABLES
Identifica una lista de tablespaces que se van a exportar.
Por ejemplo, TABLES=HR.EMPLOYEES,SH.SALES:SALES_1995.

TABLESPACES
Identifica una lista de tablespaces que se van a exportar.

TRANSPORTABLE
Especifica si el mÚtodo transportable se puede utilizar.
Los valores de palabras clave vßlidos son: ALWAYS y [NEVER].

TRANSPORT_FULL_CHECK
Verifica segmentos de almacenamiento de todas las tablas [N].

TRANSPORT_TABLESPACES
Lista de tablespaces desde los que se descargarßn los metadatos.

VERSION
Versi¾n de los objetos que se van a exportar.
Los valores de palabra cable vßlidos son: [COMPATIBLE], LATEST o cualquier versi¾n de base de datos vßlida.

------------------------------------------------------------------------------

Los siguientes comandos son vßlidos en el modo interactivo.
Nota: Estßn permitidas las abreviaturas.

ADD_FILE
Agrega un archivo de volcado al juego de archivos de volcado.

CONTINUE_CLIENT
Vuelve al modo de registro. El trabajo se reiniciarß si estß inactivo.

EXIT_CLIENT
Sale de la sesi¾n del cliente y deja el trabajo ejecutßndose.

FILESIZE
Tama±o de archivo (bytes) por defecto para los comandos ADD_FILE posteriores.

HELP
Resume los comandos interactivos.

KILL_JOB
Desconecta y suprime un trabajo.

PARALLEL
Cambia el n·mero de workers activos para el trabajo actual.

REUSE_DUMPFILES
Sobrescribe el archivo de volcado de destino si existe [N].

START_JOB
Inicia o reanuda el trabajo actual.
Los valores de palabra clave vßlidos son: SKIP_CURRENT.

STATUS
Estado del trabajo de frecuencia (seg) que se va a controlar donde
el valor por defecto (0) mostrarß el nuevo estado cuando estÚ disponible.

STOP_JOB
Cierra en orden la ejecuci¾n del trabajo y sale del cliente.
Los valores de palabra clave vßlidos son: IMMEDIATE.
```

Vamos a realizar una exportación de ejemplo:
```shell
PS C:\Users\franj> expdp system dumpfile=prueba_copia.dmp include=table content=metadata_only full=y

Export: Release 11.2.0.1.0 - Production on Mar Mar 9 12:50:34 2021

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.
Contrase±a:

Conectado a: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Iniciando "SYSTEM"."SYS_EXPORT_FULL_01":  system/******** dumpfile=prueba_copia.dmp include=table content=metadata_only full=y
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/TABLE
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/PRE_TABLE_ACTION
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/GRANT/CROSS_SCHEMA/OBJECT_GRANT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/INDEX/INDEX
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/CONSTRAINT/CONSTRAINT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/COMMENT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/FGA_POLICY
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/CONSTRAINT/REF_CONSTRAINT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/INDEX/FUNCTIONAL_AND_BITMAP/INDEX
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/INDEX/STATISTICS/FUNCTIONAL_AND_BITMAP/INDEX_STATISTICS
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/INDEX/DOMAIN_INDEX/INDEX
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/POST_TABLE_ACTION
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/TRIGGER
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/POST_INSTANCE/PROCACT_INSTANCE
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/POST_INSTANCE/PROCDEPOBJ
La tabla maestra "SYSTEM"."SYS_EXPORT_FULL_01" se ha cargado/descargado correctamente
******************************************************************************
El juego de archivos de volcado para SYSTEM.SYS_EXPORT_FULL_01 es:
  C:\APP\FRANJ\ADMIN\ORCL\DPDUMP\PRUEBA_COPIA.DMP
El trabajo "SYSTEM"."SYS_EXPORT_FULL_01" ha terminado correctamente en 12:52:43
```

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
