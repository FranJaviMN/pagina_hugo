---
title: "Fs Avanzados"
date: 2021-01-13T09:50:19+01:00
draft: true
---

# Sistemas de ficheros avanzados

En este ejercicio vamos a realizar pruebas con los sistemas de ficheros "avanzados" como pueden ser **Btrfs** o **ZFS**, para ello debemos de elegir uno de ellos pero primero vamos a ver un poco de la historia de ambos sistemas de ficheros y, posteriormente, veremos ejemplos de uso de estos sistemas de ficheros y de sus avanzadas caracteristicas.

## Btrfs

**Btrfs** es un sistema de ficheros copy-on-write (CoW) para Linux enfocado a implementar características novedosas y avanzadas y, a la vez, centrado en la tolerancia a fallos, reparación y facilidad de administración. Desarrollado conjuntamente por Oracle, Red Hat, Fujitsu, Intel, SUSE, STRATO, y muchos más, Btrfs está publicado bajo la licencia GPL y se encuentra abierto a contribuciones por parte de cualquier persona.

Algunas distribuciones ya han comenzado a cambiar en sus últimos lanzamientos y han empezado a incluir este sistema de ficheros. Btrfs tiene un gran número de características avanzadas en común con ZFS, las cuales hicieron que ZFS fuera tan popular entre las distribuciones de BSD y los dispositivos NAT. A continuacion podemos ver algunas de las caracteristicas avanzadas que tiene este sistema de ficheros:

* Copy on Write (CoW) y snapshotting: Facilitar la creación de backups incluso desde un sistema de ficheros "en caliente" o de una máquina virtual

* Checksums a nivel de fichero: Los metadatos de cada fichero incluyen un checksum que es usado para detectar y reparar errores.

* Compresión: Los ficheros pueden estar comprimidos o descomprimidos en vuelo, lo que aumenta la velocidad de lectura.

* Subvolumenes: Los sistemas de ficheros pueden compartir un determinado espacio en vez de tener que utilizar sus propias particiones.

* RAID: Btrfs tiene sus propias implementaciones de RAID de manera que LVM o mdadm no son necesarios para tener un RAID. De momento RAID 0 y 1 están soportados; RAID 5 y 6 están de camino.

* Deduplicación de datos: El soporte de deduplicación de datos es limitado; sin embargo, eventualmente se convertirá en una característica estándar en Btrfs. Ésto permite reducir el espacio ocupado mediante la comparación binaria de ficheros.

### Instalacion en Debian 10

Para la instalacion de este sistema de ficheros en un sistema Debian 10, lo unico que debemos de hacer es instalar el paquete **btrfs-tools** y de esta forma ya tendremos disponibles sus distintas caracteristicas y podremos usarlo:
```shell
#### Instalacion del paquete ####
debian@fs-avanzado:~$ sudo apt install btrfs-tools
```

## ZFS

**ZFS** es un sistema de archivos y administrador de volúmenes desarrollado originalmente por Sun Microsystems para su sistema operativo Solaris. El significado original era 'Zettabyte File System'.

El anuncio oficial de ZFS se produjo en septiembre de 2004. El código fuente del producto final se integró en la rama principal de desarrollo de Solaris el 31 de octubre de 2005 y fue lanzado el 16 de noviembre de 2005 como parte de la compilación 27 de OpenSolaris. 

El sistema de archivos ZFS se utiliza de forma nativa en sistemas operativos basados en FreeBSD, como los populares FreeNAS o XigmaNAS, dos sistemas operativos que están orientados específicamente para servidores NAS. 

EL sistema de ficheros de ZFS incluye algunas caracteristicas muy interesantes que, si bien muchas de ellas las comprarte con Btrfs, es buen mencionarlas tambien aqui:

* Copy on Write (CoW) y Snapshots: Facilitar la creación de backups incluso desde un sistema de ficheros "en caliente" o de una máquina virtual

* Verificar la integridad de los ficheros y autoreparacion: Esta función hace que, a la hora de escribir nuevos datos en el sistema de ficheros de ZFS, se guarda una suma de comprobacion para esos nuevos datos. Cuando estos datos son leidos, se verifica la suma de comprobacion, en caso de que esta se erronea, ZFS detectara el error e intentara corregirlo.

* RAID-Z: ZFS puede manejar RAID sin requerir ningún software o hardware adicional. Como era de esperar, ZFS tiene su propia implementación de RAID: RAID-Z. Para utilizar el nivel básico de RAID-Z necesitamos lo siguiente:

    * RAID-Z1: Necesita al menos dos discos para almacenamiento y uno para paridad.
    * RAID-Z2: Requería al menos dos unidades de almacenamiento y dos unidades para la paridad.
    * RAID-Z3: Requiere al menos dos unidades de almacenamiento y tres unidades para la paridad.

* Compresión: Los ficheros pueden estar comprimidos o descomprimidos en vuelo, lo que aumenta la velocidad de lectura.


### Instalacion en Debian 10

Para realizar la instalacion del sistema de ficheros de ZFS en nuestro sistema Debian 10 lo que debemos de hacer es instalar el paquete llamado **zfs-dkms**, para ello lo que vamos a hacer es añadir a nuestros repositorios la rama llamada **contrib non-free** ya que ZFS tiene una licencia distinta a la que tiene linux, por lo que debemos descargarlo desde esa rama:
```shell
#### Fichero /etc/apt/source.list ####
deb http://deb.debian.org/debian/ buster main contrib non-free
#deb-src http://deb.debian.org/debian/ buster main
deb http://deb.debian.org/debian/ buster-updates main
#deb-src http://deb.debian.org/debian/ buster-updates main
deb http://security.debian.org/debian-security buster/updates main
#deb-src http://security.debian.org/debian-security buster/updates main
```

Despues de añadir la rama **contrib non-free** debemos de instalar los paquetes de los encabezados del núcleo:
```shell
#### Instalamos los paquetes que necesitamos ####
sudo apt install linux-headers-`uname -r`
```

Una vez hecho esto ahora si instalamos los paquetes relacionados con el sistema de ficheros ZFS, para ello instalamos el paquete llamado **zfsutils-linux** y apt instalará las dependencias necesarias, debemos de tener en cuenta de que construir el módulo del núcleo DKMS puede llevar un poco de tiempo. Cuando se completa, el sistema de archivos ZFS está listo para usar.
```shell
#### Instalamos los paquetes necesarios ####
debian@fs-avanzado:~$ sudo apt install zfs-dkms

#### Una vez instalado activamos el modulo del kernel zfs ####
debian@fs-avanzado:~$ sudo modprobe zfs

debian@fs-avanzado:~$ sudo lsmod | grep zfs
zfs                  3293184  0
...

#### Ahora reiniciamos todos los servicios de zfs para empezar a usarlo ####
debian@fs-avanzado:~$ sudo systemctl restart zfs-import-cache
debian@fs-avanzado:~$ sudo systemctl restart zfs-import-scan
debian@fs-avanzado:~$ sudo systemctl restart zfs-mount
debian@fs-avanzado:~$ sudo systemctl restart zfs-share
```

Una vez hecho esto ya tendremos listo nuestro sistema de ficheros ZFS para poder usarlo.

# Ejercicios a realizar

De estos de sistemas de ficheros debemos de elegir uno con el que vamos a hacer las pruebas de su funcionalidad en un sistema debian 10, en mi caso voy a elegir el sistema de ficheros **ZFS** del cual hemos visto los pasos para su instalación en los pasos de arriba.

Para este ejercicio vamos a usar una maquina debian 10 en openstack a la cual le vamos a añadir 4 discos adicionales de 2GB cada uno, de tal forma que al realizar el comando **lsblk** nos debe de salir algo como lo siguiente:
```shell
#### Salida del comando lsblk ####
debian@fs-avanzado:~$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    254:0    0  10G  0 disk 
└─vda1 254:1    0  10G  0 part /
vdb    254:16   0   2G  0 disk 
vdc    254:32   0   2G  0 disk 
vdd    254:48   0   2G  0 disk 
vde    254:64   0   2G  0 disk
```

Una vez tengamos nuestros 4 discos asociados a nuestra maquina ya estamos listo para empezar a probar las funcionalidades que tiene ZFS.

## Creacion de POOL en ZFS

En el sistema de ficheros ZFS no es necesario que nuestros volumenes tengan particiones antes de crear nuestros sistema de ficheros ZFS ya que lo recomendado es poner el disco entero ya que este, automaticamente es capaz de crear una **tabla de particiones GPT**

A continuacion vamos a ver como crear los distintos pools de almacenamiento que nos permite crear ZFS con el comando llamado **zpool**, para ello vamos a seguir la siguiente estructura:
```shell
#### Comando de creacion de pools en ZFS ####
zpool create -m <punto-de-montaje> <nombre-del-pool> [raidz(2|3)|mirror] <discos-para-los-pools>

Donde podemos ver:
    * create: Es el comando con el cual le indicamos la creacion de un nuevo pool
    * -m: Le indicamos el punto de montaje, por defecto lo montara en la raiz /
    * [raidz(2|3)|mirror]: Le indicamos el tipo de pool a crear
    * <discos-para-los-pools>: Le indicamos el id de los discos que vamos a usar, ZFS recomienda el uso de los ids de los discos
```

Para poder sacar el id de los discos que tenemos en nuestro sistema vamos a usar el siguiente comando:
```shell
debian@fs-avanzado:~$ ls -lh /dev/disk/by-id/
```

En mi caso voy a usar el directorio donde se encuentra cada uno para asi tenerlos bien diferenciados.

### Creacion de un raid 1

Para crearlo debemos de seleccionar el tipo de pool llamado **mirror** el cual es similar a un raid1, para poder crearlo necesitamos como minimo 2 discos, uno de ellos usado como redundante, en mi caso usare los disco **vdb** y **vdc**:
```shell
#### Creamos el pool mirror con el nombre raid1 ####
debian@fs-avanzado:~$ sudo zpool create -m /mnt/ raid1 mirror /dev/vdb /dev/vdc

#### Comprobamos el estatus del pool ####
debian@fs-avanzado:~$ sudo zpool status -v
  pool: raid1
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	raid1       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    vdb     ONLINE       0     0     0
	    vdc     ONLINE       0     0     0

#### Comprobamos que este montado en /mnt/ ####
debian@fs-avanzado:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            235M     0  235M   0% /dev
tmpfs            49M  2.3M   47M   5% /run
/dev/vda1       9.9G  1.5G  7.9G  16% /
tmpfs           243M     0  243M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           243M     0  243M   0% /sys/fs/cgroup
tmpfs            49M     0   49M   0% /run/user/1000
raid1           1.9G     0  1.9G   0% /mnt

#### Borramos el pool que hemos creado para seguir con las pruebas ####
debian@fs-avanzado:~$ sudo zpool destroy raid1
```

### Creacion de un raid 5

Para ello necesitamos usar el pool de tipo **raidz1** y para este necesitamos como minimo 4 discos, uno de ellos usado como redundante, para ello usaremos los 4 discos, el ultimo que pongamos es el que quedara como redundante:
```shell
#### Creamos el pool con el raidz1 ####
debian@fs-avanzado:~$ sudo zpool create -m /mnt/ raid5 raidz1 /dev/vdb /dev/vdc /dev/vdd /dev/vde

#### Comprobamos el estado del pool ####
debian@fs-avanzado:~$ sudo zpool status -v
  pool: raid5
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	raid5       ONLINE       0     0     0
	  raidz1-0  ONLINE       0     0     0
	    vdb     ONLINE       0     0     0
	    vdc     ONLINE       0     0     0
	    vdd     ONLINE       0     0     0
	    vde     ONLINE       0     0     0
```

### Creacion de un raid 0

Para ello solo necesitamos los discos que nosotros queramos añadirle, en este caso no hace falta indicarle el tipo de pool, en mi caso vamos a ponerle 3 discos, que son vdb, vdc y vdd:
```shell
#### Creacion del raid0 ####
debian@fs-avanzado:~$ sudo zpool create -m /mnt/ raid0 /dev/vdb /dev/vdc /dev/vdd

#### Comprobamos el estado del pool ####
debian@fs-avanzado:~$ sudo zpool status -v
  pool: raid0
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	raid0       ONLINE       0     0     0
	  vdb       ONLINE       0     0     0
	  vdc       ONLINE       0     0     0
	  vdd       ONLINE       0     0     0
```

## Creacion de datasets

Para ello podemos usar los pools que hemos estado creando, yo creare uno desde el principio y lo voy a llamar prueba1 y va a ser de tipo **mirror**:
```shell
#### Creamos el pool ####
debian@fs-avanzado:~$ sudo zpool create prueba1 mirror /dev/vdb /dev/vdc

#### Creamos el nuevo dataset llamado prueba1/pruebaVOL ####
debian@fs-avanzado:~$ sudo zfs create -o mountpoint=/home/debian/pruebaVOL prueba1/pruebaVOL

#### Comprobamos que esta montado y funcional ####
debian@fs-avanzado:~$ df -h
Filesystem         Size  Used Avail Use% Mounted on
udev               235M     0  235M   0% /dev
tmpfs               49M  2.3M   47M   5% /run
/dev/vda1          9.9G  1.5G  7.9G  16% /
tmpfs              243M     0  243M   0% /dev/shm
tmpfs              5.0M     0  5.0M   0% /run/lock
tmpfs              243M     0  243M   0% /sys/fs/cgroup
tmpfs               49M     0   49M   0% /run/user/1000
prueba1            1.9G     0  1.9G   0% /prueba1
prueba1/pruebaVOL  1.9G     0  1.9G   0% /home/debian/pruebaVOL

debian@fs-avanzado:~$ sudo zfs list 
NAME                USED  AVAIL  REFER  MOUNTPOINT
prueba1             122K  1.86G    24K  /prueba1
prueba1/pruebaVOL    24K  1.86G    24K  /home/debian/pruebaVOL
```

Ahora vamos a crear otro dataset, pero esta vez con un tamaño de 1GB y lo vamos a formatearlo con ext4, para ello usamos los siguientes comandos:
```shell
#### Creamos el dataset llamado prueba1/pruebaEXT4 ####
debian@fs-avanzado:~$ sudo zfs create -s -V 1GB prueba1/pruebaEXT4

#### Lo formateamos con ext4 ####
debian@fs-avanzado:~$ sudo mkfs.ext4 /dev/zvol/prueba1/pruebaEXT4

#### Lo montamos en /mnt ####
debian@fs-avanzado:~$ sudo mount /dev/zvol/prueba1/pruebaEXT4 /mnt

#### Lo comprobamos y copiamos algo en el ####
debian@fs-avanzado:~$ df -h
Filesystem         Size  Used Avail Use% Mounted on
udev               235M     0  235M   0% /dev
tmpfs               49M  2.3M   47M   5% /run
/dev/vda1          9.9G  1.5G  7.9G  16% /
tmpfs              243M     0  243M   0% /dev/shm
tmpfs              5.0M     0  5.0M   0% /run/lock
tmpfs              243M     0  243M   0% /sys/fs/cgroup
tmpfs               49M     0   49M   0% /run/user/1000
prueba1            1.9G     0  1.9G   0% /prueba1
prueba1/pruebaVOL  1.9G     0  1.9G   0% /home/debian/pruebaVOL
/dev/zd0           976M  2.6M  907M   1% /mnt

#### Comprobamos el estado del pool con zfs ####
debian@fs-avanzado:~$ sudo zfs list
NAME                 USED  AVAIL  REFER  MOUNTPOINT
prueba1              297M  1.57G    24K  /prueba1
prueba1/pruebaEXT4   297M  1.57G   297M  -
prueba1/pruebaVOL     24K  1.57G    24K  /home/debian/pruebaVOL
```

## Snapshots

Como un buen sistema de ficheros avanzados que es ZFS, este tambien tiene soporte para **Snapshots** y para la gestion de estas, para ello, partiendo del anterior apartado que hemos hecho, vamos a crear una snapshot del dataset formateado en **ext4** que hemos creado y montado en /mnt, para ello vamos a usar el siguiente comando:
```shell
#### Creamos la Snapshot del volumen ####
debian@fs-avanzado:~$ sudo zfs snapshot prueba1/pruebaEXT4@2020-01-13

Nota: Debemos de ponerle un nombre despues de @, en mi caso la fecha de creacion.

#### Si queremos destruir la snapshot solo debemos de usar el siguiente comando ####
debian@fs-avanzado:~$ sudo zfs destroy prueba1/pruebaEXT4@2020-01-13
```

## Añadiendo un nuevo volumen a un pool ya creado

Tambien podemos incrementar el tamaño de un pool que tengamos ya creado si tenemos otro disco al que se lo podamos añadir, en mi caso vamos a crear un pool de tipo :
```shell
#### Buscamos el disco que queremos usar, en mi caso vdd ####
debian@fs-avanzado:~$ lsblk | grep vdd
vdd    254:48   0   2G  0 disk 

#### Lo añadimos al pool llamado prueba1 ####
debian@fs-avanzado:~$ sudo zpool add prueba1 /dev/vdd -f

#### Comprobamos que tenemos otro disco mas añadido ####
debian@fs-avanzado:~$ sudo zpool status 
  pool: prueba1
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	prueba1     ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    vdb     ONLINE       0     0     0
	    vdc     ONLINE       0     0     0
	  vdd       ONLINE       0     0     0
```

## Renombrar un pool ya existente

Para ello solo debemos de usar el siguiente comando, en mi caso vamos a ponerle el nombre de *rename1*:
```shell
#### Seleccionamos el pool que queremos cambiarle el nombre ####
debian@fs-avanzado:~$ sudo zpool status | grep pool
  pool: prueba1

#### Le cambiamos el nombre a rename1 ####
debian@fs-avanzado:~$ sudo zpool export prueba1

debian@fs-avanzado:~$ sudo zpool import prueba1 rename1
```

## Reemplazar un disco en un pool ya existente

Al igual que el uso de raid software de marcar los discos que estan fallando, con ZFS podemos hacer lo mismo y podemos reemplazarlo de forma manual, para ello vamos a marcar como no funcional el disco **vdb** y lo vamos a reemplazar por el disco **vde**, para ello vamos a seguir los siguientes pasos:
```shell
#### Marcamos como no disponible el disco vdb ####
debian@fs-avanzado:~$ sudo zpool offline prueba1 /dev/vdb

#### Comprobamos el pool ####
debian@fs-avanzado:~$ sudo zpool status 
  pool: prueba1
 state: DEGRADED
status: One or more devices has been taken offline by the administrator.
	Sufficient replicas exist for the pool to continue functioning in a
	degraded state.
action: Online the device using 'zpool online' or replace the device with
	'zpool replace'.
  scan: resilvered 297M in 0h0m with 0 errors on Wed Jan 13 08:03:32 2021
config:

	NAME        STATE     READ WRITE CKSUM
	prueba1     DEGRADED     0     0     0
	  mirror-0  DEGRADED     0     0     0
	    vdb     OFFLINE      0     0     0
	    vdc     ONLINE       0     0     0
	  vdd       ONLINE       0     0     0

#### Reemplazamos el disco por vde ####
debian@fs-avanzado:~$ sudo zpool replace prueba1 /dev/vdb /dev/vde

#### Comprobamos el proceso ####
debian@fs-avanzado:~$ sudo zpool status 
  pool: prueba1
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
	continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Wed Jan 13 08:06:44 2021
	297M scanned out of 297M at 42.4M/s, 0h0m to go
	297M resilvered, 99.84% done
config:

	NAME             STATE     READ WRITE CKSUM
	prueba1          DEGRADED     0     0     0
	  mirror-0       DEGRADED     0     0     0
	    replacing-0  DEGRADED     0     0     0
	      vdb        OFFLINE      0     0     0
	      vde        ONLINE       0     0     0  (resilvering)
	    vdc          ONLINE       0     0     0
	  vdd            ONLINE       0     0     0

#### Comprobamos el resultado final ####
debian@fs-avanzado:~$ sudo zpool status 
  pool: prueba1
 state: ONLINE
  scan: resilvered 297M in 0h0m with 0 errors on Wed Jan 13 08:06:52 2021
config:

	NAME        STATE     READ WRITE CKSUM
	prueba1     ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    vde     ONLINE       0     0     0
	    vdc     ONLINE       0     0     0
	  vdd       ONLINE       0     0     0

errors: No known data errors
```

## Cambiando el punto de montaje de un pool

Para ello, vemos que nuestro pool **prueba1** esta montado en un directorio en **/prueba1** pero lo vamos a querer montado en un directorio que vamos a crear en neustra carpeta de usuario, para ello debemos de usar el siguiente comando:
```shell
#### Creamos la nueva carpeta llamada nuevoMontaje ####
debian@fs-avanzado:~$ mkdir nuevoMontaje

#### Cambiamos el punto de montaje de el pool prueba1 ####
debian@fs-avanzado:~$ sudo zfs set mountpoint=/home/debian/nuevoMontaje prueba1

#### Comprobamos que se haya cambiado correctamente ####
debian@fs-avanzado:~$ sudo zfs list
NAME                 USED  AVAIL  REFER  MOUNTPOINT
prueba1              297M  3.55G    24K  /home/debian/nuevoMontaje
```

## Crear un nuevo volumen a partir del pool con encriptacion por contraseña

Para ello nosotros vamos a seguir usando el pool **prueba1**, pero esta vez vamos a crear un nuevo *dataset* el cual se va a llamar **prueba1/encriptado**, por lo que vamos a crearlo ya encriptado con una contraseña:
```shell
#### Creamos el dataset encriptado ####
sudo zfs create -o encryption=on -o keyformat=passphrase prueba1/encriptado

#### Comprobamos la encriptacion ####
```

## Ver informacion completa respecto a los pools y datasets

Si nosotros quisieramos ver toda la informacion respecto a algun pool o algun dataset en concreto para poder ver las opciones que tiene habilitada solo debemos de usar la opcion **get all** con la cual obtendremos toda la informacion del pool o el dataset que queramos:
```shell
#### Ver la informacion del pool prueba1 ####
debian@fs-avanzado:~$ sudo zfs get all prueba1
NAME     PROPERTY              VALUE                      SOURCE
prueba1  type                  filesystem                 -
prueba1  creation              Tue Jan 12 18:10 2021      -
prueba1  used                  297M                       -
prueba1  available             3.55G                      -
prueba1  referenced            24K                        -
prueba1  compressratio         1.00x                      -
...
prueba1  relatime              off                        default
prueba1  redundant_metadata    all                        default
prueba1  overlay               off                        default

#### Ver la informacion de un dataset en concreto ####
debian@fs-avanzado:~$ sudo zfs get all prueba1/pruebaEXT4 
NAME                PROPERTY              VALUE                  SOURCE
prueba1/pruebaEXT4  type                  volume                 -
prueba1/pruebaEXT4  creation              Tue Jan 12 18:21 2021  -
prueba1/pruebaEXT4  used                  297M                   -
prueba1/pruebaEXT4  available             3.55G                  -
prueba1/pruebaEXT4  referenced            297M                   -
prueba1/pruebaEXT4  compressratio         1.00x                  -
...
prueba1/pruebaEXT4  defcontext            none                   default
prueba1/pruebaEXT4  rootcontext           none                   default
prueba1/pruebaEXT4  redundant_metadata    all                    default

#### Si queremos ver el estado de los pools de forma rapida usamos lo siguiente ####
debian@fs-avanzado:~$ sudo zpool status -x
all pools are healthy
```

## Habilitar la compresion

Por defecto en los sistemas de ficheros ZFS la compresion viene deshabilitada por defecto, por lo que nosotros, manualmente, debemos de habilitar, para ello debemos de usar la opcion **set compression=on** en el pool que nosotros queramos:
```shell
#### Habilitamos la compresion en el pool prueba1 ####
debian@fs-avanzado:~$ sudo zfs set compression=on prueba1

#### Comprobamos que esta habilitado ####
debian@fs-avanzado:~$ sudo zfs get all prueba1 | grep compression
prueba1  compression           on                         local

#### Habilitamos la compresion en un dataset ####
debian@fs-avanzado:~$ sudo zfs set compression=on prueba1/pruebaEXT4

#### Comprobamos que esta habilitado ####
debian@fs-avanzado:~$ sudo zfs get all prueba1/pruebaEXT4 | grep compression
prueba1/pruebaEXT4  compression           on                     local
```

Para comprobar como funciona la compresion en ZFS, vamos a crear un fichero ya comprimido el cual lo vamos a crear en el dataset **prueba1/pruebaVOL** que debemos de recordad que ahora esta montado en el directorio **/home/debian/pruebaVOL**:
```shell
#### Creamos un fichero comprimido de /var/log y de /etc/ ####
debian@fs-avanzado:~$ sudo tar -cf /home/debian/pruebaVOL/prueba.tar /var/log/ /etc/

#### Comprobamos el tamaño del fichero .tar que hemos creado ####
debian@fs-avanzado:~$ ls -lh pruebaVOL/
total 923K
-rw-r--r-- 1 root root 3.9M Jan 13 09:52 prueba.tar

#### Ahora vemos lo que realmente ocupa con la compresion activada ####
debian@fs-avanzado:~$ sudo zfs list | grep pruebaVOL
prueba1/pruebaVOL    946K  3.55G   946K  /home/debian/pruebaVOL

#### Comprobamos la compresion que tiene ####
debian@fs-avanzado:~$ sudo zfs get compressratio prueba1/pruebaVOL 
NAME               PROPERTY       VALUE  SOURCE
prueba1/pruebaVOL  compressratio  4.27x  -
```

Como vemos el fichero que he creado pesa originalmente 4MB aproximadamente pero, con la compresion activada en el dataset, el espacio que ocupa en este es de solo 946K de espacion. Ademas con la opcion **compressratio** nos da el espacio que hemos salvado con la compresion.

## Habilitar la deduplicación

la deduplicación de datos es una técnica especializada de compresión de datos para eliminar copias duplicadas de datos repetidos. Como buen sistemas de ficheros "avanzados" que es ZFS, tambien incluye dicha opcion en los pools que, por defecto, viene desactivado. Para poder activar dicha opcion debemos de usar el siguiente comando:
```shell
#### Activamos la deduplicacion en prueba1 ####
debian@fs-avanzado:~$ sudo zfs set dedup=on prueba1

#### Comprobamos que esta habilitada ####
debian@fs-avanzado:~$ sudo zfs get all prueba1 | grep dedup
prueba1  dedup                 on                         local
```

## Scrubbing

La depuración de datos es una técnica de corrección de errores que utiliza una tarea en segundo plano para inspeccionar periódicamente la memoria principal o el almacenamiento en busca de errores, y luego corregir los errores detectados utilizando datos redundantes en forma de diferentes sumas de comprobación o copias de datos. La limpieza de datos reduce la probabilidad de que se acumulen errores corregibles únicos, lo que reduce los riesgos de errores incorregibles.

Las características de ZFS incluyen la verificación contra los modos de corrupción de datos, la verificación continua de la integridad y la reparación automática. Sun Microsystems diseñó ZFS desde cero con un enfoque en la integridad de los datos y para proteger los datos en discos contra problemas como los errores de firmware del disco y las escrituras fantasma.

Para poder depurar los datos que tenemos en un pool o un dataset solo debemos de usar la opción **scrub**:
```shell
#### Scrubbing en un pool ####
debian@fs-avanzado:~$ sudo zpool scrub prueba1

#### Vemos el proceso de scrub ####
debian@fs-avanzado:~$ sudo zpool status 
  pool: prueba1
 state: ONLINE
  scan: scrub in progress since Wed Jan 13 09:26:43 2021
	298M scanned out of 300M at 59.7M/s, 0h0m to go
	0B repaired, 99.53% done
config:

	NAME        STATE     READ WRITE CKSUM
	prueba1     ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    vde     ONLINE       0     0     0
	    vdc     ONLINE       0     0     0
	  vdd       ONLINE       0     0     0
```

Esta tecnica tambien la podemos usar con los distintos datasets que tengamos.


# FUENTES

## Btrfs

* Btrfs - https://wiki.gentoo.org/wiki/Btrfs/es#Caracter.C3.ADsticas

* Btrfs - https://es.wikipedia.org/wiki/Btrfs

* Btrfs Wiki - https://btrfs.wiki.kernel.org/index.php/Main_Page

## ZFS

* What is ZFS? Why are People Crazy About it? - https://itsfoss.com/what-is-zfs/

* Install and Setup ZFS on Debian 9 - https://linuxhint.com/install-zfs-debian/

* ZFS - https://wiki.debian.org/ZFS#Installation

* Cómo instalar ZFS en Linux - https://mundo-tips.com/como-instalar-zfs-en-linux/

* ZFS-ArchWiki - https://wiki.archlinux.org/index.php/ZFS#Storage_pools

* Sistema de archivos ZFS de Oracle Solaris - https://docs.oracle.com/cd/E26921_01/html/E25823/zfsover-1.html#scrolltoc

* ZFS Administration - https://pthree.org/2012/04/17/install-zfs-on-debian-gnulinux/
