---
title: "Diferencias Almacenamiento"
date: 2021-02-18T18:08:42+01:00
draft: true
---

# Diferencias entre MySQL y Postgres para gestionar el almacenamiento.

En el siguiente articulo vamos a ver las limitaciones que presentan los gestores de bases de datos **MySQL** y **Postgres** en cuanto al almacenamiento respector a **Oracle**

## MySQL

MySQL es un Sistema Gestor de Base de Datos, donde presenta tener un sistema de almacenamiento sobre *tablespaces*, usando sus motores de almacenamiento, que son los motores de almacenamiento **InnoDB** y **NDB**.

* **NDB Cluster** es el motor de almacenamiento usado por **MySQL Cluster**, para implementar tablas que se particionan en varias máquinas.

* **InnoDB** es un motor de búsqueda, que proporcionan tablas transaccionales.

Vamos a ver la sintaxis a la hora de crear un tablespace, ya que tiene gran similitud a Oracle:

* **InnoDB**:
```sql
CREATE TABLESPACE tablespace_name
    [ADD DATAFILE 'file_name']
    [FILE_BLOCK_SIZE = value]
    [ENCRYPTION [=] {'Y' | 'N'}]
    [ENGINE [=] InnoDB];
```

* **NDB**:

Primero debemos crear un logfile, ya que es necesario para crear el tablespace.
```sql
CREATE LOGFILE GROUP logfile_group;
```

Ya podemos crear el tablespace.
```sql
CREATE TABLESPACE tablespace_name
    [ADD DATAFILE 'file_name']
    USE LOGFILE GROUP logfile_group
    [EXTENT_SIZE [=] extent_size]
    [INITIAL_SIZE [=] initial_size]
    [AUTOEXTEND_SIZE [=] autoextend_size]
    [MAX_SIZE [=] max_size]
    [NODEGROUP [=] nodegroup_id]
    [WAIT]
    [COMMENT [=] 'string']
    [ENGINE [=] NDB];
```

La principal diferencia con Oracle se puede encontrar a la hora de asignarle una cuota de almacenamiento ya que con el motor InnoDB, no es posible.


## Postgres

**PostgreSQL** es un Sistema Gestor de Bases de Datos, donde no presenta tener una cláusula de almacenamiento de datos como si presenta tener **Oracle** y **MySQL**.

Con la función, **pg_total_relation_size**, podemos controlar el almacenamiento que tiene la tabla y su sintaxis de creación es la siguiente a mostrar:
```sql
CREATE VIEW user_disk_usage AS SELECT r.rolname, SUM(pg_total_relation_size(c.oid)) AS total_disk_usage 
FROM pg_class c, pg_roles r 
WHERE c.relkind = 'r' 
AND c.relowner = r.oid 
GROUP BY c.relowner;
```

Si observamos la sintaxis mostrada, la función de control de espacio de almacenamiento es creada a través de una vista, donde en dicha vista, vamos a ir configurando la función “pg_total_relation_size” a una tabla en concreto de la base de datos. Con la vista creada, podremos ejecutarla en un futuro, para poder ver el espacio de almacenamiento ocupado en la base de datos.

Cabe destacar que aún presenta tener una función que nos informa del almacenamiento ocupado en una tabla de la base de datos **PostgreSQL**, no presenta tener cláusulas de almacenamiento, por lo que no podemos asignar una cuota de almacenamiento, crear tablespace y limitar el almacenamiento de una tabla en la base de datos.

## En conclusión

Cada Sistema Gestor de Base de Datos presenta tener una serie de características, diferencias y limitaciones a la hora de la gestión de almacenamiento de datos, así como el control del almacenamiento.

**MySQL** no puede asignar cuotas de almacenamiento a espacios de tablas creadas pero si permite tener una cláusula de almacenamiento similar, para los datos almacenados en una tabla de la base de datos.

**PostgreSQL** no tiene cláusulas de almacenamiento como presentan tener los Sistemas Gestores de Bases de Datos **Oracle** y **MySQL**, aunque si presenta tener una función que controla el almacenamiento de datos de una tabla de la base de datos.