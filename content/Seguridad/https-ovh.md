---
title: "Https Ovh"
date: 2020-11-27T11:04:59+01:00
draft: true
---

# HTTPS en ovh

1. Vamos a utilizar el servicio https://letsencrypt.org para solicitar los certificados de nuestras páginas.

2. Explica detenidamente cómo se solicita un certificado en Let's Encrypt. En tu explicación deberás responder a estas preguntas:
    * ¿Qué función tiene el cliente ACME?
    * ¿Qué configuración se realiza en el servidor web?
    * ¿Qué pruebas realiza Let's Encrypt para asegurar que somos los administrados del sitio web?
    * ¿Se puede usar el DNS para verificar que somos administradores del sitio?

3. Utiliza dos ficheros de configuración de nginx: uno para la configuración del virtualhost HTTP y otro para la configuración del virtualhost HTTPS.

4. Realiza una redirección o una reescritura para que cuando accedas a HTTP te redirija al sitio HTTPS.

5. Comprueba que se ha creado una tarea cron que renueva el certificado cada 3 meses.

6. Comprueba que las páginas son accesible por HTTPS y visualiza los detalles del certificado que has creado.

7. Modifica la configuración del cliente de Nextcloud para comprobar que sigue en funcionamiento con HTTPS.

## Proceso de solicitar el certificado en Let's Encrypt

* ¿Qué es Let's Encrypt?

Let’s Encrypt es una autoridad de certificación (CA) automatizada, libre y gratuita que nace de un esfuerzo conjunto para el beneficio de la comunidad y no para el control de cualquier organización. Es un servicio proporcionado y promovido por el Internet Security Research Group en su empeño por hacer que todo el tráfico en la web sea seguro.

* ¿Que es el protocolo ACME?

El protocolo ACME permite comunicar con la AC directamente del servidor y sirve para la obtención e instalación automática de los certificados SSL/TLS. Un cliente ACME de calidad es capaz de instalar el certificado en el servidor de manera que no se tenga que hacer otra cosa. El proceso es completamente automático y por eso, el certificado no es necesario que sea instalado por un administrador, lo que ahorra tiempo y gastos.

* ¿Qué configuración se realiza en el servidor web?

La primera vez que nuestro cliente de Let’s Encrypt se conecta con la CA se genera el par de claves. El proceso entre el cliente de Let’s Encrypt y la CA se resume asi:

1. Let’s Encrypt CA le pide a nuestro cliente que aloje un fichero en una dirección concreta del dominio, y le pasa además un token para que lo firme.

2. El cliente procede a guardar el archivo bajo el dominio correspondiente y firma el token con la clave privada; notifica a la CA de que está listo.

3. La CA verifica la firma del token con la clave pública de nuestro servidor e intenta descargar el fichero desde la dirección acordada.

4. Si todo es correcto, entonces el servidor correspondiente a esa clave pública queda autorizado para realizar peticiones de certificados.

* ¿Qué pruebas realiza Let's Encrypt para asegurar que somos los administrados del sitio web?

Para validar que se tiene control sobre el dominio se utiliza un par de claves pública-privada más una serie de retos (challenges) que el servidor ha de realizar.

* ¿Se puede usar el DNS para verificar que somos administradores del sitio?

Si se puede usar ya que existe un *challenge* que se hace mediante dns, pero ese *challenge* no lo vamos a necesitar en esta practica, por lo que no explicare como se hace con dns.


En mi caso vamos a configurar dos sitios web en el dominio **iesgn13.es** que son **www.iesgn13.es** y **portal.iesgn13.es**.

## Configurando portal.iesgn13.es

Para esta configuracion vamos a usar el plugin de nginx el cual nos va a configurar el sitio completamente solo, sin necesidad de tocar nada, para ello en nuestra maquina de ovh debemos de instalar el agente de clertbot, para ello:
```shell
#### Instalamos clertbot ####
debian@omega:~$ sudo apt install certbot

#### Instalamos el plugin de nginx ####
debian@omega:~$ sudo apt install python-certbot-nginx
```

Una vez lo tengamos instalado solo debemos de ejecutar el comando siguiente para poder usar certbot con el siguiente comando y seguir sus instrucciones:
```shell
debian@omega:~$ sudo certbot --nginx -d portal.iesgn13.es
```

Con la opcion **-d** le indicamos el dominio que vamos a configurar. Asi, si entramos en http://portal.iesgn13.es veremos que nos redirige a https.

En el enunciado de la practica nos indica que tenemos que tener por separado el fichero de http y el fichero de https, para ello nos dirigimos al directorio donde tenemos los fichero de configuracion y ahi debemos de realizar una copia del fichero de drupal.conf que el nombre que tiene mi fichero, lo copiamos con el nombre de **drupal-https.conf**. Una vez lo tengamos tenemos que eliminar las lineas donde nos hace las redirecciones hacia https ya que este fichero es el de https, por lo que debe de quedar asi:
```shell
#### Realizamos la copia del fichero ####
debian@omega:/etc/nginx/sites-available$ sudo cp drupal.conf ./drupal-https.conf

#### Realizamos las modificaciones pertinentes ####
debian@omega:/etc/nginx/sites-available$ sudo nano drupal-https.conf

server {

    root /srv/www/drupal;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html;

    server_name portal.iesgn13.es;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        #try_files $uri $uri/ =404;
        try_files $uri /index.php?$query_string;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        # With php-fpm (or other unix sockets):
        fastcgi_pass unix:/run/php/php7.3-fpm.sock;
    }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/portal.iesgn13.es/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/portal.iesgn13.es/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

#### Creamos el enlace simbolico hacia sites-enabled ####
debian@omega:/etc/nginx/sites-available$ sudo ln -s /etc/nginx/sites-available/drupal-https.conf /etc/nginx/sites-enabled/
```

Una vez lo tengamos este sera nuestro fichero para https, donde tenemos todo lo necesario para drupal funcione con https pero ahora debemos de tener nuestro otro fichero llamado drupal.conf en el cual vamos a poner solo las redirecciones que no ha creado certbot, para ello el fichero debe de quedar de la siguiente forma:
```shell
#### Editamos el fichero para que nos quede asi ####
debian@omega:/etc/nginx/sites-available$ sudo nano drupal.conf

server {
    if ($host = portal.iesgn13.es) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80;

        server_name portal.iesgn13.es;
    return 404; # managed by Certbot


}
```

Una vez tengamos esto listo solo debemos de comprobar la sintaxis de nginx con el comando **nginx -t**, si todo esta correcto reiniciamos nginx e intentamos entrar en **portal.iesgn13.es** y veremos que ya tenemos nuestro sitio funcionando perfectamente.

![https](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/https-certbot/https-portal.png)

### Creacion de la renovacion automatica

La renovacion automatica del certificado se crea mediante una tarea cron en el servidor de OVH, pero de esto ya se encarga certbot para crearla y que se ejecute automaticamnete para renovar el certificado cada 3 meses.

## Configurando www.iesgn13.es

Para realizar esta configuracion vamos a usar en vez del plugin que usamos anteriormente, vamos a usar el plugin de **standalone**, para poder usarlo vamos a usar el siguiente comando:

**nota**: Debemos de parar el servicio de nginx para poder realizar los challenges de http.
```shell
debian@omega:/etc/nginx/sites-available$ sudo certbot certonly --standalone --preferred-challenges http -d www.iesgn13.es
```

Una vez hecho esto debemos de crear dos ficheros, uno para http y otro para https, como el de http ya lo tenemos lo copiamos con otro nombre. Una vez copiado añadimos las credenciales de los certificados y claves necesarias para que pueda servir https y en el fichero de http creamos una redireccion hacia https:
```shell
#### Copiamos el fichero con otro nombre ####
debian@omega:/etc/nginx/sites-available$ sudo cp iesgn13.conf ./iesgn13-https.conf

#### Modificamos las lineas del fichero iesgn13-https.conf ####
#Añadir
listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/www.iesgn13.es/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.iesgn13.es/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

#eliminar
listen 80 default_server;
listen [::]:80 default_server;

#### En el fichero iesgn13.conf debemos de borrar toda la configuracion y tener solo lo siguiente ####
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        #redireccion hacia https
        return 301 https://$server_name:443$request_uri;

        server_name www.iesgn13.es;

}

#### Comprobamos sintaxis y reiniciamos el servicio ####
debian@omega:/etc/nginx/sites-available$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

debian@omega:/etc/nginx/sites-available$ sudo systemctl restart nginx.service
```

Ahora si entramos en ***www.iesgn13.es** debe de redirigirnos a la pagina con https.

![iesgn https](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/https-certbot/https-iesgn.png)

Ahora si nos dirigimos a nuestro cliente de nextcloud e introducimos en vez de **http** usamos **https** veremos que nos dejara hacer el login y podremos visualizar nuestro cloud, esta vez con **https**

![nextcloud https](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/https-certbot/https-portal.png)

Si queremos ver mas informacion respecto al certificado que nos ha generado podemos simplemente ir al candado verde, ver mas informacion acerca del certificado, y nos saldra la siguiente ventana con la informacion del certificado:

![certificado drupal](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/https-certbot/cert-drupal.png)

![certificado www](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/https-certbot/cert-www.png)