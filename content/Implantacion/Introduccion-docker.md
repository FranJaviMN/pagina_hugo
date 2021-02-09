---
title: "Introduccion Docker"
date: 2021-02-01T13:22:55+01:00
draft: true
---

# Intrducción a Docker

Docker es una tecnología de virtualización “ligera” cuyo elemento básico es la utilización de contenedores en vez de máquinas virtuales y cuyo objetivo principal es el despliegue de aplicaciones encapsuladas en dichos contenedores.

Docker está formado por varios componentes:

* Docker Engine: Es un demonio que corre sobre cualquier distribución de Linux y que expone una API externa para la gestión de imágenes y contenedores. Con ella podemos crear imágnenes, subirlas y bajarla de un registro de docker y ejecutar y gestionar contenedores.

* Docker Client: Es el cliente de línea de comandos (CLI) que nos permite gestionar el Docker Engine. El cliente docker se puede configurar para trabajar con con un Docker Engine local o remoto, permitiendo gestionar tanto nuestro entorno de desarrollo local, como nuestro entorno de producción.

* Docker Registry: La finalidad de este componente es almacenar las imágenes generadas por el Docker Engine. Puede estar instalada en un servidor independiente y es un componente fundamental, ya que nos permite distribuir nuestras aplicaciones. Es un proyecto open source que puede ser instalado gratuitamente en cualquier servidor, pero, como hemos comentado, el proyecto nos ofrece DockerHub.

## Indice

* [Introducción a Docker]()
* [Almacenamiento en Docker]()
* [Redes en Docker]()

# Instalando docker

En Debian 10 el paquete que debemos de instalar se llama **docker.io** que es la version de la comunidad y esta en repositorios de Debian:
```shell
#### Instalar docker.io ####
francisco@debian10:~$ sudo apt install docker.io
```

Si con un usuario que no sea root queremos usar docker, debemos de añadir a ese usuario al grupo de **docker**:
```shell
root@debian10:~# usermod -aG docker usuario
```

# Primer contenedor: Hola mundo

Para la primera prueba con docker vamos a usar una imagen que tenemos en los repositorio de DockerHub que se llama **hello-world** con la que vamos a comprobar si funciona docker correctamente, para ello usaremos el comando **run** que nos permite ejecutar un contenedor seguido del nombre de la imagen que vamos a usar, en este caso vamos a usar la imagen **hello-world**:
```shell
#### Ejecutamos el contenedor con la imagen hello-world ####
francisco@debian10:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:31b9c7d48790f0d8c50ab433d9c3b7e17666d6993084c002c2ff1ca09b96391d
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the DockerHub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
...
```

Como vemos, al no tener la imagen que le estamos diciendo a docker, este busca la imagen en los repositorios de DockerHub y la descarga y muestra el mensaje de bienvenida que es la consecuencia de crear y arrancar un contenedor basado en esa imagen.

Ahora podemos listar los contenedores que tenemos activos con el comando **ps** y si le añadimos la opcion **-a** nos muestra los contenedores que tenemos inactivos:
```shell
#### Listamos los contenedores que se estan ejecutando ####
francisco@debian10:~$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

#### Listamos los contenedores que no se estan ejecutando ####
francisco@debian10:~$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
9ba7ea0d8fc2        hello-world         "/hello"            13 minutes ago      Exited (0) 13 minutes ago                       objective_carson
```

Comprobamos que este contenedor no se está ejecutando. **Un contenedor ejecuta un proceso y cuando termina la ejecución, el contenedor se para**.

Si queremos eliminar un contenedor que tenemos creado, **es importante saber que debe de estar inactivo antes de poder eliminarlo**, de lo contrario no nos dejara eliminar dicho contenedor:
```shell
#### Paramos el contenedor si estuviera en ejecución ####
francisco@debian10:~$ docker stop nombre-contenedor

#### Lo borramos ####
francisco@debian10:~$ docker rm objective_carson
objective_carson
```

Como vemos si no le indicamos un nombre al contenedor, docker le da uno aleatorio.


# Contenedor interactivo

En este caso usamos la opción **-i** para abrir una sesión interactiva, **-t** nos permite crear un pseudo-terminal que nos va a permitir interaccionar con el contenedor, indicamos un nombre del contenedor con la opción **--name**, y la imagen que vamos a utilizar para crearlo, en este caso se va a llamar prueba-debian y va a ejecutar comoi comando **/bin/bash**
```shell
#### Iniciamos el contenedor interactivo ####
francisco@debian10:~$ docker run -it --name prueba-debian debian /bin/bash
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
b9a857cbf04d: Pull complete 
Digest: sha256:b16f66714660c4b3ea14d273ad8c35079b81b35d65d1e206072d226c7ff78299
Status: Downloaded newer image for debian:latest
root@f7b0f807a1a2:/# 

#### Salimos del contenedor y vemos que se ha parado ####
root@f7b0f807a1a2:/# exit
francisco@debian10:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
f7b0f807a1a2        debian              "/bin/bash"         45 seconds ago      Exited (0) 3 seconds ago                       prueba-debian

#### Lo volvemos a iniciar ####
francisco@debian10:~$ docker start prueba-debian

#### Le podemos pasar comando con la opcion exec ####
francisco@debian10:~$ docker exec prueba-debian ls -la
total 72
drwxr-xr-x   1 root root 4096 Feb  1 12:18 .
drwxr-xr-x   1 root root 4096 Feb  1 12:18 ..
-rwxr-xr-x   1 root root    0 Feb  1 12:18 .dockerenv
drwxr-xr-x   2 root root 4096 Jan 11 00:00 bin
drwxr-xr-x   2 root root 4096 Nov 22 12:37 boot
drwxr-xr-x   5 root root  360 Feb  1 12:20 dev
drwxr-xr-x   1 root root 4096 Feb  1 12:18 etc
drwxr-xr-x   2 root root 4096 Nov 22 12:37 home
drwxr-xr-x   7 root root 4096 Jan 11 00:00 lib
drwxr-xr-x   2 root root 4096 Jan 11 00:00 lib64
drwxr-xr-x   2 root root 4096 Jan 11 00:00 media
drwxr-xr-x   2 root root 4096 Jan 11 00:00 mnt
drwxr-xr-x   2 root root 4096 Jan 11 00:00 opt
dr-xr-xr-x 221 root root    0 Feb  1 12:20 proc
drwx------   2 root root 4096 Jan 11 00:00 root
...

#### Vemos la informacion del contenedor ####
francisco@debian10:~$ docker inspect prueba-debian
[
    {
        "Id": "f7b0f807a1a2e02dbc614e971b32f26cd93f396a2d87987019a658a6ab16a602",
        "Created": "2021-02-01T12:18:33.172823479Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
...
```


# Ejercicios

1. Instala docker en una máquina y configuralo para que se pueda usar con un usuario sin privilegios.

Para ello seguimos los pasos de esta introduccion e instalamos docker en nuestra maquina personal desde los repositorios, en mi caso usando una maquina Debian 10:
```shell
#### Instalamos el paquete llamado docker.io #####
root@debian10:~# apt update && apt upgrade

root@debian10:~# apt install docker.io

#### Añadimos nuestro usuario al grupo de docker ####
root@debian10:~# usermod -aG docker francisco

#### Comprobamos que esta en el grupo docker ####
francisco@debian10:~$ groups 
.   .   .  docker

#### Vemos la version de docker ####
francisco@debian10:~$ docker --version
Docker version 18.09.1, build 4c52b90
```

Debemos de tener en cuenta que a la hora de añadir a nuestro usuario al grupo docker, debemos de reiniciar la sesion para que el cambio pueda verse.

2. Crea un contenedor interactivo desde una imagen debian. Instala un paquete (por ejemplo nano). Sal de la terminal, ¿sigue el contenedor corriendo? ¿Por qué?. Vuelve a iniciar el contenedor y accede de nuevo a él de forma interactiva. ¿Sigue instalado el nano?. Sal del contenedor, y borralo. Crea un nuevo contenedor interactivo desde la misma imagen. ¿Tiene el nano instalado?

Para ello vamos a usar la imagen que tenemos en **DockerHub** de debian, por lo que primero vamos a descargar la imagen, para ello usaremos el comando **docker run nombre-imagen** y este comando lo que hara es descargar la imagen y ejecutar el contenedor al mismo tiempo. Como tambien queremos que nuestro contenedor sea interactivo, a este comando debemos de añadirle las opciones **--it** donde le indicamos que va a ser una consola interactiva:
```shell
#### Ejecutamos el contenedor interactivo de debian ####
francisco@debian10:~$ docker run -it --name prueba-debian debian /bin/bash
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
b9a857cbf04d: Pull complete 
Digest: sha256:b16f66714660c4b3ea14d273ad8c35079b81b35d65d1e206072d226c7ff78299
Status: Downloaded newer image for debian:latest
root@c26222c4e600:/# 
```

Como vemos ya estamos dentro del contenedor con debian, ahora, siguiendo con el ejercicio, vamos a instalar el paquete nano en nuestro contenedor, cabe destacar que el contenedor es muy basico y solo trae lo necesario para poder funcionar, por lo que la gran mayoria de paquetes no estaran disponibles.
```shell
#### Instalamos nano ####
root@c26222c4e600:/# apt update && apt install nano
```

Ahora vamos a salir de la terminal para ver si el contenedor sigue corriendo despues de salir y como podemos ver el contenedor ya esta parado:
```shell
francisco@debian10:~$ docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Esto se debe a que el contenedor solo debia de ejecutar la bash interactiva para que nosotros entremos y hagamos lo que necesitemos, pero a la hora de salir ya deja de ser interactivo y el proceso de bash que estaba ejecutando ya no lo ejecuta, por lo que el contenedor se para:
```shell
francisco@debian10:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
c26222c4e600        debian              "/bin/bash"         5 minutes ago       Exited (0) 2 minutes ago                       prueba-debian
```

Bien, ahora vamos a iniciar el contenedor y vamos a entrar de forma interactiva a este para poder ver si sigue instalado el programa nano, por lo que vaos a ejecutar el siguiente comando:
```shell
#### Iniciamos el contenedor ####
francisco@debian10:~$ docker start prueba-debian
prueba-debian

francisco@debian10:~$ docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c26222c4e600        debian              "/bin/bash"         7 minutes ago       Up 3 seconds                            prueba-debian

#### Entramos de forma interactiva al contenedor ####
francisco@debian10:~$ docker attach prueba-debian
root@c26222c4e600:/#

#### Comprobamos que el programa nano esta instalado ####
root@c26222c4e600:/# whereis nano
nano: /bin/nano /usr/share/nano /usr/share/man/man1/nano.1.gz /usr/share/info/nano.info.gz
```

Como vemos el comando nano si esta instalado aunque hayamos parado el contenedor pero ahora vamos a comprobar que ocurre cuando lo borramos y volvemos a crear:
```shell
#### Salimos del contenedor y lo borramos ####
francisco@debian10:~$ docker rm prueba-debian
prueba-debian
francisco@debian10:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

#### Creamos de nuevo el contenedor con la misma imagen ####
francisco@debian10:~$ docker run -it --name prueba-debian2 debian
root@a278a0e80ee6:/# 

#### Comprobamos que el contenedor no contiene el programa nano ####
root@a278a0e80ee6:/# whereis nano
nano:
```

Esto se debe a que los datos que estan en un contenedor solo estan en ese contenedor por lo que al borrar ese contenedor todo lo que haya en su interior se borra y este contenido no se guarda, por eso al volver a usar la imagen de debian que habiamos descargado esta no contiene nada.

3. Crea un contenedor demonio con un servidor nginx, usando la imagen oficial de nginx. Al crear el contenedor, ¿has tenido que indicar algún comando para que lo ejecute? Accede al navegador web y comprueba que el servidor esta funcionando. Muestra los logs del contenedor.

Bien, ahora vamos a usar la imagen de **nginx** para poder tener un servidor web con nginx, para ello vamos a usar la imagen de docker llamada **nginx**. En este comando para iniciar el servidor web de nginx debemos de indicarle la opción **-p** con la que mapeamos un puerto del equipo donde tenemos instalado el docker, con un puerto del contenedor. Para ello vamos a usar el puerto 8080 de nuestra maquina, y nginx usara el puerto 80 para conectar con nuestro equipo y tambien usaremos el comando **-d** para que se ejecute en segundo plano:
```shell
#### Ejecutamos el contenedor con las opciones necesarias ####
francisco@debian10:~$ sudo docker run -d --name prueba-nginx -p 8080:80 nginx

#### Comprobamos el contenedor ####
francisco@debian10:~$ sudo docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
c0bf146c82b6        nginx               "/docker-entrypoint.…"   28 seconds ago      Up 24 seconds       0.0.0.0:8080->80/tcp   prueba-nginx

#### Comprobamos los logs de ese contenedor ####
francisco@debian10:~$ sudo docker logs prueba-nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
172.17.0.1 - - [01/Feb/2021:08:48:03 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0" "-"
2021/02/01 08:48:03 [error] 28#28: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080"
172.17.0.1 - - [01/Feb/2021:08:48:03 +0000] "GET /favicon.ico HTTP/1.1" 404 153 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0" "-"
```

Asi, al entrar en http://localhost:8080 nos debe de aparecer el siguiente contenido:

![docker nginx](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/docker/nginx-docker.png)

4. Crea un contenedor con la aplicación Nextcloud, mirando la documentación en DockerHub, para personalizar el nombre de la base de datos sqlite que va autilizar.

Para ello nos debemos de ir a la pagina de DockerHub y ahi vamos a buscar la imagen de **nextcloud**, para ell vamos a seguir el mismo proceso que hemos seguido en el anterior ejercicio y vamos a crear un contenedor con nextcloud:
```shell
#### Creamos el contenedor con nextcloud en el puerto 8081 ####
francisco@debian10:~$ docker run -d --name prueba-nextcloud -p 8081:80 nextcloud

#### Comprobamos el contenedor corriendo ####
francisco@debian10:~$ docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
b2dc78dc82ab        nextcloud           "/entrypoint.sh apac…"   41 seconds ago      Up 34 seconds       0.0.0.0:8081->80/tcp   prueba-nextcloud

#### Vemos los logs ####
francisco@debian10:~$ docker logs prueba-nextcloud
Initializing nextcloud 20.0.6.1 ...
Initializing finished
New nextcloud instance
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
[Mon Feb 01 09:02:22.356402 2021] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.38 (Debian) PHP/7.4.14 configured -- resuming normal operations
[Mon Feb 01 09:02:22.356848 2021] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
```

Asi, si entramos en http://localhost:8081 podemos ver que tenemos una aplicacion de nextcloud corriendo:

![docker nextcloud](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/docker/nextcloud-docker.png)