---
title: "Practica Integridad"
date: 2020-10-27T18:30:09+01:00
draft: true
---

# Práctica: Integridad, firmas y autenticación


## Tarea 1: Firmas electrónicas

En este apartado vamos a con las firmas electrónicas. Para ello vamos a necesitar subir nuestra clave publica a un servidor de claves, en este caso lo he subido al servidor ***keys.gnupg.net*** con el siguiente comando:

```shell
gpg --send-keys --keyserver pgp.rediris.es D6BE9BF1C7D6A435BF1B9D38BEAECEA4DC2F7A96
```

1. ¿Qué significa el mensaje que aparece en el momento de verificar la firma?

EL mensaje que nos muestra en nuestra terminal siginufica que, si bien tenemos la clave publica para poder verificar la firma, esta no la tenemos firmada por nadie de confianza y nosotros tampoco le hemos dado una confianza a esa clave.

2. Vamos a crear un anillo de confianza entre los miembros de nuestra clase.

El anillo que hemos formado consta de tres personas que en este caso son: Juan Luis Millan Hidalgo, Álvaro Vaca Ferreras y Adrián Rodríguez Povea.

3. Muestra las firmas que tiene tu clave pública.
```shell
francisco@debian10:~$ gpg --list-sign D6BE9BF1C7D6A435BF1B9D38BEAECEA4DC2F7A96
pub   rsa3072 2020-10-07 [SC] [caduca: 2022-10-07]
      D6BE9BF1C7D6A435BF1B9D38BEAECEA4DC2F7A96
uid        [  absoluta ] Francisco Javier Martín Núñez <franjaviermn17100@gmail.com>
sig 3        BEAECEA4DC2F7A96 2020-10-07  Francisco Javier Martín Núñez <franjaviermn17100@gmail.com>
sig          3E0DA17912B9A4F8 2020-10-21  Álvaro Vaca Ferreras <avacaferreras@gmail.com>
sig          73986F40D4BB0593 2020-10-21  Adrián Rodríguez Povea <arodriguezpovea@gmail.com>
sig          15E1B16E8352B9BB 2020-10-27  Juan Luis Millan Hidalgo <juanluismillanhidalgo@gmail.com>
sub   rsa3072 2020-10-07 [E] [caduca: 2022-10-07]
sig          BEAECEA4DC2F7A96 2020-10-07  Francisco Javier Martín Núñez <franjaviermn17100@gmail.com>
```

4. Comprueba que ya puedes verificar sin “problemas” una firma recibida por una persona en la que confías.

5. Comprueba que puedes verificar con confianza una firma de una persona en las que no confías, pero sin embargo si confía otra persona en la que tu tienes confianza total.

Para ello hemos pedido a nuestro compañero **Javier Pérez Hidalgo** que nos ayude. Primero uno de nosotros debe de firmar su clave publica, en este caso **Juan Luis Millan Hidalgo** ha sido el que ha firmado la clave publica de **Javier**. Una vez firmada tenemos que importar su clave firmada a nuestro anillo de claves. Como nosotros tenemos a **Juan Luis Millan Hidalgo** en nuestro anillo de confianza y él confia en **Javier**, a la hora de verificar un fichero firmado nos saldra los siguiente:
```shell
francisco@debian10:~$ gpg --verify Documentos/Seguridad/Encriptado/Claves\ firmadas/apariencia_dashtopanel.gpg 
gpg: Firmado el mar 27 oct 2020 12:50:41 CET
gpg:                usando RSA clave 76270D5E766E0F22D70466BE6F7A456BC67662D3
gpg: Firma correcta de "Javier Pérez Hidalgo <javierperezhidalgo01@gmail.com>" [total]
```


## Tarea 2: Correo seguro con evolution/thunderbird

Ahora vamos a configurar nuestro cliente de correo electrónico para poder mandar correos cifrados, para ello:

1. Configura el cliente de correo evolution con tu cuenta de correo habitual.

Lo primero que debemos de hacer es instalar el cliente de correos evolution, una vez instalado solo debemos de configurarlo siguiendo los pasos que nos indica el configurador, introduciendo nuestro correo electronico.

2. Añade a la cuenta las opciones de seguridad para poder enviar correos firmados con tu clave privada o cifrar los mensajes para otros destinatarios.

Para poder usar las claves privadas para cifrar los correos debemos de irnos al menú **editar -> preferencias**. Una vez ahi nos vamos a la zona de **cuentas de correo** y editamos nuestro correo. UNa vez dentro solo debemos ir a la seccion de seguridad y veremos el apartado de gpg donde tendremos que poner el id de nuestra clave, en mi caso **DC2F7A96**, y seleccionamos los apartados siempre cifrar a mi mismo cuando envie correo cifrado y siempre confiar en las claves de mi almacen al cifrar. De esta forma ya tendremos configurado nuestro servidor para enviar mensajes cifrados con nuestra clave privada.

3. Envía y recibe varios mensajes con tus compañeros y comprueba el funcionamiento adecuado de GPG

* Para enviar: Cuando queramos enviar un mensaje cifrado debemos de irnos a la pestaña de **opciones** y ahi seleccionar **firmar con pgp** y **cifrar con pgp**, asi nuestros mensajes enviados estaran cifrados.
![firma valida](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/Encriptado/firma-valida.png)

## Tarea 3: Integridad de ficheros

Vamos a descargarnos la ISO de debian, y posteriormente vamos a comprobar su integridad.

Puedes encontrar la ISO en la dirección [siguiente](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)

1. Para validar el contenido de la imagen CD, solo asegúrese de usar la herramienta apropiada para sumas de verificación. Para cada versión publicada existen archivos de suma de comprobación con algoritmos fuertes (SHA256 y SHA512); debería usar las herramientas sha256sum o sha512sum para trabajar con ellos.

2. Verifica que el contenido del hash que has utilizado no ha sido manipulado, usando la firma digital que encontrarás en el repositorio. Puedes encontrar una guía para realizarlo en este [artículo](https://linuxconfig.org/how-to-verify-an-authenticity-of-downloaded-debian-iso-images)

Lo primero que debemos de hacer es verificar que la firma es correcta, para ello si realizamos un **--verify** sobre el fichero SHA512SUMS.sign veremos que no podra verificar ya que no tenemos la clave publica de Debian, por lo que debemos de seguir los siguiente pasos para verificar la firma:
```shell
#### Importamos la clave publica de Debian a nuestro anillo de claves ####
francisco@debian10:~/Descargas$ gpg --keyserver keyring.debian.org --recv DF9B9C49EAA9298432589D76DA87E80D6294BE9B


#### Verificamos el fichero SHA512SUMS.sign ####
francisco@debian10:~/Descargas$ gpg --verify SHA512SUMS.sign SHA512SUMS
gpg: Firmado el dom 27 sep 2020 02:24:23 CEST
gpg:                usando RSA clave DF9B9C49EAA9298432589D76DA87E80D6294BE9B
gpg: Firma correcta de "Debian CD signing key <debian-cd@lists.debian.org>" [desconocido]
gpg: ATENCIÓN: ¡Esta clave no está certificada por una firma de confianza!
gpg:          No hay indicios de que la firma pertenezca al propietario.
Huellas dactilares de la clave primaria: DF9B 9C49 EAA9 2984 3258  9D76 DA87 E80D 6294 BE9B
```


## Tarea 4: Integridad y autenticidad

Cuando nos instalamos un paquete en nuestra distribución linux tenemos que asegurarnos que ese paquete es legítimo. Para conseguir este objetivo se utiliza criptografía asimétrica, y en el caso de Debian a este sistema se llama apt secure. Esto lo debemos tener en cuenta al utilizar los repositorios oficiales. Cuando añadamos nuevos repositorios tendremos que añadir las firmas necesarias para confiar en que los paquetes son legítimos y no han sido modificados.

1. ¿Qué software utiliza apt secure para realizar la criptografía asimétrica?

El binario de *apt-secure* usa el software de **gpg**

2. ¿Para que sirve el comando apt-key? ¿Qué muestra el comando apt-key list?

* apt-key: Es un binario que se usa para gestionar el anillo de llaves de gpg para asegurar Apt.
* apt-key list: Nos lista las claves de confianza que usa nuestro sistema con apt

3. ¿En que fichero se guarda el anillo de claves que guarda la herramienta apt-key?

El anillo de claves se guarda en el archivo /etc/apt/trusted.gpg. En mi caso todas la claves se guardan en un directorio llamado **trusted.gpg.d/**

4. ¿Qué contiene el archivo Release de un repositorio de paquetes?. ¿Y el archivo Release.gpg?. Puedes ver estos archivos en el repositorio http://ftp.debian.org/debian/dists/Debian10.1/. Estos archivos se descargan cuando hacemos un apt update.

El archivo Release de un repositorio lo que contiene es la clave publica de ese repositorio para validar que los paquetes que descargamos desde ahi son validos y no han sido modificados.


5. Explica el proceso por el cual el sistema nos asegura que los ficheros que estamos descargando son legítimos.

Siempre descarga los ficheros Release.gpg cuando está descargando los ficheros Release, y si no puede descargar el Release.gpg, o si la firma está mal, lo advertirá, y te dirá que el fichero Packages al cual apunta el fichero Release, y todos los paquetes enumerados dentro, son de una fuente sin autentificar.

6. Añade de forma correcta el repositorio de virtualbox añadiendo la clave pública de virtualbox como se indica en la documentación.

Lo primero que debemos de hacer es añadir la linea del repositorio de virtualbox en el fichero **/etc/apt/sources.list**:
```shell
deb http://deb.debian.org/debian/ buster main contrib non-free
deb http://security.debian.org/debian-security buster/updates main contrib non-free
deb http://deb.debian.org/debian/ buster-updates main contrib non-free
deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian <mydist> contrib
```

Una vez añadido el repositorio necesitamos la clave de oracle para añadirla a nuestro circulo de llaves, para ello podemos hacer lo siguiente:
```
sudo apt-key add oracle_vbox_2016.asc
sudo apt-key add oracle_vbox.asc
```

Con lo cual ya tendriamos las claves de oracle añadidas junto al repositorio de virtualbox y solo nos queda realizar un **apt update** y luego instalar la version de virtualbox que nosotros necesitemos.


## Tarea 5: Autentificación: ejemplo SSH

Vamos a estudiar como la criptografía nos ayuda a cifrar las comunicaciones que hacemos utilizando el protocolo ssh, y cómo nos puede servir también para conseguir que un cliente se autentifique contra el servidor. Responde las siguientes cuestiones:

1. Explica los pasos que se producen entre el cliente y el servidor para que el protocolo cifre la información que se transmite? ¿Para qué se utiliza la criptografía simétrica? ¿Y la asimétrica?

* Claves simetricas: Las claves simétricas se utilizan para cifrar toda la comunicación durante una sesión SSH. Tanto el cliente como el servidor derivan la clave secreta utilizando un método acordado, y la clave resultante nunca se revela a terceros. El proceso de creación de una clave simétrica se lleva a cabo mediante un algoritmo de intercambio de claves.

* Claves asimetricas: el cifrado asimétrico no se utiliza para cifrar toda la sesión SSH. En lugar de eso, sólo se utiliza durante el algoritmo de intercambio de claves de cifrado simétrico. Antes de iniciar una conexión segura, ambas partes generan pares de claves públicas-privadas temporales y comparten sus respectivas claves privadas para producir la clave secreta compartida.

Una vez que se ha establecido una comunicación simétrica segura, el servidor utiliza la clave pública de los clientes para generar y desafiar y transmitirla al cliente para su autenticación. Si el cliente puede descifrar correctamente el mensaje, significa que contiene la clave privada necesaria para la conexión. Y entonces comienza la sesión SSH.

2. Explica los dos métodos principales de autentificación: por contraseña y utilizando un par de claves públicas y privadas.

* Usando par de claves: Esta forma es la mas eficaz y rapida ya que no tendremos que recordar la contraseña para entrar en el servidor. Para usar el metodo por par de claves tenemos que generar en nuestro equipo cliente un par de claves privada y publica:
```shell
vagrant@maquina:~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/vagrant/.ssh/id_rsa.
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:5n5dc+Lwx+B/yFnHx/Ne6JbLOmmAsSGqhyr6Hculc6U vagrant@maquina
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|                 |
|      . o        |
|     . .S=     o |
|    .  o+ . . =oB|
|   o. .o.  o O.@*|
|. ooo=E.  . =oX *|
|=o..=o  .. ..++=+|
+----[SHA256]-----+
```
De esta forma vamos a generar dos claves en el directorio **/home/usuario/.ssh** de las cuales una es la publica(id_rsa.pub), que es la que tenemos que pasar al servidor donde vamos a hacer conexion ssh, y un aclave privada(id_rsa) que no debe de ser revelada a nadie. Ahora que tenemos la clave privada y la publica tenemos que pasarle a nuestro servidor la clave publica, para ello nos dirigimos al directorio .ssh/ y ahi ejecutamos el siguiente comando:
```shell
ssh-copy-id -i ./id_rsa.pub usuario@host
```
De esta forma ya tendriamos en nuestro servidor la clave publica. Ya solo queda entrar en el servidor con la clave publica, para ello vamos al directorio .ssh/ y ejecutamos el siguiente comando:
```shell
ssh -i id_rsa usuario@host
```

* Usando el sistema con contraseña: Este es el sistema mas sencillo donde lo unico que debemos de conocer es la contraseña del usuario del servidor a donde nos vamos a conectar, de tal forma al realizar la conexion nos va a pedir la contraseña del usuario:
```shell
ssh usuario@host
```

3. En el cliente para que sirve el contenido que se guarda en el fichero ~/.ssh/know_hosts?

Este fichero contiene las claves de host DSA de los servidores SSH a los que se accede mediante el usuario. Este fichero es muy importante para asegurse de que el cliente SHH está conectado al servidor SSH correcto.

4. ¿Qué significa este mensaje que aparece la primera vez que nos conectamos a un servidor?
```shell
$ ssh debian@172.22.200.74
The authenticity of host '172.22.200.74 (172.22.200.74)' can't be established.
ECDSA key fingerprint is SHA256:7ZoNZPCbQTnDso1meVSNoKszn38ZwUI4i6saebbfL4M.
Are you sure you want to continue connecting (yes/no)?
```

Este es el mensaje que nos muestra la primera vez que nos intentamos conectar por via ssh por el metodo de clave publica y privada a una maquina a la que nunca nos hemos conectado. Nos pregunta si de verdad queremos entrar en esa maquina.

5. En ocasiones cuando estamos trabajando en el cloud, y reutilizamos una ip flotante nos aparece este mensaje:
```shell
$ ssh debian@172.22.200.74
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:W05RrybmcnJxD3fbwJOgSNNWATkVftsQl7EzfeKJgNc.
Please contact your system administrator.
Add correct host key in /home/jose/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/jose/.ssh/known_hosts:103
    remove with:
    ssh-keygen -f "/home/jose/.ssh/known_hosts" -R "172.22.200.74"
ECDSA host key for 172.22.200.74 has changed and you have requested strict checking.
```

ESto ocurre porque esa ip flotante pertenecia a otra maquina que no es a la que nos estamos intentado conectar con shh ya que en nuestro fichero de **known_hosts** aparece que nos conectamos a esa ip en una determinada maquina pero esa maquina ya no existe y a cambiado a otra maquina, por lo que, como bien dice en el mensaje de error debemos de remover de nuestro fichero **known_hosts** esa ip.

6. ¿Qué guardamos y para qué sirve el fichero en el servidor ~/.ssh/authorized_keys?

El fichero authorized_keys es el fichero que contiene las claves públicas utilizadas durante el proceso de autentificación pública.