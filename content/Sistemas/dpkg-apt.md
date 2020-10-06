---
title: "Dpkg Apt"
date: 2020-10-05T18:13:34+02:00
draft: true
---

# Ejercicios de dpkg y APT
En primer lugar prepara una máquina virtual con Debian buster, puedes hacerlo de la forma que prefieras o usando el fichero Vagrantfile que se proporciona.

La versión de debian buster a fecha de hoy, es la versión 10.6. Sobre una máquina de versión anterior realizar las siguientes acciones:

## Trabajo con apt, aptitude, dpkg

1. ¿Qué acciones consigo al realizar apt update y apt upgrade? Explica detalladamente.

Al realizar el comando de **apt update** lo que estamos haciendo no es una actualizacion de los paquetes, sino que descarga la lista de paquetes desde los repositorios que tenemos indicados en el fichero ***/etc/apt/sources.list*** y "actualiza" la informacion de las nuevas versiones de los paquetes y las dependencias de estos. 

Cuando usamos el comando **apt upgrade** lo que estamos haciendo el leer la lista de paquetes que hemos descargado con **apt update** para proceder a, o bien actualizar un paquete que tengamos instalado en nuestro sistema o instalar una dependencia nueva del paquete para que este funcione.

2. Lista la relación de paquetes que pueden ser actualizados. ¿Qué información puedes sacar a tenor de lo mostrado en el listado?.

Si en nuestro sistema, despues de realizar un **apt update**, queremos ver la lista de los paquetes que pueden ser actualizados solo debemos de usar el comando **apt list --upgradable** y si queremos que la salida del comando la podamos ver paginada solo debemos de usar:

    apt list --upgradable | less

De esta forma veremos la informacion paginada. Cuando realizamos el comando y vemos su salida podemos ver los paquetes que se van a actualizar en nuestro sistema indicandonos la version que tenemos en nuestro sistema y la version que se va a instalar.

3. Indica la versión instalada, candidata así como la prioridad del paquete openssh-client.

Para poder ver la version que tenemos de algun paquete asi como de la version candidata que existe para instalar y su prioridad en el sistema debemos de introducir el comando:

    apt-cache policy (nombre del paquete)

en este caso seria el siguiente comando:

    apt-cache policy openssh-client

De esta forma veremos si el paquete que estamos consultando esta instalado en nuestra sistema y, si esta instalado como si no lo esta, nos indicara el paquete candidato a instalar que sera la version mas actual que se encuentre.

4. ¿Cómo puedes sacar información de un paquete oficial instalado o que no este instalado?

Si queremos ver la informacion de un paquete, este instalado o no, debemos usar el comando:


    apt-cache show (nombre del paquete)

De esta forma veremos el nombre del paquete, la version que tenemos en nuestra lista de paquetes, los que mantienen el paquete y mucha mas informacion acerca del paquete consultado.


7. todo el contenido referente al paquete openssh-client actual de tu máquina. Utiliza para ello tanto dpkg como apt.

Si queremos ver el contenido referente a cualquier paquete podemos usar tanto **dpkg** como **apt**.

    Con dpkg: dpkg -L (nombre del paquete)
    Con apt: apt-file list (nombre del paquete)
    apt-file puede que requiera una instalacion: apt install apt-file y luego apt-file update.

8. Listar el contenido de un paquete sin la necesidad de instalarlo o descargarlo.

Para ver el contenido de paquetes no instalados o no descargados podemos usar el comando:

    apt-list
    
De esta forma podremos ver el contenido del paquete, aunque como se ha dicho en el apartado anterior, **apt-file** puede que necesite una instalacion y una actualizacion de su lista de paquetes.

9. Simula la instalación del paquete openssh-client.

Para la instalacion de un paquete lo podemos hacer con el comando:
    
    apt install openssh-client

o lo podemos hacer con 

    aptitude install openssh-client

10. ¿Qué comando te informa de los posible bugs que presente un determinado paquete?

Para poder ver los posibles bugs que puede tener un paquete que tengamos en meten instalar, podemos usar el comando:         
    
    apt-listbugs list (nombre del paquete)

11. Después de realizar un apt update && apt upgrade. Si quisieras actualizar únicamente los paquetes que tienen de cadena openssh. ¿Qué procedimiento seguirías?. Realiza esta acción, con las estructuras repetitivas que te ofrece bash, así como con el comando xargs.

12. ¿Cómo encontrarías qué paquetes dependen de un paquete específico?

Para encontrar las despendecias de un paquete en concreto que podemos usar el comando:

    apt-cache depends (nombre del paquete)

Cuando realizamos el comando apt-cache depends nos mostrara por pantalla los paquetes que dependen del paquete del que hemos hecho la busqueda como los paquetes que se recomiendan instalar y con aquellos paquetes con los que tiene conflicto.

13. Como procederías para encontrar el paquete al que pertenece un determinado archivo.

Para realizar la busqueda de un paquete mediante un fichero de este paquete solo debemos usar el comando:

    apt-file search (nombre del fichero del paquete en cuestion)

14. ¿Que procedimientos emplearías para eliminar liberar la cache en cuanto a descargas de paquetería?

Para poder liberar la cache que se genera cuando realizamos un **apt update**, solo debemos de realizar el siguiente comando:

    apt autoclean -> Elimina los paquetes obsoletos del cache de apt

    apt clean -> Elimina todo el cache de apt

## Trabajo con ficheros .deb

1. Descarga un paquete sin instalarlo, es decir, descarga el fichero .deb correspondiente. Indica diferentes formas de hacerlo.

Para poder descargar un fichero .deb podemos hacerlo de una forma grafica descargando el fichero desde la pagina o repositorio que indiquemos. Tambien podemos descargar el paquete que nosotros indiquemos desde los repositorios debian que tengamos asignados y con todas sus dependencias, para ello solo debemos usar el siguiente comando:

    apt install --download-only (nombre del paquete) -> Este fichero se guardara en /var/cache/apt/archives

2. ¿Cómo puedes ver el contenido, que no extraerlo, de lo que se instalará en el sistema de un paquete deb?

Para poder listar el contenido de un archivo .deb podemos usar **dpkg** con el siguiente comando:

    dpkg -c (nombre del fichero .deb) -> Vemos su interior sin descomprimirlo

o tambien podemos usar el comando **ar**

    ar t (nombre del fichero .deb)


3. Sobre el fichero .deb descargado, utiliza el comando ar. ar permite extraer el contenido de una paquete deb. Indica el procedimiento para visualizar con ar el contenido del paquete deb. Con el paquete que has descargado y utilizando el comando ar, descomprime el paquete. ¿Qué información dispones después de la extracción?. Indica la finalidad de lo extraído.

Si no tubieramos la utilidad **ar** en nuestro sistema solo debemos ejecutar el siguiente comando para poder tenerla disponible:

    apt install binutils 

Una vez instalado la utilidad **ar** solo debemos de mirar su documentacion para poder ver la forma de ver el contenido de un fichero .deb y la forma de descomprimirlos:

    ar t (nombre del fichero .deb) -> Vemos el interiro del fichero .deb

    ar x (nombre del fichero .deb) -> Descomprimimos el fichero .deb

De esta forma, al descomprimir el fichero .deb nos apareceran 3 nuevos ficheros, donde:

    debian-binary -> es un archivo de texto plano que indica la versión del archivo .deb.

    control.tar.gz -> contiene toda la metadata disponible. Entre otras cosas, la información contenida en este archivo indica si se puede instalar el paquete en cuestión basándose en las dependencias necesarias y en su presencia en el sistema.

    data.tar.gz -> contiene todos los archivos que serán extraídos por dpkg. Esto incluye archivos ejecutables, el man page, y todo lo relacionado con el paquete.


4. Indica el procedimiento para descomprimir lo extraído por ar del punto anterior. ¿Qué información contiene?

Para poder descomprimir lo anteriormente descomprimido con **ar**, debemos usar la utilidad **tar** que nos permitira descomprimir los ficheros tar.xz, para ello usamos el siguiente comando:

    tar -xvf (fichero a descomprimir)
    donde:
    -x -> Extraer el archivo
    -v -> Muestra una descripción detallada del progreso de la compresión
    -f -> Nombre del archivo

## Trabajo con repositorios

1. Añade a tu fichero sources.list los repositorios de backports y sid.

Para ello debemos de dirigirnos hacia el fichero /etc/apt/source.list con privilegios de root para poder modificar este fichero, una vez estemos con el fichero delante debemos de poner las siguientes lineas:

    deb http://deb.debian.org/debian buster-backports main -> repositorios de backports

    deb http://deb.debian.org/debian sid main -> repositorios sid

2. ¿Cómo añades la posibilidad de descargar paquetería de la arquitectura i386 en tu sistema. ¿Que comando has empleado?. Lista arquitecturas no nativas. ¿Cómo procederías para desechar la posibilidad de descargar paquetería de la arquitectura i386?

En nuestro sistema podemos tener paqueteria de varias arquitecturas, si nosotros queremos añadir mas arquitecturas aparte de la nuestra debemos de realizar el siguiente comando, en mi caso añadire la arquitectura i386:

    dpkg –add-architecture (arquitectura)

    dpkg --add-architecture i386 --> Añadimos la arquitectura i386

Si queremos ver las arquitecturas no nativas que tenemos en nuestro sistema usamos el siguiente comando:

    dpkg --print-foreign-architectures -> Nos muestra las arquitecturas no nativas

Si tuvieramos que desechar la descarga de paqueteria de i386:

    dpkg –remove-architecture i386


4. Indica el procedimiento para descargar un paquete del repositorio stable.

    apt install -t stable (nombre paquete)

5. Indica el procedimiento para descargar un paquete del repositorio de backports.

    apt install -t backports (nombre paquete)

6. Indica el procedimiento para descargar un paquete del repositorio de sid.

    apt install -t sid (nombre paquete)

7. Indica el procedimiento para descargar un paquete de arquitectura i386.

    apt install (nombre paquete):(nombre arquitectura)

## Trabajo con directorios

1. Que cometidos tienen:

    /var/lib/apt/lists/ -> 

    /var/lib/dpkg/available ->

    /var/lib/dpkg/status ->

    /var/cache/apt/archives/ ->


