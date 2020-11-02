---
title: "Ej6 Apache2"
date: 2020-11-02T17:15:07+01:00
draft: true
---

# Configuración de apache mediante archivo .htaccess

Un fichero .htaccess (hypertext access), también conocido como archivo de configuración distribuida, es un fichero especial, popularizado por el Servidor HTTP Apache que nos permite definir diferentes directivas de configuración para cada directorio (con sus respectivos subdirectorios) sin necesidad de editar el archivo de configuración principal de Apache.

Para permitir el uso de los ficheros .htaccess o restringir las directivas que se pueden aplicar usamos ela directiva AllowOverride, que puede ir acompañada de una o varias opciones: All, AuthConfig, FileInfo, Indexes, Limit, …

## Ejercicios

Date de alta en un proveedor de hosting. ¿Si necesitamos configurar el servidor web que han configurado los administradores del proveedor?, ¿qué podemos hacer? Explica la directiva AllowOverride de apache2. Utilizando archivos .htaccess realiza las siguientes configuraciones:

1. Habilita el listado de ficheros en la URL http://host.dominio/nas.

Nosotros vamos a usar el hosting gratuito llamado [000webhost](https://es.000webhost.com/) en el cual vamos a crear un host llamado **francisco-htaccess** en el que dentro del directorio llamado *public_html* vamos a tener el fichero ya generado de *.htaccess*:

![htaccess](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/htaccess.png)


Si queremos habilitar el listado de ficheros debemos de saber que directiva es la que nos permite tener el listado de los fichero que tenemos en el directorio **/nas**, para ello tenemos la directiva **Indexes** que nos permite el listado de ficheros:
```shell
Options +Indexes
```
Lo cual, al entrar en /nas veremos que nos permite el lista de ficheros que haya en el directorio /nas:

![nas](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/nas-htaccess.png)

2. Crea una redirección permanente: cuando entremos en ttp://host.dominio/google salte a www.google.es.

Si queremos redireccionar vamos a hacer mediante las directivas redirect debemos de tener activado la directiva **FileInfo** la cual esta relacionada con el mapeo de url como son los alias, redirecciones... Nosotros en este caso lo queremos para realizar una redireccion, por lo que en nuestro fichero de .htaccess tendriamos la siguiente linea:

```shell
AllowOverride FileInfo
Redirect 301 "/google" "www.google.es"
```

3. Pedir autentificación para entrar en la URL http://host.dominio/prohibido. (No la hagas si has elegido como proveedor CDMON, en la plataforma de prueba, no funciona.)

Lo primero sera crear el directorio llamado *prohibido* dentro de public_html. UNa vez creado creamos un index.html con un mensaje de bienvenida. Una vez tengamos todos estos elementos debemos de pasar a la creacion del fichero con los usuarios y las claves, para ello debemos de tener instalado el paquete llamado *apache2-utils* que nos va a permitir la creacion del fichero con las claves y los usuarios:
```shell
#### Creamos el fichero ####
francisco@debian10:~$ htpasswd -n fran
New password: 
Re-type new password: 
fran:$apr1$Z/qhsf3p$TpUscn5.hmJ8bLw6SkEx10

#### Creamos el fichero passwd.txt con la linea que ha salido en pantalla ####
fran:$apr1$Z/qhsf3p$TpUscn5.hmJ8bLw6SkEx10
```

Una vez hecho esto, dentro de la carpeta de prohibido creamos otro fichero *.htaccess* en el cual vamos a tener la siguiente linea:
```shell
AuthType Basic
AuthName "password"
AuthUserFile "/storage/ssd5/943/15297943/passwd.txt"
Require valid-user
```
Donde le indicamos el tipo de autentificación, el mensaje y el fichero que debe de leer para los usuarios. Entonces si intentamos entrar en */prohibido* nos saldra algo como lo siguiente:

![autentificacion](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/apache/passwd-htaccess.png)

