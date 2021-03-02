---
title: "Tarea4 Docker Php"
date: 2021-03-02T10:33:40+01:00
draft: true
---
**Repositorio git: https://github.com/FranJaviMN/Docker-php/tree/main/Tarea%204**

# Tarea 4: Ejecución de un CMS en dockerPermalink

* A partir de una imagen base (que no sea una imagen con el CMS), genera una imagen que despliegue un CMS PHP (que no sea wordpress).

* Crea los volúmenes necesarios para que la información que se guarda sea persistente.

Para ello vamos a usar el CMS de **Drupal** por lo que vamos a necesitar de nuevo dos directorios, uno llamado **Deploy** donde tendremos nuestro fichero docker-compose y otro directorio llamado **Build**

## Creamos nuestra imagen

Para ello vamos a usar una imagen **debian** en la cual vamos a instalar tanto apache2 como php. Ademas debemos de llevar el fichero descargado a nuestro contenedor donde vamos a tener nuestro CMS:
```dockerfile
FROM debian

RUN apt-get update && apt-get install -y apache2 libapache2-mod-php7.3 php7.3 php7.3-mysql php-xml php-gd php-mysql php-mbstring && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN rm /var/www/html/index.html && a2enmod rewrite

EXPOSE 80

COPY ./drupal-9.1.4 /var/www/html

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Como vemos ya tenemos nuestro fichero dockerfile vamos a generar la imagen que vamos a usar en nuestro ejercicio, para ello usamos el siguiente comando de docker:
```shell
#### Creamos la nueva imagen ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 4/Build$ docker build -t franjavimn/drupal-tarea4:v1 .

#### Listamos las imagenes para ver si se ha creado ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 4/Build$ docker image list
REPOSITORY                    TAG                 IMAGE ID            CREATED              SIZE
franjavimn/drupal-tarea4      v1                  89aa79d074c2        About a minute ago   320MB
```

## Creacion de docker-compose

Ahora que ya tenemos nuestra imagen ya creada vamos a proceder a crear nuestro nuevo fichero **docker-compose.yml** en el que vamos a definir nuestra base de datos la cual va a ser una base de datos mariadb y vamos a definir la creación del contenedor con la imagen que hemos generado anteriormente:
```yml
version: "3.1"

services:
  db:
    container_name: servidor_mysql-drupal
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: drupal
      MYSQL_USER: drupal
      MYSQL_PASSWORD: drupal
      MYSQL_ROOT_PASSWORD: franciscojavier
    volumes:
      - /opt/mysql_drupal:/var/lib/mysql

  drupal:
    container_name: drupal
    image: franjavimn/drupal-tarea4:v1
    restart: always
    ports:
      - 8084:80
    volumes:
      - /opt/drupal:/var/log/apache2
```

Una vez hecho esto solo debemos de ejecutar nuestro escenario ya tendriamos nuestro CMS de drupal desplegado, solo faltaria instalarlo y estaria listo:
```shell
#### Ejecutamos el fichero docker-compose ####
francisco@debian10:~/Documentos/Implantacion/Docker/Docker-php/Tarea 4/Deploy$ docker-compose up -d

#### Comprobamos que esta iniciado ####
        Name                       Command               State          Ports        
-------------------------------------------------------------------------------------
drupal                  /usr/sbin/apache2ctl -D FO ...   Up      0.0.0.0:8084->80/tcp
servidor_mysql-drupal   docker-entrypoint.sh mysqld      Up      3306/tcp            
```

Una vez hecho esto solo debemos de irnos a nuestro navegador y solo debemos de entrar en **localhost:8084**:

![instalador de drupal](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/docker/instalar-drupal-docker.png)

![drupal en docker](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/docker/Drupal-docker.png)