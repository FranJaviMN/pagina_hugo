---
title: "Practica Servidor Nginx"
date: 2020-11-04T17:42:53+01:00
draft: true
---

# Práctica: Servidor Web Nginx

**Nginx** es un famoso software de servidor web de código abierto. En su versión inicial, funcionaba en servidores web HTTP. Sin embargo, hoy en día también sirve como proxy inverso, balanceador de carga HTTP y proxy de correo electrónico para IMAP, POP3 y SMTP.

Debido a su excelente capacidad para manejar muchas conexiones y a su velocidad, muchos sitios web de alto tráfico usan el servicio de NGINX. Algunos de estos gigantes del internet son Google, Netflix, Adobe, Cloudflare, WordPress.com y muchos más.

Según W3Techs, en términos de números aproximados, Apache es el servidor web más popular que existe y se usa en un 43.6% (en comparación con el 47% en 2018) de todos los sitios web con un servidor web conocido. Nginx queda en segundo lugar cerca del 41.8%.

Netcraft realizó una [encuesta](https://news.netcraft.com/archives/2020/10/21/october-2020-web-server-survey.html) en 233 millones de dominios y encontró que el uso de Apache es de 31.54% y el uso de Nginx es 26.20%.

## Tarea 1

Crea una máquina del cloud con una red pública. Añade la clave pública del profesor a la máquina. Instala el servidor web nginx en la máquina. Modifica la página index.html que viene por defecto y accede a ella desde un navegador.

* Entrega la ip flotante de la máquina para que el profesor pueda acceder a ella.
* Entrega una captura de pantalla accediendo a ella.

Para ello, previamente debemos de crear una instancia en nuestro servicio de cloud que tenemos en nuestro instituto, para ello hemos elegido una maquina con un sistema operativo **Debian Buster 10.6**, de *flavor* hemos escogido **m1.mini** y como clave hemos usado la clave publica que anteriormente hemos generado en nuestro ordenador y que hemos copiado a nuestro servicio cloud. Con todo ello le hemos dado como nombre **practica-nginx**. Para poder entrar a dicha instancia usaremos la ip flotante que le hemos asociado, en este caso la ip es **172.22.200.118**.
```shell
#### Entrando en la maquina via ssh con nuestra clave publica ####
francisco@debian10:~/.ssh$ ssh -i id_rsa debian@172.22.200.118
The authenticity of host '172.22.200.118 (172.22.200.118)' cant be established.
ECDSA key fingerprint is SHA256:X+YOR4W1fJ5XZ7KRBBdauEpUHxCbYrRX1kOfBZFHIuk.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.22.200.118' (ECDSA) to the list of known hosts.
Enter passphrase for key 'id_rsa': 
Linux practica-nginx 4.19.0-11-cloud-amd64 #1 SMP Debian 4.19.146-1 (2020-09-17) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

debian@practica-nginx:~$ 
```

## Virtual Hosting

Queremos que nuestro servidor web ofrezca dos sitios web, teniendo en cuenta lo siguiente:

1. Cada sitio web tendrá nombres distintos.

2. Cada sitio web compartirán la misma dirección IP y el mismo puerto (80).

Los dos sitios web tendrán las siguientes características:

* El nombre de dominio del primero será www.iesgn.org, su directorio base será /srv/www/iesgn y contendrá una página llamada index.html, donde sólo se verá una bienvenida a la página del Instituto Gonzalo Nazareno.

* En el segundo sitio vamos a crear una página donde se pondrán noticias por parte de los departamento, el nombre de este sitio será departamentos.iesgn.org, y su directorio base será /srv/www/departamentos. En este sitio sólo tendremos una página inicial index.html, dando la bienvenida a la página de los departamentos del instituto.

## Tarea 2

Configura la resolución estática en los clientes y muestra el acceso a cada una de las páginas.

Para ello debemos de instalar previamente nuestro servidor **nginx**, para ello debemos de instalar el siguiente paquete:
```shell
#### Instalamos el paquete de nginx ####
debian@practica-nginx:~$ sudo apt install nginx
```

Siguiendo con la filosofia que teniamos en **apache2**, podemos encontrar los sitios disponibles y activos en los mismos directorios donde los encontrabamos en **apache2**, es decir:
```shell
#### Directorio de sitios disponibles ####
/etc/nginx/sites-availables/

#### Directorio de sitios activos ####
/etc/nginx/sites-enabled/
```

Por lo tanto, si necesitamos tener nuestras dos paginas debemos de generar primero los dos virtualhost, para ello vamos a realizar la copia del fichero *default* para crear nuestros dos virtualhost:
```shell
#### Copiamos el fichero default ####
debian@practica-nginx:/etc/nginx/sites-available$ sudo cp default ./iesgn.conf

debian@practica-nginx:/etc/nginx/sites-available$ sudo cp default ./departamentos.conf
```

Una vez tengamos copiados los ficheros debemos de tener el siguiente contenido en los fichero que hemos creado:
```shell
#### Fichero de iesgn.conf ####
...
server {
    listen 80;

    ...
    root /srv/www/iesgn;
    index index.html index.htm index.nginx-debian.html;
    server_name www.iesgn.org;
    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }
...
}

#### Fichero de departamentos.conf ####
...
server {
    listen 80;

    ...
    root /srv/www/departamentos;
    index index.html index.htm index.nginx-debian.html;
    server_name departamentos.iesgn.org;
    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }
...
}
```

Una vez tengamos ya nuestros fichero debemos de crear un enlace simbolico para hacer que estos sitios esten activos, para ello usamos el siguiente comando:
```shell
#### Fichero departamentos.conf ####
debian@practica-nginx:/etc/nginx/sites-available$ sudo ln -s /etc/nginx/sites-available/departamentos.conf /etc/nginx/sites-enabled/

#### Fichero iegn.conf ####
debian@practica-nginx:/etc/nginx/sites-available$ sudo ln -s /etc/nginx/sites-available/iesgn.conf /etc/nginx/sites-enabled/
```

Ahora debemos de crear en el directorio **/srv/www** las dos carpetas llamadas iesgn y departamentos y dentro de ellas un fichero index.html en el cual tendremos un mensaje de bienvenida:
```shell
#### Creamos las carpetas ####
debian@practica-nginx:~$ sudo mkdir /srv/www/{iesgn,departamentos}

#### Creamos los dos fichero index.html ####
debian@practica-nginx:~$ sudo touch /srv/www/departamentos/index.html
debian@practica-nginx:~$ sudo touch /srv/www/iesgn/index.html

#### Cambiamos los permisos a www-data ####
debian@practica-nginx:~$ sudo chown -R www-data:www-data /srv/www
```

Entonces ya tendremos nuestros dos sitios, si queremos verificar que todo esta correcto podemos ejecutar el siguiente comando:
```shell
#### Verificamos que esta correcto todo ####
debian@practica-nginx:~$ sudo nginx -t

#### Reiniciamos el servicio ####
debian@practica-nginx:~$ sudo systemctl restart nginx.service
```

Una vez hecho todo solo debemos de entrar en nuestro **/etc/hosts** de la maquina clientes, es decir la nuestra, y poner las siguientes lineas:
```shell
172.22.200.118  www.iesgn.org
172.22.200.118  departamentos.iesgn.org
```

Y si nos dirigimos a las direcciones en nuestro navegador veremos las siguientes paginas:

![pagina iesgn](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/iesgn-nginx.png)

![pagina departamentos](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/departamentos-nginx.png)


# Mapeo de URL

Cambia la configuración del sitio web www.iesgn.org para que se comporte de la siguiente forma:

## Tarea 3

Cuando se entre a la dirección www.iesgn.org se redireccionará automáticamente a www.iesgn.org/principal, donde se mostrará el mensaje de bienvenida. En el directorio principal no se permite ver la lista de los ficheros, no se permite que se siga los enlaces simbólicos y no se permite negociación de contenido. Muestra al profesor el funcionamiento.

Lo primero que vamos a hacer es crear el directorio **/principal** en el directorio **/srv/www/iesgn**. Una vez la tengamos creada vamos a añadir un fichero index.html dando la bienvenida a principal:
```shell
#### Creamos el directorio /principal ####
debian@practica-nginx:~$ sudo mkdir /srv/www/iesgn/principal

#### Creamos el fichero index.html con el saludo ####
debian@practica-nginx:~$ sudo touch /srv/www/iesgn/principal/index.html

#### Cambiamos el propietario ####
debian@practica-nginx:~$ sudo chown -R www-data:www-data /srv/www
```

Ahora que ya tenemos los elementos anteriores debemos de crear el alias, para ello debemos de añadir la siguiente liena a nuestro fichero **iesgn.conf**:
```shell
#### Interior del fichero iesgn.conf
rewrite ^/$ principal permanent;

#### Vemos si hay algun error ####
debian@practica-nginx:~$ sudo nginx -t

#### Reiniciamos el servicio ####
debian@practica-nginx:~$ sudo systemctl restart nginx.service
```

De esta forma, si entramos en **www.iesgn.org** nos mostrara **www.iesgn.org/principal/**:

![redireccion](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/redirect-nginx.png)

## Tarea 4

Si accedes a la página www.iesgn.org/principal/documentos se visualizarán los documentos que hay en /srv/doc. Por lo tanto se permitirá el listado de fichero y el seguimiento de enlaces simbólicos siempre que sean a ficheros o directorios cuyo dueño sea el usuario. Muestra al profesor el funcionamiento.

Lo primero creamos el directorio **srv/doc** en el cual vamos a añadir unos cuantos ficheros de prueba, para ello usamos los siguinetes comandos:
```shell
#### Creamos el directorio doc ####
debian@practica-nginx:~$ sudo mkdir /srv/doc

#### Creamos los fichero de prueba ####
debian@practica-nginx:~$ sudo touch /srv/doc/{hola.txt,esto.txt,es_una_prueba.txt}

#### Cambiamos el propietario de los nuevos ficheros ####
debian@practica-nginx:~$ sudo chown -R www-data:www-data /srv/
```

Ahora que ya tenemos nuestros fichero de prueba vamos a crear nuestro nuevo alias para que nos muestre todo los fichero de **/srv/doc**:
```shell
#### Contenido del fichero iesgn.conf ####
location /principal/documentos {
    alias /srv/doc/;
    autoindex on;
}

#### Vemos si hay algun error ####
debian@practica-nginx:~$ sudo nginx -t

#### Reiniciamos el servicio ####
debian@practica-nginx:~$ sudo systemctl restart nginx.service
```

De esta forma, si entramos en **www.iesgn.org/principal/documentos** veremos los siguiente:

![documentos](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/documentos-nginx.png)

## Tarea 5

En todo el host virtual se debe redefinir los mensajes de error de objeto no encontrado y no permitido. Para el ello se crearan dos ficheros html dentro del directorio error. Entrega las modificaciones necesarias en la configuración y una comprobación del buen funcionamiento.

Para ello vamos a crear el directorio error y dentro de el vamos a tener dos ficheros de error, para ello usamos el siguiente comando:
```shell
#### Creamos el directorio de error ####
debian@practica-nginx:~$ sudo mkdir /srv/www/iesgn/error

#### Creamos los ficheros de error con un mensaje de error ####
debian@practica-nginx:~$ sudo touch /srv/www/iesgn/error/{404.html,403.html}

#### Cambiamos el propiertario de los ficheros ####
debian@practica-nginx:~$ sudo chown -R www-data:www-data /srv/

#### Añadimos la linea donde indicamos el error personalizado ####
error_page 404 /error/404.html; -> Error personalizado para 404
error_page 403 /error/403.html; -> Error personalizado para 403
```

![error 404](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/error404-nginx.png)



# Autentificación, Autorización, y Control de Acceso

## Tarea 6

Añade al escenario otra máquina conectada por una red interna al servidor. A la URL departamentos.iesgn.org/intranet sólo se debe tener acceso desde el cliente de la red local, y no se pueda acceder desde la anfitriona por la red pública. A la URL departamentos.iesgn.org/internet, sin embargo, sólo se debe tener acceso desde la anfitriona por la red pública, y no desde la red local.

Para ello hemos creado una maquina llamada **cliente-nginx** con la ip privada 10.0.0.11.

Lo primero que vamos a hacer va a ser crear los dos directorios llamados internet e intranet, para ello usamos el siguiente comando:
```shell
#### Creamos los directorios ####
debian@practica-nginx:~$ sudo mkdir /srv/www/iesgn/internet
debian@practica-nginx:~$ sudo mkdir /srv/www/iesgn/intranet

#### Creamos los fichero index.html con un mensaje de bienvenida ####
debian@practica-nginx:~$ sudo nano /srv/www/iesgn/internet/index.html
debian@practica-nginx:~$ sudo nano /srv/www/iesgn/intranet/index.html

#### Cambiamos los propietarios de los ficheros ####
debian@practica-nginx:~$ sudo chown -R www-data:www-data /srv/
```

Entonces en nuestro fichero de iesgn.conf debemos de tener las siguientes lineas:
```shell
#### COntenido del fichero iesgn.conf ####
location /internet {
    allow 172.22.200.118;
    autoindex on;
}

location /intranet {
    allow 10.0.0.11;
    deny all;
    autoindex on;
}

#### Reiniciamos el servicio ####
debian@practica-nginx:~$ sudo systemctl restart nginx.service
```

Entonces al intentar entrar en **/intranet** no nos dejara:

![intranet](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/intranet-nginx.png)

Mientras que si queremos entrar en **/internet** si nos dejara:

![internet](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/internet-nginx.png)

## Tarea 7

Autentificación básica. Limita el acceso a la URL departamentos.iesgn.org/secreto. Comprueba las cabeceras de los mensajes HTTP que se intercambian entre el servidor y el cliente.

Para ello creamos el directorio **/secreto** en el cual tendremos un mensaje de bienvenida y este estara restringido por un usuario y una contraseña, primero creamos el directorio y el fichero:
```shell
#### Creamos el directorio ####
debian@practica-nginx:~$ sudo mkdir /srv/www/iesgn/secreto

#### Creamos el fichero con un mensaje ####
debian@practica-nginx:~$ sudo nano /srv/www/iesgn/secreto/index.html

#### Le cambiamos el propietario a nuestro nuevo directorio ####
debian@practica-nginx:~$ sudo chown -R www-data:www-data /srv/
```

Una vez tengamos lo anterior debemos de crear nuestro fichero donde vamos a guardar el usuario y la contraseña de este, para ello vamos a usar una utilidad llamada htpasswd que esta dentro del paquete que debemos instalar llamado **apache2-utils**:
```shell
#### Primero instalamos el paquete ####
debian@practica-nginx:~$ sudo apt install apache2-utils

#### Creamos el fichero con el usuario fran ####
debian@practica-nginx:~$ sudo htpasswd -c /srv/passwd.txt fran
New password: ***
Re-type new password:*** 
Adding password for user fran

Nota: Si queremos añadir mas usuarios debemos de quitar la opcion -c

#### Cambiamos el propietario del fichero ####
debian@practica-nginx:~$ sudo chown -R www-data:www-data /srv/
```

De esta forma ya tendremos todo listo, ahora solo debemos de añadir la siguiente linea a nuestro fichero de iesgn.conf:
```shell
#### Linea que tenemos que añadir ####
location /secreto {
    auth_basic "Area secreta";
    auth_basic_user_file /srv/passwd.txt;
}

#### Vemos que todo esta bien ####
debian@practica-nginx:~$ sudo nginx -t

#### Reiniciamos el servicio ####
debian@practica-nginx:~$ sudo systemctl restart nginx.service
```

Por lo que si queremos entrar en **/secreto** nos saldra la siguiente ventana:

![secreto](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/basic-nginx.png)

## Tarea 8

Vamos a combinar el control de acceso (tarea 6) y la autentificación (tarea 7), y vamos a configurar el virtual host para que se comporte de la siguiente manera: el acceso a la URL departamentos.iesgn.org/secreto se hace forma directa desde la intranet, desde la red pública te pide la autentificación. Muestra el resultado al profesor.

Para poder hacer estas pruebas debemos de usar lo que hemos hecho en las tareas 6 y 7, por lo que no veo necesario tener que volver a explicar todo los pasos ya que los hemos dado y solo debemos de cambiar la url por **departamentos.iesgn.org**:

1. Creamos el directorio secreto:
```shell
debian@practica-nginx:~$ sudo mkdir /srv/www/departamentos/secreto
```

2. Creamos el fichero de bienvenida:
```shell
debian@practica-nginx:~$ sudo nano /srv/www/departamentos/secreto/index.html
```

3. Creamos el usuario con htpasswd:
```shell
debian@practica-nginx:~$ sudo htpasswd -c /srv/www/departamentos/passwd.txt fran
New password: 
Re-type new password: 
Adding password for user fran
```

4. Configuramos el fichero de **departamentos.conf** con las siguientes lineas:
```shell
#### Linea a añadir ####
location /secreto {
    auth_basic "Area secreta";
    auth_basic_user_file /srv/passwd.txt;
    allow 10.0.0.11;
    satisfy any;
}
```
Entonces, si desde nuestro navegador entramos en secretos nos pedira las credenciales:

![credenciales](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/secreto-departamento.png)

Pero si entramos desde nuestro cliente no nos pedira las credenciales:

![sin credenciales](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/nginx/secretolynx-nginx.png)










