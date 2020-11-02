---
title: "Ej5 Apache2"
date: 2020-11-02T17:15:12+01:00
draft: true
---

# Ejercicio 5: Control de acceso, autentificación y autorización

El Control de acceso en un servidor web nos permite determinar desde donde podemos acceder a los recursos del servidor. En los servidores apache podemos tener dos tipos de Autentificación:

## Autentificación básica

Es un método simple que utiliza codificación Base64 y por lo tanto es menos seguro. Podemos hacer aún más seguro el acceso a nuestras páginas sensibles haciendo uso conjunto de HTTPS y autenticación Basic, es decir, que en primer lugar nuestro servidor web utilice SSL o TLS en las conexiones y sobre la conexión encriptada, usar Basic como método de autenticación. La configuración que tenemos que añadir en el fichero de definición del Virtual Host a proteger podría ser algo así:

```shell
<Directory "/var/www/misitio/secreto">
    AuthUserFile "/etc/apache2/claves/passwd.txt"
    AuthName "Password"
    AuthType Basic
    Require valid-user
</Directory>
```
Donde podemos ver los siguientes elementos:
* En AuthUserFile ponemos el fichero que guardará la información de usuarios y contraseñas que debería de estar, como en este ejemplo, en un directorio que no sea visitable desde nuestro Apache. Ahora comentaremos la forma de generarlo.

* Por último, en AuthName personalizamos el mensaje que aparecerá en la ventana del navegador que nos pedirá la contraseña.

* Para controlar el control de acceso, es decir, que usuarios tienen permiso para obtener el recurso utilizamos las siguientes directivas: AuthGroupFile, Require user, Require group.

El fichero de contraseñas se genera mediante la utilidad htpasswd. Su sintaxis es bien sencilla. Para añadir un nuevo usuario al fichero operamos así:
```shell
#### Creamos un directorio en /etc/apache2 llamado claves ####
vagrant@servidorApache2:~$ sudo mkdir /etc/apache2/claves

#### Generamos las claves con sus usuarios ####
vagrant@servidorApache2:~$ sudo htpasswd -c /etc/apache2/claves/passwd.txt (usuario)

#### Si queremos añadir un nuevo usuario usamos: ####
vagrant@servidorApache2:~$ sudo htpasswd /etc/apache2/claves/passwd.txt (nuevo usuario)

#### Contenido del fichero passwd.txt ####
fran:$apr1$ZAyQI3OK$.sThEHeftqb1OoNK1qPnT.
pilar:$apr1$6i7G2usl$m7fUsxL6PqW/abilugzVr0
```

Para denegar el acceso a algún usuario basta con que borremos la línea correspondiente al mismo. No es necesario que le pidamos a Apache que vuelva a leer su configuración cada vez que hagamos algún cambio en este fichero de contraseñas.

### ¿Cómo funciona este metodo?

Cuando desde el cliente intentamos acceder a una URL que esta controlada por el método de autentificación básico:

1. El servidor manda una respuesta del tipo 401 HTTP/1.1 401 Authorization Required con una cabecera WWW-Authenticate al cliente de la forma:
```shell
WWW-Authenticate: Basic realm="Palabra de paso"
```

2. El navegador del cliente muestra una ventana emergente preguntando por el nombre de usuario y contraseña y cuando se rellena se manda una petición con una cabecera Authorization:
```shell
Authorization: Basic am9zZTpqb3Nl
```

## Autentificación tipo digest

La autentificación tipo digest soluciona el problema de la transferencia de contraseñas en claro sin necesidad de usar SSL. El módulo de autenticación necesario suele venir con Apache pero no habilitado por defecto. Para activarlo usamos la utilidad a2enmod y, a continuación reiniciamos el servidor Apache:
```shell
#### Activamos el modulo auth_digest ####
vagrant@servidorApache2:~$ sudo a2enmod auth_digest

#### Reiniciamos el servicio ####
vagrant@servidorApache2:~$ sudo systemctl reload apache2.service
```

Luego incluimos una sección como esta en el fichero de configuración de nuestro Virtual Host:
```shell
<Directory "/var/www/misitio/secreto">
    AuthType Digest
    AuthName "dominio"
    AuthUserFile "/etc/claves/digest.txt"
    Require valid-user
</Directory>
```
Donde vemos:
* AuthName: se usa también para identificar un nombre de dominio (realm) que debe de coincidir con el que aparezca después en el fichero de contraseñas.

Para poder generar las contraseñas para los usuarios usamos el comando **htdigest**:
```shell
#### Creamos el fichero con htdigest ####
vagrant@servidorApache2:~$ sudo htdigest -c /etc/apache2/claves/digest.txt dominio1 fran

#### Añadir una nueva clave con el usuario ####
vagrant@servidorApache2:~$ sudo htdigest /etc/apache2/claves/digest.txt dominio1 pilar

#### Contenido del fichero digest.txt ####
fran:dominio1:b3b17cd36b06d1b16ab66c2a9e9e27e6
pilar:dominio1:8e015255f5a4174dbabe9ab3c208d4d6
```

### ¿Cómo funciona el metodo digest?

Cuando desde el cliente intentamos acceder a una URL que esta controlada por el método de autentificación de tipo digest:

1. El servidor manda una respuesta del tipo 401 HTTP/1.1 401 Authorization Required con una cabecera WWW-Authenticate al cliente de la forma:
```
WWW-Authenticate: Digest realm="dominio", 
                  nonce="cIIDldTpBAA=9b0ce6b8eff03f5ef8b59da45a1ddfca0bc0c485", 
                  algorithm=MD5, 
                  qop="auth"
```

2. El navegador del cliente muestra una ventana emergente preguntando por el nombre de usuario y contraseña y cuando se rellena se manda una petición con una cabecera Authorization:
```shell
Authorization	Digest username="jose", 
                 realm="dominio", 
                 nonce="cIIDldTpBAA=9b0ce6b8eff03f5ef8b59da45a1ddfca0bc0c485",
                 uri="/digest/", 
                 algorithm=MD5, 
                 response="814bc0d6644fa1202650e2c404460a21", 
                 qop=auth, 
                 nc=00000001, 
                 cnonce="3da69c14300e446b"
```

La información que se manda es responde que en este caso esta cifrada usando md5 y que se calcula de la siguiente manera:

* Se calcula el md5 del nombre de usuario, del dominio (realm) y la contraseña, la llamamos HA1.
* Se calcula el md5 del método de la petición (por ejemplo GET) y de la uri a la que estamos accediendo, la llamamos HA2.
* El reultado que se manda es el md5 de HA1, un número aleatorio (nonce), el contador de peticiones (nc), el qop y el HA2.



# Ejercicios

Crea un escenario en Vagrant o reutiliza uno de los que tienes en ejercicios anteriores, que tenga un servidor con una red publica, y una privada y un cliente conectada a la red privada. Crea un host virtual departamentos.iesgn.org.

1. A la URL departamentos.iesgn.org/intranet sólo se debe tener acceso desde el cliente de la red local, y no se pueda acceder desde la anfitriona por la red pública. A la URL departamentos.iesgn.org/internet, sin embargo, sólo se debe tener acceso desde la anfitriona por la red pública, y no desde la red local.

2. Autentificación básica. Limita el acceso a la URL departamentos.iesgn.org/secreto. Comprueba las cabeceras de los mensajes HTTP que se intercambian entre el servidor y el cliente. ¿Cómo se manda la contraseña entre el cliente y el servidor?. Entrega una breve explicación del ejercicio.

3. Cómo hemos visto la autentificación básica no es segura, modifica la autentificación para que sea del tipo digest, y sólo sea accesible a los usuarios pertenecientes al grupo directivos. Comprueba las cabeceras de los mensajes HTTP que se intercambian entre el servidor y el cliente. ¿Cómo funciona esta autentificación?

4. Vamos a combinar el control de acceso (tarea 6) y la autentificación (tareas 7 y 8), y vamos a configurar el virtual host para que se comporte de la siguiente manera: el acceso a la URL departamentos.iesgn.org/secreto se hace forma directa desde la intranet, desde la red pública te pide la autentificación. Muestra el resultado al profesor.

## Tarea 1

Lo primero va a ser la creacion de un virtualhost, para ellos seguimos los siguientes pasos:

1. Lo primero que debemos de hacer es crear nuestro virtualhost, para ello lo que debemos de hacer es crear nuestro virtualhost. Nos dirigimos al directorio */etc/apache2/sites-available* y ahi copiamos el fichero de configuracion de default con un nuevo nombre:
```shell
sudo cp 000-default.conf ./departamentos.conf
```

2. Asi en nuestro fichero de configuracion de departamentos debemos de tenerlo de la siguiente forma:
```shell
<VirtualHost *:80>
    ServerName www.departamentos.iesgn.org
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/departamentos
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

3. Ahora que tenemos el fichero configurado vamos a crear una carpeta llamada *departamentos* en el directorio */var/www*. Una vez tengamos creado este directorio en el vamos a crear un fichero llamado *index.html* y vamos a crear una linea con un mensaje de bienvenida:
```shell
#### Creamos el index.html ####
sudo nano /var/www/departamentos/index.html

#### Escribimos un mensaje de bienvenida ####

<h1> Hola que tal, bienvenido a departamentos </h1>
```

4. Activamos el nuevo virtualhost:
```shell
sudo a2ensite departamentos
```

Ahora que tenemos nuestro virtualhost vamos a empezar la practica, lo primero que debemos de hacer es crear las dos carpetas *internet y intranet* en **/var/www/departamentos**:
```shell
sudo mkdir /var/www/departamentos/{intranet,internet}
```

Ahora que hemos creado las carpetas, dentro de estas vamos a crear un index.html con un mensaje de bienvenida a la pagina. UNa vez lo hayamos hecho nos dirigimos al fichero de configuracion de departamentos y ahi debemos de tener el siguiente esquema:
```shell
<VirtualHost *:80>
    ServerName www.departamentos.org
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/departamentos
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <directory "/var/www/departamentos/internet">
        Require ip 192.168
    </directory>
    <directory "/var/www/departamentos/intranet">
        Require ip 10.0
    </directory>
</VirtualHost>
```

Donde hemos indicado que en directorio /var/www/departamentos/internet solo puedan acceder las maquinas con una ip 192.168.X.X mientras que el directorio /var/www/departamentos/intranet solo pueden acceder desde las ip 10.0.X.X. Por lo que si nosotros intentamos entrar desde neustra maquina anfitrion a /var/www/departamentos/intranet no nos dejara y nos dira que no tenemos permisos para entrar pero si entramos en /var/www/departamentos/internet nos dejara entrar sin problemas.

Entrando en internet:

![Internet](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/prueba-internet.png)

Entrando en Intranet:

![Intranet](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/prueba-intranet.png)


## Tarea 2

En esta tarea debemos de tener una autentificación basica en ***secreto***, para ello vamos a crear un directorio nuevo llamado secreto en nuestro documentroot:
```shell
sudo mkdir /var/www/departamentos/secreto
```

Una vez lo tengamos creado vamos a crear un index.html con un mensaje de bienvenida a secreto. Cuando ya hayamos hecho esto debemos de configura nuestro apache para que nos pida una contraseña y un usuario si queremos entrar en secreto, para ello vamos a generar un fichero con el usuario y la clave en /etc/apache2/claves:
```shell
#### Creamos la carpeta ####
sudo mkdir /etc/apache2/claves

#### Generamos el fichero passwd.txt con el usuario fran####
sudo htpasswd -c /etc/apache2/claves/passwd.txt fran
New password: ********
Re-type new password: *********
Adding password for user fran
```

Ahora debemos de modificar el fichero de configuracion de nuestro virtualhost, para ello debemos de tener la siguiente linea:
```shell
<Directory "/var/www/departamentos/secreto">
    AuthUserFile "/etc/apache2/claves/passwd.txt"
    AuthName "Password"
    AuthType Basic
    Require valid-user
</Directory>
```

Con lo cual estamos indicando que lea los usuarios y contraseñas del fichero que hemos creado, que solo deje entrar a esos usuarios que estan en el fichero para poder entrar en secreto y ver su mensaje.

Si queremos ver las cabeceras http podemos usar el siguiente comando:
```shell
curl -I www.departamentos.org/secreto
    HTTP/1.1 401 Unauthorized
    Date: Tue, 27 Oct 2020 18:39:24 GMT
    Server: Apache/2.4.38 (Debian)
    WWW-Authenticate: Basic realm="Password"
    Content-Type: text/html; charset=iso-8859-1
```

![Clave](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/prueba-secreto.png)

## Tarea 3

Ahora debemos de tener una autentificación digest que es mas segura que la autentificación basica, para ello vamos a usar el mismo directorio secreto y solo podran acceder aquellos del grupo *directivos*, para ello comentamos la autentificación basica que habiamos hecho y creamos de nuevo los fichero de contraseñas, usuario y grupos en /etc/apache2/claves
```shell
#### Creamos el fichero de contraseñas ####
sudo htdigest -c /etc/apache2/claves/digest.txt directivos fran
    Adding password for fran in realm directivos.
    New password:******* 
    Re-type new password:*******
#### Añadimos la siguiente linea a departamentos.conf ####
<Directory "/var/www/departamentos/secreto">
    AuthUserFile "/etc/apache2/claves/digest.txt"
    AuthName "directivos"
    AuthType Digest
    Require valid-user
</Directory>
```

## Tarea 4

Ahoara vamos a aplicar todo lo anterior que hemos aprendido para hacer el escenario que nos marca el enunciado de la tarea 4. Para ello vamos a tener el siguiente fichero departamentos.conf:
```shell

<Directory "/var/www/departamentos/internet">
    <RequireAny>
        <RequireAll>
            Require ip 10.0
        </RequireAll>
        <RequireAll>
            AuthUserFile "/etc/apache2/claves/passwd.txt"
            AuthName "Password"
            AuthType basic
            Require valid-user
        </RequireAll>
    </RequireAny>
</Directory>
```

De esta forma le decimos que debe de cumplirse una de las dos condiciones que le imponemos, en este caso son que, si viene con un ip que comience por 10.0.X.X le deje entrar automaticamente, en cambio, si queremos acceder desde otra ip nos pedira una identificacion, en este caso basica, para poder ver el contenido de la pagina.