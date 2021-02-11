---
title: "Docker Compose"
date: 2021-02-11T14:17:18+01:00
draft: true
---

# Docker Compose

La herramienta Docker Compose, que nos permitirá definir y ejecutar múltiples aplicaciones utilizando contenedores de software.

Con Compose utilizaremos ficheros en formato YAML, que nos servirán para definir la configuración de la aplicación en cuestión. De esta manera podemos, con un solo comando, crear e iniciar los servicios configurados en estos ficheros.

Vamos a definir el escenario en un fichero llamado docker-compose.yaml y vamos a gestionar el ciclo de vida de la aplicaciones y de todos los contenedores que necesitamos con la utilidad docker-compose.

## Ventajas de uso de Docker Compose

* Hacer todo de manera declarativa para que no tenga que repetir todo el proceso cada vez que construyo el escenario.

* Poner en funcionamiento todos los contenedores que necesita mi aplicación de una sola vez y debidamente configurados.

* Garantizar que los contenedores se arrancan en el orden adecuado. Por ejemplo: Mi aplicación no podrá funcionar debidamente hasta que no esté el servidor de bases de datos funcionando en marcha.

* Asegurarnos de que hay comunicación entre los contenedores que pertenecen a la aplicación.

## Instalando Docker Compose

La instalación es simple y podemos hacerlo tanto desde los repositorios de nuestra maquina, en mi caso una maquina Debian 10, o bien podemos hacer con el gesto de paquete **pip** en un entorno **virtual python**
```shell
#### Instalacion desde repositorios ####
francisco@debian10:~$ sudo apt install docker-compose

#### Instalacion desde un entorno virtual ####
francisco@debian10:~$ python3 -m venv docker-compose

francisco@debian10:~$ source docker-compose/bin/activate
(docker-compose) francisco@debian10:~$ pip install docker-compose
```

## Estructura del fichero docker-compose.yml

En el fichero docker-compose.yml vamos a definir el escenario. El programa docker-compose se debe ejecutar en el directorio donde este ese fichero. Lo mejor que podemos hacer es fijarnos en otros ficheros **docker-compose.yml** que sean sencillos para saber la estrutuca que debe de tener, para ello y como ejemplo, vamos a usar el fichero **docker-compose** de ejemplo que tiene wordpress en DockerHub:
```yaml
version: '3.1' #La version que estemos usando de docker-compose

services: #Definimos los contenedores que vamos a levantar

  wordpress:
    #Nomber del contenedor
    container_name: servidor_wp

    #Imagen que vamos a usar
    image: wordpress

    #Indicamos la política de reinicio del contenedor si por cualquier condición se para
    restart: always

    #Definimos las variables de entorno que nos ofrecen en la documentacion de wordpress
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: user_wp
      WORDPRESS_DB_PASSWORD: asdasd
      WORDPRESS_DB_NAME: bd_wp

    #Puertos que vamos a exponer de nuestra maquina y el contenedor  
    ports:
      - 80:80

    #Volumenes que vamos a usar, en este caso del tipo volumenes de docker  
    volumes:
      - /opt/wordpress:/var/www/html/wp-content
    #Si queremos que se de tipo bind mount
    #volumes:
    #  - /home/usuario/wordpress:/var/www/html/wp-content

  db:
    container_name: servidor_mysql
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: bd_wp
      MYSQL_USER: user_wp
      MYSQL_PASSWORD: asdasd
      MYSQL_ROOT_PASSWORD: asdasd
    volumes:
      - /opt/mysql_wp:/var/lib/mysql
```

Podemos encontrar mas parametros del fichero **docker-compose.yml** en [la documentación oficial](https://docs.docker.com/compose/compose-file/compose-file-v3/)

Cuando creamos un escenario con docker-compose se crea una nueva red definida por el usuario docker donde se conectan los contenedores, por lo tanto, se pueden tenemos resolución por dns que resuelve tanto el nombre del contenedor (por ejemplo, servidor_mysql) como el alias (por ejemplo, db).

Una vez tengamos este fichero creado solo debemos de ejecutar el escenario mediante el comando **docker-compose up** y le añadimos la opcion **-d** para que sea como un demonio, para ello lo hacemos de la siguiente forma:
```shell
#### Levantamos el escenario ####
francisco@debian10:~/Documentos/Implantacion/Docker/docker-compose/wordpress$ docker-compose up -d
...
Creating servidor_mysql ... done
Creating servidor_wp    ... done
```

* Para listar los contenedores podemos usar el comando **docker-compose ps**:
```shell
francisco@debian10:~/Documentos/Implantacion/Docker/docker-compose/wordpress$ docker-compose ps
     Name                   Command               State          Ports        
------------------------------------------------------------------------------
servidor_mysql   docker-entrypoint.sh mysqld      Up      3306/tcp            
servidor_wp      docker-entrypoint.sh apach ...   Up      0.0.0.0:8080->80/tcp
```

* Si queremos para los contenedores debemos de usar el comando **docker-compose stop**:
```shell
francisco@debian10:~/Documentos/Implantacion/Docker/docker-compose/wordpress$ docker-compose stop
Stopping servidor_wp    ... done
Stopping servidor_mysql ... done
```

* Si esta vez queremos borrar los contenedores, debemos de tener en cuenta que antes deben de estar parados, una vez parados usamos el comando **docker-compose rm**:
```shell
francisco@debian10:~/Documentos/Implantacion/Docker/docker-compose/wordpress$ docker-compose rm
Going to remove servidor_wp, servidor_mysql
Are you sure? [yN] y
Removing servidor_wp    ... done
Removing servidor_mysql ... done
```

## Comando docker-compose

Una vez hemos creado el archivo **docker-compose.yml** tenemos que empezar a trabajar con él, es decir a crear los contenedores que describe su contenido.

Esto lo haremos mediante el ejecutable *docker-compose*. **Es importante destacar que debemos invocarla desde el directorio en el que se encuentra el fichero docker-compose.yml**.

Los subcomandos más usados son:

* **docker-compose up**: Crear los contenedores (servicios) que están descritos en el docker-compose.yml

* **docker-compose up -d**: Crear en modo detach los contenedores (servicios) que están descritos en el docker-compose.yml. E
significa que no muestran mensajes de log en el terminal y que se nos vuelve a mostrar un prompt

* **docker-compose stop**: Detiene los contenedores que previamente se han lanzado con docker-compose up

* **docker-compose run**: Inicia los contenedores descritos en el docker-compose.yml que estén parados

* **docker-compose rm**: Borra los contenedores parados del escenario. Con las opción -f elimina también los contenedores 
ejecución

* **docker-compose pause**: Pausa los contenedores que previamente se han lanzado con docker-compose up

* **docker-compose unpause**: Reanuda los contenedores que previamente se han pausado

* **docker-compose restart**: Reinicia los contenedores. Orden ideal para reiniciar servicios con nuevas configuraciones

* **docker-compose down**: Para los contenedores, los borra y también borra las redes que se han creado con docker-compose up (
caso de haberse creado)

* **docker-compose down -v**: Para los contenedores y borra contenedores, redes y volúmene

* **docker-compose logs servicio1**: Muestra los logs del servicio llamado servicio1 que estaba descrito en el docker-compose.yml

* **docker-compose exec servicio1 /bin/bash**: Ejecuta una orden, en este caso /bin/bash en un contenedor llamado servicio1 q
estaba descrito en el docker-compose.ym

* **docker-compose build**: Ejecuta, si está indicado, el proceso de construcción de una imagen que va a ser usado en 
docker-compose.yml a partir de los ficheros Dockerfile que se indican

* **docker-compose top**: Muestra los procesos que están ejecutándose en cada uno de los contenedores de los servicios.





