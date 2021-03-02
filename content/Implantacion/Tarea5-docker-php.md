---
title: "Tarea5 Docker Php"
date: 2021-03-02T10:33:43+01:00
draft: true
---

**Repositorio git: https://github.com/FranJaviMN/Docker-php/tree/main/Tarea%205**

# Tarea 5: Ejecución de un CMS en docker

Busca una imagen oficial de un CMS PHP en docker hub (distinto al que has instalado en la tarea anterior, ni wordpress), y crea los contenedores necesarios para servir el CMS, siguiendo la documentación de dockerhub.

En mi caso vamos a instalar el cms llamado **Mediawiki** y [aquí](https://hub.docker.com/_/mediawiki) podemos ver su pagina oficial.

Siguiendo la documentación que nos ofrece la imagen en dockerhub, vamos a generar un fichero **docker-compose** con el cual vamos a levantar los contenedores que necesitemos, para ello solo debemos de leer la documentación para poder enetender como funciona y que variables de entorno necesita:
```yml
version: "3.1"
services:
  mediawiki:
    image: mediawiki
    restart: always
    ports:
      - 8080:80
    volumes:
      - /var/www/html/images
      - ./LocalSettings.php:/var/www/html/LocalSettings.php

  database:
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: my_wiki
      MYSQL_USER: wikiuser
      MYSQL_PASSWORD: example
      MYSQL_RANDOM_ROOT_PASSWORD: "yes"
```

Una vez hecho esto, ejecutamos el fichero de docker compose:
```shell
#### Ejecutamos el fichero de docker compose ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 5/Deploy$ docker-compose up -d
Creating deploy_database_1  ... done
Creating deploy_mediawiki_1 ... done

#### Comprobamos que estan ejecutandose ####
       Name                     Command               State          Ports        
----------------------------------------------------------------------------------
deploy_database_1    docker-entrypoint.sh mysqld      Up      3306/tcp            
deploy_mediawiki_1   docker-php-entrypoint apac ...   Up      0.0.0.0:8080->80/tcp
```

Asi, solo debemos de entrar en **localhost:8080** y seguir los pasos que nos muestra:

![mediawiki portada](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/docker/mediawiki.png)


Llegados al punto de descargar el ficher **LocalSettings.php** debemos de añadir una nueva linea a nuestro fichero de docker compose, por lo que debe de quedarnos de esta forma:
```yml
version: "3.1"
services:
  mediawiki:
    image: mediawiki
    restart: always
    ports:
      - 8080:80
    volumes:
      - /var/www/html/images
      - ./LocalSettings.php:/var/www/html/LocalSettings.php

  database:
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: my_wiki
      MYSQL_USER: wikiuser
      MYSQL_PASSWORD: example
      MYSQL_RANDOM_ROOT_PASSWORD: "yes"
```

Y volvemos a cargar el fichero:
```shell
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 5/Deploy$ docker-compose up -d
Recreating deploy_mediawiki_1 ... 
Recreating deploy_mediawiki_1 ... done
```

![mediawiki](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/docker/mediawiki-docker.png)