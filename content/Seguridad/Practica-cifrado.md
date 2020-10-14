---
title: "Practica Cifrado"
date: 2020-10-07T18:57:04+02:00
draft: true
---

# Cifrado asimétrico con gpg y openssl

En esta practica sobre el cifrado asimétrico con gpg y openssl vamos a realizar las siguientes tareas:

* Tarea 1: Generación de claves
    1. Genera un par de claves (pública y privada). ¿En que directorio se guarda las claves de un usuario?
    2. Lista las claves públicas que tienes en tu almacén de claves. Explica los distintos datos que nos muestra. ¿Cómo deberías haber generado las claves para indicar, por ejemplo, que tenga un 1 mes de validez?
    3.  las claves privadas de tu almacén de claves.

* Tarea 2: Importar / exportar clave pública
    1. Exporta tu clave pública en formato ASCII y guardalo en un archivo nombre_apellido.asc y envíalo al compañero con el que vas a hacer esta práctica.
    2. Importa las claves públicas recibidas de vuestro compañero.
    3. Comprueba que las claves se han incluido correctamente en vuestro keyring.

* Tarea 3: Cifrado asimétrico con claves públicas
    1. Cifraremos un archivo cualquiera y lo remitiremos por email a uno de nuestros compañeros que nos proporcionó su clave pública.
    2. compañero, a su vez, nos remitirá un archivo cifrado para que nosotros lo descifremos.
    3. Tanto nosotros como nuestro compañero comprobaremos que hemos podido descifrar los mensajes recibidos respectivamente.
    4. Por último, enviaremos el documento cifrado a alguien que no estaba en la lista de destinatarios y comprobaremos que este usuario no podrá descifrar este archivo.
    5. Para terminar, indica los comandos necesarios para borrar las claves públicas y privadas que posees.

* Tarea 4: Exportar clave a un servidor público de claves PGP
    1. Genera la clave de revocación de tu clave pública para utilizarla en caso de que haya problemas.
    2. Exporta tu clave pública al servidor *pgp.rediris.es*
    3. Borra la clave pública de alguno de tus compañeros de clase e impórtala ahora del servidor público de rediris.

* Tarea 5: Cifrado asimétrico con openssl
    1. Genera un par de claves (pública y privada).
    2. Envía tu clave pública a un compañero.
    3. Utilizando la clave pública cifra un fichero de texto y envíalo a tu compañero.
    4. Tu compañero te ha mandado un fichero cifrado, muestra el proceso para el descifrado.


## Tarea 1

Lo primero que vamos a hacer es crearnos nuestro par de claves, para ello debemos usar el siguiente comando:

```shell
gpg --gen-key
```

Una vez generada nuestro par de claves nos pedirán unas credenciales como son el nombre y el correo electrónico con una frase de paso, la cual podemos ponerla o no, en este caso no la pondremos. Estas claves se guardan en el directorio ***/home/francisco/.gnupg/***

Si quisiéramos ver la lista de claves que tenemos en nuestro sistema solo debemos de ejecutar el siguiente comando:

```shell
#### Ver lista de claves públicas ####
gpg --list-keys

#### Ver lista de claves privadas ####
gpg --list-secret-keys
```

Al ejecutar el comando anterior veremos que nos aparece en pantalla información como son el dia de creación de la clave publica y su fecha de caducidad, el uuid de nuestra clave publica, las credenciales de esa clave publica y la fecha de creación y validez de la subclave pública.

Si nosotros quisiéramos hacer que esta clave que hemos creado caduque en un periodo de un mes solo debemos de localizar el uuid de esta clave mediante el listado de claves, una vez lo tengamos debemos de escribir en la terminal el siguiente comando:

```shell
#### Entrar en prompt de gpg ####

gpg --edit-key (uuid de la clave publica)
```

De esta forma nos abrira un prompt de gpg en el cual debemos de seleccionar nuestra clave publica, para ello usamos el comando **key 0** y veremos que tendremos seleccionada la clave publica, una vez hecho esto solo debemos escribir **expire** y nos mostrara las siguientes opciones para poder hacer que solo tenga un mes de validez:

```shell
gpg> expire
Cambiando caducidad de clave primaria.
Por favor, especifique el período de validez de la clave.
         0 = la clave nunca caduca
      <n>  = la clave caduca en n días
      <n>w = la clave caduca en n semanas
      <n>m = la clave caduca en n meses
      <n>y = la clave caduca en n años
¿Validez de la clave (0)? 1m
La clave caduca jue 12 nov 2020 16:50:19 CET
¿Es correcto? (s/n) s

```

De esta forma hemos cambiado la validez de la clave publica a 1 mes, pero tambien debemos de hacer el mismo proceso con la subclave publica, para ello primero la seleccionamos con el comando **key 1** y una vez seleccionada solo debemos de realizar el mismo proceso que con la clave publica, primero escibimos **expire** y luego seleccionamos el tiempo de validez.

## Tarea 2

Para enviar archivos cifrados a otras personas, necesitamos disponer de sus claves públicas. De la misma manera, si queremos que cierta persona pueda enviarnos datos cifrados, esta necesita conocer nuestra clave pública. Para ello, podemos hacérsela llegar por email por ejemplo. Cuando recibamos una clave pública de otra persona, esta deberemos incluirla en nuestro keyring o anillo de claves, que es el lugar donde se almacenan todas las claves públicas de las que disponemos.

Si nosotros queremos exportar nuestra clave publica solo debemos de realizar el siguiente comando:

```shell
#### Exportar en formato binario ####

gpg --export "Claves_pruebas" > francisco_martin.asc

#### Exportar en formato ascii ####

gpg --armor --export "Claves_pruebas" > francisco_martin.asc
```

Cuando hayamos recibido la clave publica de nuestro compañero debemos de importar esas claves a nuestro equipo, para ello vamos a usar el siguiente comando:

```shell
#### Importar una clave publica ####

gpg --import nombre_clave_publica.key

#### Importar una clave privada ####

gpg --allow-secret-key-import --import nombre_clave_privada.key
```

Una vez importada la clave de nuestro compañero vamos a ver que se haya añadido correctamente, para ello listamos las claves publicas que tenemos.

## Tarea 3

Tras realizar el ejercicio anterior, podemos enviar ya documentos cifrados utilizando la clave pública de los destinatarios del mensaje.

Primero lo que debemos de hacer es cifrar un mensaje para que nuestro compañero con nuestra clave publica que nosotros le hemos proporcionado pueda descifrarlo, para ello usamos el siguiente listado de comandos:

```shell
#### Creamos el fichero de prueba ####

nano prueba1.txt -> Escribimos dentro algo para poder hacer la prueba

#### Ciframos el fichero de prueba con la clave publica de nuestro compañero ####

gpg -e -r nombre_clave_compañero prueba1.txt

donde:
-e: le indicamos a gpg que vamos a encriptar un fichero
-r: Indicamos la clave publica con la que vamos a encriptar, en este caso la clave publica de nuestro compañero
```

De esta forma nos va a generar un fichero llamado ***prueba1.txt.gpg*** que es el fichero que tenemos que enviarle a nuestro compañero y nosotros a su vez tenemos que recibir un fichero como el que hemos enviado pero esta vez el fichero que hemos recibido esta cifrado con nuestra clave publica y tendremos que desemcriptar con nuestra clave privada, para ello usamos el siguiente comando:

```shell
#### Desencriptamos el fichero que hemos recibido ####

gpg -d prueba1.txt.gpg
...
gpg: cifrado con clave de 3072 bits RSA, ID F1274AC86BAC6840, creada el 2020-10-13
      "Claves_pruebas"
hola esto es una prueba

#### Si queremos redireccionar el resultado a un fichero ####

gpg -d prueba1.txt.gpg > resultado.txt
```

Una vez hecho esto el contenido del fichero cifrado lo tendremos, en este caso, en el fichero *resultado.txt*. Ahora bien, si queremos ver que de verdad funciona el cifrado le vamos a enviar a otro compañero el mismo fichero cifrado que hemos enviado al compañero que tiene la clave privada para descifrarlo. Cuando el compañero que no tiene la clave privada para poder desencriptar el fichero intente desencriptarlo le aparecerá el siguiente error:

```shell
gpg -d prueba1.txt.gpg > resultado.txt
...
gpg: cifrado con clave RSA, ID F1274AC86BAC6840
gpg: descifrado fallido: No secret key
```

Ya que hemos realizado las correspondientes pruebas con la clave publica de nuestro compañero vamos a borrarla de nuestro llavero de claves, para ello vamos a usar el siguiente comando:

```shell
#### Borrar una clave privada ####

gpg --delete-secret-key nombre_claves

#### Borrar clave publica ####

gpg --delete-key nombre_claves

Tenemos que tener en cuenta que si queremos borrar un par de claves primero debemos de borrar la clave privada y
luego la clave publica
```

## Tarea 4

Para distribuir las claves públicas es mucho más habitual utilizar un servidor específico para distribuirlas, que permite a los clientes añadir las claves públicas a sus anillos de forma mucho más sencilla.

Para ello voy a usar la clave publica de mi par de claves que tiene como nombre *Francisco Javier Martín Núñez*, esa clave de revocacion de nuestra clave publica la vamos a guardar en un fichero llamado ***revocacion.asc***, para poder hacer esto debemos de realizar el siguiente comando:

```shell
#### Creacion de clave de revocacion ####

gpg --gen-revoke "Francisco Javier Martín Núñez" > revocacion.asc

sec  rsa3072/BEAECEA4DC2F7A96 2020-10-07 Francisco Javier Martín Núñez <franjaviermn17100@gmail.com>

¿Crear un certificado de revocación para esta clave? (s/N) s
Por favor elija una razón para la revocación:
  0 = No se dio ninguna razón
  1 = La clave ha sido comprometida
  2 = La clave ha sido reemplazada
  3 = La clave ya no está en uso
  Q = Cancelar
(Probablemente quería seleccionar 1 aquí)
¿Su decisión? 0
Introduzca una descripción opcional; acábela con una línea vacía:
> Pruebas para seguridad
> 
Razón para la revocación: No se dio ninguna razón
Pruebas para seguridad
¿Es correcto? (s/N) s
se fuerza salida con armadura ASCII.
Certificado de revocación creado.
```

De esta forma tendremos nuestra clave de revocacion en el fichero ***revocacion.asc***. Ahora que tenemos la clave de revocacion vamos a exportar nuestra clave publica a el sevidor de claves ***pgp.rediris.es*** de la siguiente forma:

```shell
gpg --send-keys --keyserver pgp.rediris.es (UUID de la clave publica)
```

Ahora vamos a importar desde el servidor la clave publica de nuestro compañero, para ello vamos a usar el siguiente comando:

```shell
gpg --search-key --keyserver pgp.rediris.es (identificador de la clave publica)
```

## Tarea 5

En esta ocasión vamos a cifrar nuestros ficheros de forma asimétrica utilizando la herramienta openssl. Para ello nos vamos a generar nuestro par de claves, para ello vamos a usar los siguientes comandos:

```shell
#### Generar clave pública y privada en formato PEM sin contraseña ####

openssl genrsa -out claves.pem 2048

#### Generar clave pública y privada en formato PEM con contraseña ####

openssl genrsa -aes128 -out claves.pem 2048

#### Obtención o separación de la clave pública en formato PEM ####

openssl rsa -in claves.pem -pubout -out publica.public.pem
```

Una vez hecho esto, tenemos que separar nuestras claves y obtener la clave publica que es la que le vamos a enviar a nuestro compañero, en este caso, nuestra clave publica llamada ***publica.public.pem***.

Una vez hecho esto vamos a cifrar algun fichero con nuestra clave publica, para ello vamos a usar el siguiente comando:

```shell
#### Paso 1, generar clave aleatoria ####

openssl rand -base64 48 -out key.txt

#### Paso 2, cifrar el archivo con la clave simétrica ####

openssl enc -aes-256-cbc -pass file:key.txt -in archivoSINcifrar -out archivoCIFRADO.enc

#### Paso 3, cifrar la clave generada en el paso 1 con la llave pública ####

openssl rsautl -encrypt -in key.txt -out key.enc -inkey publica.public.pem -pubin

Donde:
    -enc -aes-256-cbc: Tipo de cifrado simétrico.
    -pass file:key.txt: La clave con la que cifrar el archivo.
    -in: Fichero de entrada.
    -out: Fichero de salida.
    -rsautl: Indica que vamos a usar RSA para firmar, verificar, cifrar o descifrar.
    -encrypt: Encriptar el fichero de entrada con una llave RSA pública.
    -inkey: Llave con la que cifrar
    -pubin: Indica que vamos a firmar con una llave pública.
```

Y si queremos desemcriptar el fichero que hemos encriptado:

```shell
#### Paso 1, desciframos la clave generada en 1 y cifrada con la llave pública en 3 ####

openssl rsautl -decrypt -inkey ./claves.pem -in key.enc -out key.txt

#### Paso 2, Descifrar el archivo con la clave ####

openssl enc -aes-256-cbc -d -pass file:key.txt -in archivoCIFRADO.encrypted -out archivoSINcifrar

Donde:
    -d: Descifra los datos de entrada.
```






