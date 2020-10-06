---
title: "Raid5"
date: 2020-10-05T20:01:02+02:00
draft: true
---

# Raid 5

## ¿Qué es RAID 5?

Un RAID 5 (también llamado distribuido con paridad) es una división de datos a nivel de bloques que distribuye la información de paridad entre todos los discos miembros del conjunto. El RAID 5 ha logrado popularidad gracias a su bajo coste de redundancia. Generalmente, el RAID 5 se implementa con soporte hardware para el cálculo de la paridad. RAID 5 necesitará un mínimo de 3 discos para ser implementado. 

## Ejercicios a realizar

A continuación vamos a ver que ejercicios debemos de realizar para practicar con los RAID 5, para ello vamos a usar una maquina creada con vagrant y que va a usar Debian 10, nuestro RAID 5 debe de disponer de 2GB de espacio de almacenamiento por lo que usaremos discos virtuales de 1GB.

* Tarea 1: Crea una raid llamado md5 con los discos que hemos conectado a la máquina. ¿Cuantos discos tienes que conectar? ¿Qué diferencia existe entre el RAID 5 y el RAID1?

* Tarea 2: Comprueba las características del RAID. Comprueba el estado del RAID. ¿Qué capacidad tiene el RAID que hemos creado?

* Tarea 3: Crea un volumen lógico (LVM) de 500Mb en el raid 5.

* Tarea 4: Formatea ese volumen con un sistema de archivo xfs.

* Tarea 5: Monta el volumen en el directorio /mnt/raid5 y crea un fichero. ¿Qué tendríamos que hacer para que este punto de montaje sea permanente?

* Tarea 6: Marca un disco como estropeado. Muestra el estado del raid para comprobar que un disco falla. ¿Podemos acceder al fichero?

* Tarea 7: Una vez marcado como estropeado, lo tenemos que retirar del raid.

* Tarea 8: Imaginemos que lo cambiamos por un nuevo disco nuevo (el dispositivo de bloque se llama igual), añádelo al array y comprueba como se sincroniza con el anterior.

* Tarea 9: Añade otro disco como reserva. Vuelve a simular el fallo de un disco y comprueba como automática se realiza la sincronización con el disco de reserva.

* Tarea 10: Redimensiona el volumen y el sistema de archivo de 500Mb al tamaño del raid.

### Tarea 1

En la tarea 1 debemos de crear el RAID 5 con 3 discos de 1GB en nuestra maquina virtual que hayamos levantado con vagrant, para ello usamos el siguiente codigo en Vagrantfile:

```ruby
# -*- mode: ruby -*- 
# vi: set ft=ruby : 

Vagrant.configure("2") do |config| 
	disco1 = '.vagrant/midisco1.vdi' 
	disco2 = '.vagrant/midisco2.vdi' 
	disco3 = '.vagrant/midisco3.vdi' 
	config.vm.define :raid5 do |raid5| 
		raid5.vm.box = "debian/buster64" 
		raid5.vm.hostname = "raid5" 
		raid5.vm.provider :virtualbox do |v| 
			v.customize ["createhd", "--filename", disco1, "--size", 1024] 
			v.customize ["storageattach", :id, "--storagectl", "SATA 				Controller", "--port", 1, "--device", 0, "--type", "hdd", "–			medium", disco1] 
			end 
		raid5.vm.provider :virtualbox do |e| 
			e.customize ["createhd", "--filename", disco2, "--size", 1024] 
			e.customize ["storageattach", :id, "--storagectl", "SATA 				Controller", "--port", 2, "--device", 0, "--type", "hdd", "–			medium", disco2] 
			end 
		raid5.vm.provider :virtualbox do |r| 
			r.customize ["createhd", "--filename", disco3, "--size", 1024] 
			r.customize ["storageattach", :id, "--storagectl", "SATA 				Controller", "--port", 3, "--device", 0, "--type", "hdd", "–			medium", disco3] 
			end 
	end 
end
```
Una vez tengamos el anterior código en nuestro fichero de configuración de Vagrantfile, lo que debemos de hacer es levantar nuestra maquina y comprobar que tenemos 3 discos de 1GB cada uno con los que conseguiremos el RAID 5 de 2GB ya que, como sabemos, los RAID 5 guardan datos de paridad sobre los demás discos en su almacenamiento por lo que se pierde uno de los discos como almacenamiento.

Para poder crear nuestro RAID5 debemos de tener en nuestro ordenador instalado el paquete de mdadm, para instalarlo usamos el siguiente comando:

```shell
vagrant@raid5:~$ sudo apt install mdadm
```

Una vez instalado el paquete de mdadm, solo debemos elegir los discos que vamos a usar en nuestro RAID5, que mínimo deben ser 3, para ver los discos que disponemos podemos usar el comando:

* lsblk: Visualiza los dispositivos, unidades, particiones y sus capacidades (estén montadas o no las unidades).

* fdisk -l: listar todas las particiones existentes en nuestro sistema.

Una vez tengamos localizados los discos que vamos a usar, en mi caso usare los disco sdb, sdc y sdd:

* Formato largo:
```shell
vagrant@raid5:~$ sudo mdadm --create /dev/md5 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
```

* Formato corto:
```shell
vagrant@raid5:~$ sudo mdadm -C /dev/md5 -l=5 -n=3 /dev/sdb /dev/sdc /dev/sdd
```

#### ¿Cual es la diferencia entre RAID1 y RAID5?

Raid1 es tener en espejo 2 discos, los 2 contienen la misma información y se replican entre ellos, esto lo puedes hacer por software.

Raid5 es tener 3 o + discos y te ofrece redundancia de datos y con menos espacio perdido cosa que con RAID1 pierdes la mitad del disco.

### Tarea 2

Ahora vamos a comprobar el estado en el que se encuentra el raid5 que hemos creado, y la capacidad que tiene este raid5. Para poder ver la información que tiene nuestro raid 5 creado en el apartado anterior podemos hacerlo de dos formas distintas:

* Mediante el fichero mdstat podemos ver el estado en el que se encuentran los raid software que tengamos en nuestro equipo o equipo remoto:

```shell
vagrant@raid5:~$ cat /proc/mdstat
```

* Mediante un comando del paquete mdadm también podemos ver informacion mas detallada de un raid en concreto:

```shell
vagrant@raid5:~$ sudo mdadm -D /dev/md5
```

Si nos fijamos bien en la información que hemos obtenido con este ultimo comando nos daremos cuenta que el tamaño total de nuestro raid es de 2GB.

### Tarea 3

Ahora vamos crear dentro de nuestro raid5 un volumen logico de 500MB, para ello lo primero que debemos de hacer es tener instalado en nuestro equipo el paquete lvm2 que contiene las herramientas necesarias para poder crear volúmenes logicos.

* Instalamos el paquete lvm2:

```shell
vagrant@raid5:~$ sudo apt install lvm2
```

* Una vez instalado el paquete de lvm2 lo que debemos de hacer es crear un grupo de volúmenes donde incluyamos a nuestro raid5, para ello el grupo de volúmenes se llamara “prueba” e incluirá a nuestro raid5 llamado “md5”:

```shell
vagrant@raid5:~$ sudo vgcreate prueba /dev/md5
```

* Una vez creado el grupo de volúmenes llamado “prueba”, lo que debemos hacer es crear nuestro volumen lógico de 500MB al que llamaremos “storage”:

```shell
vagrant@raid5:~$ sudo lvcreate prueba -L 500M -n storage
```

De esta forma, si realizamos el comando “lsblk -f” nos mostrara a información de los discos y particiones del sistema y podremos ver como dentro del raid5 “md5” que habíamos creado tenemos un grupo de volúmenes llamado “prueba” y que dentro de este grupo de volúmenes tenemos un volumen lógico llamado “storage”.

### Tarea 4

En la siguiente tarea debemos darle un formato a nuestro volumen lógico que hemos creado, para ello vamos a darle el formato xfs, para poder dar este formato necesitamos tener instalado el paquete “xfsprogs”:

* Instalamos xfsprogs:

```shell
vagrant@raid5:~$ sudo apt install xfsprogs
```

Una vez instalado debemos usar el comando “mkfs” para poder formatear el volumen logico “storage” que hemos creado, para ello usaremos el siguiente comando:

```shell
vagrant@raid5:~$ sudo mkfs.xfs -f /dev/prueba/storage
```

Ahora si hacemos un “lsblk -f” veremos que el formato que tiene el volumen “storage” es el de xfs por lo que ya tendríamos el volumen formateado.

### Tarea 5

Para esta tarea lo que debemos de hacer es montar el volumen “storage” en el directorio mnt/raid5, para ello debemos de tener creado en el directorio /mnt la carpeta “raid5”, una vez creada dicha carpeta lo que debemos de hacer es montar este volumen en ese directorio:

* Primero creamos la carpeta en el directorio /mnt:

```shell
vagrant@raid5:~$ sudo mkdir /mnt/raid5
```

* Ahora debemos de montar el volumen en el directorio creado y dentro de este crear un fichero vacío que llamaremos “hola.txt”

```shell
    vagrant@raid5:~$ sudo mount -t xfs /dev/prueba/storage /mnt/raid5/

    y creamos el fichero “hola.txt”

    vagrant@raid5:~$ sudo touch /mnt/raid5/hola.txt
```

Llegados a este punto, si queremos que este volumen se monte automáticamente cuando iniciemos el sistema de nuevo, tenemos que modificar el fichero fstab que es el fichero que tiene nuestro sistema para leer los volúmenes, particiones o discos que tiene que montar al inicio. Por lo que para montar nuestro volumen debemos de tener los siguientes elementos:

* El uuid del volumen a montar, para ello vamos a usar el siguiente comando:

```shell
vagrant@raid5:~$ sudo blkid /dev/mapper/prueba-storage
```

* Necesitaremos también el punto de montaje, que en este caso sera /mnt/raid5

* necesitaremos también el tipo de formato que tiene, que es xfs

* Las opciones que pondremos serán las defaults

* dump y pass lo dejaremos en 0

En ese caso deberia de quedar de la siguiente forma en el fichero **/etc/fstab**:

```shell
UUID=84f5ed91-538a-4764-b123-624c797bb0dc	/mnt/raid5	xfs	defaults	0	0
```

### Tarea 6

Ahora debemos ver como actúa nuestro raid 5 cuando nosotros marcamos un disco como estropeado, para poder marcar un disco como estropeado ejecutamos el siguiente comando, en mi caso marcare como estropeado el disco “sdc”:

```shell
vagrant@raid5:~$ sudo mdadm -f /dev/md5 /dev/sdc
```

Vemos que no ha ocurrido nada, pero si nos vamos al comando que nos muestra los detalles de nuestro radi5, veremos lo siguiente:

Donde vemos que donde dice “failed devices” nos marca como que uno de los disco esta estropeado y solo funcionan 2 de los 3 que tenemos. Y veremos que si queremos entrar en el fichero que hemos creado anteriormente veremos que nos deja verlo.

### Tarea 7

Como tenemos el disco “sdc” estropeado lo que debemos de hacer ahora es retirarlo de nuestro raid, para ello debemos usar el siguiente comando:

```shell
vagrant@raid5:~$ sudo mdadm --remove /dev/md5 /dev/sdc
```

Y nos saldrá el mensaje de que se ha retirado en caliente nuestro disco dañado, si nos vamos a la información del raid veremos que solo nos aparece que tiene 2 discos.

### Tarea 8

Ahora que hemos sacado un disco de nuestro raid, vamos a volver a añadirlo para así poder ver como se sincroniza con los otros 2 discos que tenemos en el raid, para ello solo debemos localizar el disco que vayamos a introducir en el raid, en este caso sera el disco “sdc” que es el disco que hemos sacado anteriormente, para ello usamos el siguiente comando:

```shell
vagrant@raid5:~$ sudo mdadm --add /dev/md5 /dev/sdc
```

Ahora si queremos ver como se sincroniza solo debemos de ver los detalles del raid y veremos como nos aparece que 1 disco esta en “repuesto” y se esta sincronizando con los otros dos discos de nuestro raid:

### Tarea 9

Hemos visto que, si eliminamos uno de los discos de nuestro raid, luego podemos poner otro pata que este se sincronice con los que tenemos en nuestro raid. En situaciones de raid que este en producción no se puede dejar solo los discos que tiene el raid y ya, sino que tenemos que asegurarnos que si uno de los discos falla, tenga otro de respaldo para así poder continuar en producción, a estos disco se le llamo discos de respaldo. Para añadir un disco de respaldo necesitamos otro disco mas en nuestra maquina. Una vez lo tengamos solo debemos de añadirlo a nuestro raid como hemos hecho anteriormente:

```shell
vagrant@raid5:~$ sudo mdadm --add /dev/md5 /dev/sde
```

Aquí vemos como ahora tenemos un disco de respaldo añadido a nuestro raid.  Ahoara vamos a probar a marca un disco como estropeado, por ejemplo el disco “sdd”, para ello lo que debemos hacer es el mismo comando que hemos hecho anteriormente:

```shell
vagrant@raid5:~$ sudo mdadm -f /dev/md5 /dev/sdd
```

Y ahora si nos vamos a los detalles del disco veremos como automáticamente el disco de reserva a sustituido al disco que ha fallado:

### Tarea 10

Ahora solo debemos de redimensionar nuestro volumen lógico con el siguiente comando.

```shell
vagrant@raid5:~$ sudo lvextend -L +1.5G /dev/mapper/prueba-storage
```
