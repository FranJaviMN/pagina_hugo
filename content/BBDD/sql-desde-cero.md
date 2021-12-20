---
title: "Sql desde cero"
date: 2021-10-12T20:52:50+02:00
draft: true
---

# Bases de datos

Una **base de datos** es una recopilación organizada de información o datos estructurados, que normalmente se almacena de forma electrónica en un sistema informático. Normalmente, una base de datos está controlada por un **sistema de gestión** de bases de datos **(DBMS)**. En conjunto, los datos y el DBMS, junto con las aplicaciones asociadas a ellos, reciben el nombre de **sistema de bases de datos**, abreviado normalmente a simplemente **base de datos**.

Los datos de los tipos más comunes de bases de datos en funcionamiento actualmente se suelen utilizar como **estructuras de filas y columnas** en una serie de tablas para aumentar la eficacia del procesamiento y la consulta de datos. Así, se puede acceder, gestionar, modificar, actualizar, controlar y organizar fácilmente los datos. La mayoría de las bases de datos utilizan un lenguaje de consulta estructurada (SQL) para escribir y consultar datos.

## Tipos de bases de datos

Existen muchos tipos diferentes de bases de datos. La mejor base de datos para una organización específica depende de cómo pretenda la organización utilizar los datos:

* **Bases de datos relacionales**: Las bases de datos se hicieron predominantes en la década de 1980. Los elementos de una base de datos relacional se organizan como un conjunto de tablas con columnas y filas. La tecnología de bases de datos relacionales proporciona la forma más eficiente y flexible de acceder a información estructurada.

![base de datos relacional](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/sql_desde_cero/bases_relacionales.jpeg)

* **Bases de datos orientadas a objetos**: La información de una base de datos orientada a objetos se representa en forma de objetos, como en la programación orientada a objetos.

![base de datos orientadas a objetos](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/sql_desde_cero/Modelo_orientado_a_objetos.png)

* **Bases de datos distribuidas**: Una base de datos distribuida consta de dos o más archivos que se encuentran en sitios diferentes. La base de datos puede almacenarse en varios ordenadores, ubicarse en la misma ubicación física o repartirse en diferentes redes.

![base de datos distribuidas](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/sql_desde_cero/Base_de_Datos_distribuida.jpg)

* **Almacenes de datos**: Un repositorio central de datos, un ***data warehouse*** es un tipo de base de datos diseñado específicamente para consultas y análisis rápidos.

![Almacenes de datos](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/sql_desde_cero/Data_warehouse.JPG)

* **Bases de datos NoSQL**: Una base de datos NoSQL, o **base de datos no relacional**, permite almacenar y manipular datos no estructurados y semiestructurados (a diferencia de una base de datos relacional, que define cómo se deben componer todos los datos insertados en la base de datos). Las bases de datos NoSQL se hicieron populares a medida que las aplicaciones web se volvían más comunes y complejas.

![base de datos NoSQL](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/sql_desde_cero/NSQL.jpg)

* **Bases de datos orientadas a grafos**: Una base de datos orientada a grafos almacena datos relacionados con entidades y las relaciones entre entidades.

![base de datos orientadas a grafos](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/sql_desde_cero/GraphDatabase.png)

* **Bases de datos OLTP**: Una base de datos OLTP es una base de datos rápida y analítica diseñada para que muchos usuarios realicen un gran número de transacciones.

# Bases de datos relacionales: SQL

**SQL** (Structured Query Language) es un lenguaje estándar e interactivo de acceso a bases de datos relacionales que permite especificar diversos tipos de operaciones en ellas, gracias a la utilización del álgebra y de cálculos relacionales, el SQL brinda la posibilidad de realizar consultas con el objetivo de recuperar información de las bases de datos de manera sencilla. Las consultas toman la forma de un lenguaje de comandos que permite seleccionar, insertar, actualizar, averiguar la ubicación de los datos, y más.

El SQL se desarrolló por primera vez en IBM en la década de 1970 con Oracle como uno de los principales contribuyentes, lo que dio lugar a la implementación del estándar ANSI SQL. El SQL ha propiciado muchas ampliaciones de empresas como IBM, Oracle y Microsoft. Aunque el SQL se sigue utilizando mucho hoy en día, están empezando a aparecer nuevos lenguajes de programación.