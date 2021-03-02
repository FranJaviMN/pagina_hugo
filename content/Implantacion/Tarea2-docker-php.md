---
title: "Tarea2 Docker Php"
date: 2021-03-02T10:33:38+01:00
draft: true
---

**Repositorio git: https://github.com/FranJaviMN/Docker-php/tree/main/Tarea%202**

# Tarea 2: Ejecución de una aplicación web PHP en docker

* Realiza la imagen docker de la aplicación a partir de la imagen oficial PHP que encuentras en docker hub. Lee la documentación de la imagen para configurar una imagen con apache2 y php, además seguramente tengas que instalar alguna extensión de php.

* Crea esta imagen en docker hub

* Crea un script con docker compose que levante el escenario con los dos contenedores.

Para ello vamos a seguir los mismo pasos que hemos seguido en el ejercicio 1, primero lo que vamos a hacer es irnos a la documentación de la imagen de php que tenemos en [DockerHub](https://hub.docker.com/_/php/)

Una vez la hayamos leido vamos a generar, en primer lugar nuestra nuevo **Dockerfile**

## Generando nueva imagen

Vamos a generar una imagen a partir de la imagen de **php:7.4-apache** para poder hacer lo que hicimos en el ejercicio 1 pero esta vez con otra imagen, por lo que vamos a generar la imagen con el siguiente fichero de **Dockerfile**:
```dockerfile
FROM php:7.4-apache

RUN docker-php-ext-install pdo pdo_mysql mysqli json
RUN a2enmod rewrite

ENV DATABASE_USER=bookmedik
ENV DATABASE_PASSWORD=bookmedik
ENV DATABASE_HOST=db

EXPOSE 80

WORKDIR /var/www/html

COPY ./bookmedik /var/www/html
ADD script.sh /usr/local/bin/script.sh

RUN chmod +x /usr/local/bin/script.sh

CMD ["/usr/local/bin/script.sh"]
```

Debemos de tener en cuenta que ahora nuestro nuevo fichero de *script.sh* es el mismo que en el ejercicio 1, ya que vamos a desplegar la aplicación de **bookmedik** por lo que tenemos el siguiente fichero **script.sh**:
```shell
#!/bin/bash

sed -i 's/$this->user="root";/$this->user="'${DATABASE_USER}'";/g' /var/www/html/core/controller/Database.php
sed -i 's/$this->pass="";/$this->pass="'${DATABASE_PASSWORD}'";/g' /var/www/html/core/controller/Database.php
sed -i 's/$this->host="localhost";/$this->host="'${DATABASE_HOST}'";/g' /var/www/html/core/controller/Database.php
apache2ctl -D FOREGROUND
```

Una vez hecho solo debemos de generar la imagen con el comando **build**:
```shell
#### Generamos la imagen ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 2/Build$ docker build -t franjavimn/bookmedik-php:v1 .

#### Comprobamos que se haya creado ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 2/Build$ docker image list
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
franjavimn/bookmedik-php      v1                  c8d4ad15a095        5 hours ago         418MB
```

## Creando el fichero docker-compose

Una vez hayamos creado la imagen, vamos a editar neustro fichero de **docker-compose.yml** y le vamos a añadir un nuevo contenedor, en este caso es donde vamos a tener nuestra aplicacion de **bookmedik**:
```yml
version: "3.1"

services:
  db:
    container_name: servidor_mysql-php
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: bookmedik
      MYSQL_USER: bookmedik
      MYSQL_PASSWORD: bookmedik
      MYSQL_ROOT_PASSWORD: franciscojavier
    volumes:
      - /opt/mysql_bookmedik-php:/var/lib/mysql

  bookmedik:
    container_name: bookmedik-php
    image: franjavimn/bookmedik-php:v1
    restart: always
    ports:
      - 8083:80
    volumes:
      - /opt/bookmedik-php:/var/log/apache2
```

Una vez hecho esto, debemos de pasar el script de la creacion de las tablas a nuestro contenedor de mariadb, este fichero esta en el directorio que hemos generado tras clonar el repositorio y su nombre es **schema.sql**:
```shell
#### Levantamos el escenario ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 2/Deploy$ docker-compose up -d
Creating servidor_mysql-php ... done
Creating bookmedik-php      ... done

#### Le pasamos el script ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 2$ cat ../Build/bookmedik/schema.sql | docker exec -i servidor_mysql-php /usr/bin/mysql -u root --password=franciscojavier bookmedik
```

Y ahora, si entramos en **localhost:8083** veremos que nos sale nuestro login el cual es **admin/admin** y debe de salirnos algo como lo siguiente:

![imagen de php con bookmedik](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/docker/bookmedik-php.png)