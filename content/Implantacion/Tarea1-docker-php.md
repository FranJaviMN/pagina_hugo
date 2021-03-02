---
title: "Tarea1 Docker Php"
date: 2021-03-02T10:33:35+01:00
draft: true
---

**Repositorio git: https://github.com/FranJaviMN/Docker-php/tree/main/Tarea%201**

# Tarea 1: Ejecución de una aplicación web PHP en docker

* Queremos ejecutar en un contenedor docker la aplicación web escrita en PHP: bookMedik (https://github.com/evilnapsis/bookmedik).

* Es necesario tener un contenedor con mariadb donde vamos a crear la base de datos y los datos de la aplicación. El script para generar la base de datos y los registros lo encuentras en el repositorio y se llama schema.sql. Debes crear un usuario con su contraseña en la base de datos. La base de datos se llama bookmedik y se crea al ejecutar el script.

* Ejecuta el contenedor mariadb y carga los datos del script schema.sql. Para más información.

* El contenedor mariadb debe tener un volumen para guardar la base de datos.

* El contenedor que creas debe tener un volumen para guardar los logs de apache2.

* Crea una imagen docker con la aplicación desde una imagen base de debian o ubuntu. Ten en cuenta que el fichero de configuración de la base de datos (core\controller\Database.php) lo tienes que configurar utilizando las variables de entorno del contenedor mariadb. (Nota: Para obtener las variables de entorno en PHP usar la función getenv. Para más infomación).

* La imagen la tienes que crear en tu máquina con el comando docker build.

* Crea un script con docker compose que levante el escenario con los dos contenedores.(Usuario: admin, contraseña: admin).

## Contenedor con mariadb

debemos de irnos al directorio de **Deploy** y debemos de generar un fichero para **docker-compose** para poder desplegar nuestro contenedor mariadb, el contenido de dicho fichero **docker-compose.yml** debe de ser el siguiente:
```yml
version: "3.1"

services:
  db:
    container_name: servidor_mysql
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: bookmedik
      MYSQL_USER: bookmedik
      MYSQL_PASSWORD: bookmedik
      MYSQL_ROOT_PASSWORD: franciscojavier
    volumes:
      - /opt/mysql_wp:/var/lib/mysql
```

## Generar imagen bookmedik

Una vez hayamos creado la base de datos con nuestro fichero **docker-compose.yml**, debemos de generar una imagen donde tengamos instalado la aplicación **bookmedik** por lo que debemos de generer, en el directorio **Build** un fichero llamado **Dockerfile** en el cual vamos a indicar como se va a generar nuestra imagen y, a su vez, debemos de clonar el repositorio de [bookmedik](https://github.com/evilnapsis/bookmedik):
```Dockerfile
FROM debian

RUN apt-get update && apt-get install -y apache2 libapache2-mod-php7.3 php7.3 php7.3-mysql && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN rm /var/www/html/index.html

ENV APACHE_SERVER_NAME=www.bookmedik-francisco.org
ENV DATABASE_USER=bookmedik
ENV DATABASE_PASSWORD=bookmedik
ENV DATABASE_HOST=bd

EXPOSE 80

COPY ./bookmedik /var/www/html
ADD script.sh /usr/local/bin/script.sh

RUN chmod +x /usr/local/bin/script.sh

CMD ["/usr/local/bin/script.sh"]
```

Una vez tengamos este fichero debemos de generar en el mismo directorio de **Build** un fichero llamado **script.sh** en el cual vamos a indicar una serie de pasos que se debe de ejecutar como son la asignación de variables de entorno que necesita:
```shell
#!/bin/bash

sed -i 's/$this->user="root";/$this->user="'${DATABASE_USER}'";/g' /var/www/html/core/controller/Database.php
sed -i 's/$this->pass="";/$this->pass="'${DATABASE_PASSWORD}'";/g' /var/www/html/core/controller/Database.php
sed -i 's/$this->host="localhost";/$this->host="'${DATABASE_HOST}'";/g' /var/www/html/core/controller/Database.php
apache2ctl -D FOREGROUND
```

De esta forma estamos cambiando las variables de entorno para que asi pueda reconocer la base de datos que esta en nuestro otro contenedor como la ejecución de apache2. Una vez hecho debemos de generar nuestra nueva imagen a partir del fichero de Dockerfile, para ello debemos de irnos al directorio **Build** y ejecutamos el siguiente comando:
```shell
#### Construimos la imagen ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 1/Build$ docker build -t franjavimn/bookmedik:v1 .

#### Vemos la imagen que nos ha creado ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 1/Build$ docker image list
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
franjavimn/bookmedik          v1                  fa389d1f6461        22 minutes ago      251MB
...
```

## Desplegar docker-compose

Una vez hayamos creado la imagen, vamos a editar neustro fichero de **docker-compose.yml** y le vamos a añadir un nuevo contenedor, en este caso es donde vamos a tener nuestra aplicacion de **bookmedik**:
```yml
version: "3.1"

services:
  db:
    container_name: servidor_mysql
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: bookmedik
      MYSQL_USER: bookmedik
      MYSQL_PASSWORD: bookmedik
      MYSQL_ROOT_PASSWORD: franciscojavier
    volumes:
      - /opt/mysql_bookmedik:/var/lib/mysql

  bookmedik:
    container_name: bookmedik
    image: franjavimn/bookmedik:v1
    restart: always
    ports:
      - 8082:80
    volumes:
      - /opt/bookmedik:/var/log/apache2
```

Una vez hecho esto, debemos de pasar el script de la creacion de las tablas a nuestro contenedor de mariadb, este fichero esta en el directorio que hemos generado tras clonar el repositorio y su nombre es **schema.sql**:
```shell
#### Le pasamos el script ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 1/Deploy$ cat ../Build/bookmedik/schema.sql | docker exec -i servidor_mysql /usr/bin/mysql -u root --password=franciscojavier bookmedik
```

Y ahora, si entramos en **localhost:8082** veremos que nos sale nuestro login el cual es **admin/admin** y debe de salirnos algo como lo siguiente:

![imagen bookmedik](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/docker/bookmedik.png)


