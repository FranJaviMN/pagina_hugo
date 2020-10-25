---
title: "Compilacion Programa"
date: 2020-10-25T12:23:58+01:00
draft: true
---

# Compilación de un programa en C utilizando un Makefile

En la siguiente tarea vamos a realizar la compilacion de un programa en c usando la herramienta *make*, para ello vamos a usar dos programas para hacer la compilacion: *htop* y *nano*

## Compilando el programa nano

Primero vamos a proceder a la compilacion del programa nano, nano es un editor de texto fácil de usar. Nano trata de emular la funcionalidad y la interfaz de fácil manejo de Pico, pero sin la integración con Pine. 

Ahora vamos a ir con los pasos necesarios para poder realizar la compilacion y la instalacion:

1. Descargamos el codigo fuente del programa *nano* mediante este [enlace](http://deb.debian.org/debian/pool/main/n/nano/nano_3.2.orig.tar.xz). El anterior enlace nos va a descargar un fichero comprimido en **tar.xz** por lo que debemos descomprimir el fichero con el comando:
```shell
#### Descargamos el codigo fuente de nano ####
vagrant@buster:/usr/local/nano$ sudo wget http://deb.debian.org/debian/pool/main/n/nano/nano_3.2.orig.tar.xz

#### Descomprimimos el fichero y entramos en el directorio que se ha creado ####
tar xfvJ nano_3.2.orig.tar.xz nano-3.2/
```

2. Una vez descomprimidos nos dirigimos a la carpeta que nos ha generado y buscamos el fichero llamado *README* o *INSTALL* y lo leemos. Una vez lo hayamos leido veremos que nos da las directrices para poder compilar e instalar el programa, asi que seguimos los pasos indicados.

3. Siguiendo los pasos del fichero *README* o *INSTALL* debemos de realizar las siguientes acciones:
```shell
#### Usamos ./configure ####

vagrant@maquina:~/nano-3.2$ ./configure

Puede que nos de algun error de que nos falta algun paquete para realizar la anterior accion, en mi caso el paquete que me faltaba era el libncursesw5-dev por lo que lo instalamos:

vagrant@maquina:~/nano-3.2$ apt install libncursesw5-dev

#### Usamos la herramienta make ####

vagrant@maquina:~/nano-3.2$ make

#### Ahora instalamos el programa, en este caso se instalara en el directorio /usr/local/share/doc/nano ####

vagrant@maquina:~/nano-3.2$ sudo make install
```

4. Si queremos desinstalar este binario que hemos compilado e instalado debemos de nuevo seguir los pasos que nos da el documento *README* o *INSTALL*:

* Si queremos borrar todos los objetos creados por la compilacion del programa usamos:
```shell
vagrant@buster:/usr/local/nano/nano-3.2$ sudo make clean
```

* Si queremos borrar los archivos de configuracion que ha creado el script ***configure***:
```shell
vagrant@buster:/usr/local/nano/nano-3.2$ sudo make distclean
```

* Si simplemente queremos desinstalar el programa que hemos instalado con **make install** usamos:
```shell
vagrant@buster:/usr/local/nano/nano-3.2$ sudo make uninstall
```


## Compilando el programa htop

Ahora vamos a realizar la compilacion del programa htop que se realiza de manera muy similar a la de nano.

1. Al igual que con nano, debemos descargarnos el codigo fuente del programa htop en el siguiente [enlace](http://deb.debian.org/debian/pool/main/h/htop/htop_2.2.0.orig.tar.gz), una vez descargado debemos de descomprimir el fichero que nos ha descargado con el comando:
```shell
#### Descargar el codigo fuente del paquete ####

vagrant@buster:/usr/local/htop/htop-2.2.0$ sudo wget http://deb.debian.org/debian/pool/main/h/htop/htop_2.2.0.orig.tar.gz

#### Descomprimimos el fichero ####

vagrant@buster:/usr/local/htop$ sudo tar -xvzf htop_2.2.0.orig.tar.gz
```

2. Una vez lo hayamos descomprimido buscamos el fichero *README* y el fichero *INSTALL* en el cual tendremos las directrices para la compilacion y la instalacion del programa htop.

3. Siguiendo los pasos del fichero *README* e *INSTALL* debemos de realizar las siguientes acciones:
```shell
#### Usamos ./configure ####

vagrant@buster:/usr/local/htop/htop-2.2.0$ sudo ./configure

En este caso no he tenido que instalar ningun paquete adicional

#### Usamos la herramienta make ####

vagrant@buster:/usr/local/htop/htop-2.2.0$ sudo make

#### Instalamos el programa en /usr/local/bin/ ####

vagrant@buster:/usr/local/htop/htop-2.2.0$ sudo make install
```

4. Si queremos desinstalar este binario que hemos compilado e instalado debemos de nuevo seguir los pasos que nos da el documento *README* o *INSTALL*:

* Si queremos borrar todos los objetos creados por la compilacion del programa usamos:
```shell
vagrant@buster:/usr/local/htop/htop-2.2.0$ sudo make clean
```

* Si queremos borrar los archivos de configuracion que ha creado el script ***configure***:
```shell
vagrant@buster:/usr/local/htop/htop-2.2.0$ sudo make distclean
```

* Si simplemente queremos desinstalar el programa que hemos instalado con **make install** usamos:
```shell
vagrant@buster:/usr/local/htop/htop-2.2.0$ sudo make uninstall
```