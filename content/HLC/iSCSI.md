---
title: "ISCSI"
date: 2021-02-10T19:19:56+01:00
draft: true
---

# iSCI

## ¿Que es iSCSI?

iSCSI (Abreviatura de Internet SCSI) es un estándar que permite el uso del protocolo SCSI sobre redes TCP/IP. iSCSI es un protocolo de la capa de transporte definido en las especificaciones SCSI-3.

El protocolo iSCSI utiliza TCP/IP para sus transferencias de datos. Al contrario que otros protocolos de red diseñados para almacenamiento solamente requiere una simple y sencilla interfaz Ethernet para funcionar. Esto permite una solución de almacenamiento centralizada de bajo coste sin la necesidad de realizar inversiones costosas ni sufrir las habituales incompatibilidades asociadas a las soluciones de canal de fibra para Redes de área de almacenamiento.

# Ejercicios a realizar


Configura un escenario con vagrant o similar que incluya varias máquinas que permita realizar la configuración de un servidor iSCSI y dos clientes (uno linux y otro windows). Explica de forma detallada en la tarea los pasos realizados.

* Crea un target con una LUN y conéctala a un cliente GNU/Linux. Explica cómo escaneas desde el cliente buscando los targets disponibles y utiliza la unidad lógica proporcionada, formateándola si es necesario y montándola.

* Utiliza systemd mount para que el target se monte automáticamente al arrancar el cliente

* Crea un target con 2 LUN y autenticación por CHAP y conéctala a un cliente windows. Explica cómo se escanea la red en windows y cómo se utilizan las unidades nuevas (formateándolas con NTFS)

En mi caso vamos a usar el siguiente escenario de vagrant en el que vamos a configurar **iSCSI**
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|
  disco = '.vagrant/midisco.vdi'
  config.vm.define :maquina1 do |maquina1|
    maquina1.vm.box = "debian/buster64"
    maquina1.vm.hostname = "maquina1"
    maquina1.vm.network :public_network, :bridge=>"wlp2s0"
    maquina1.vm.network :private_network, ip: "192.168.100.100"
    maquina1.vm.provider :virtualbox do |v|
      v.customize ["createhd", "--filename", disco, "--size", 1024]
      v.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 1, "--device", 0, "--type", "hdd", "--medium", disco]
    end
  end
  config.vm.define :maquina2 do |maquina2|
    maquina2.vm.box = "debian/buster64"
    maquina2.vm.hostname = "maquina2"
    maquina2.vm.network :private_network, ip: "192.168.100.101"
  end
end
```

Donde tenemos dos maquinas donde *maquina1* tiene una interfaz en modo puente para el exterior y un disco adicional y las dos maquinas tienen una interfaz de red con una ip privada.

## Tarea 1

Para ello vamos a usar a *maquina1* para empezar la configuración, lo que vamos a hacer es instalar en ella el paquete **lvm2** en nuestra maquina para emepezar a crear el grupo de volumenes y mas tarde crear los volumenes logicos que necesitemos, en este caso haremos solo un volumen logico de 500MB:
```shell
#### Instalamos el paquete de lvm2 ####
vagrant@maquina1:~$ sudo apt install lvm2

#### Creamos el grupo de volumenes con el disco secundario sdb ####
vagrant@maquina1:~$ sudo pvcreate /dev/sdb 
  Physical volume "/dev/sdb" successfully created.

vagrant@maquina1:~$ sudo vgcreate grupo1 /dev/sdb
  Volume group "grupo1" successfully created

#### Creamos el volumen logico ####
vagrant@maquina1:~$ sudo lvcreate -L 500M -n v_grupo1 grupo1
  Logical volume "v_grupo1" created.
```

Ya hecho el volumen logico lo que vamos a hacer ahora es empezar la configuración del target en nuestra *maquina1*, por lo que debemos de instalar en ella un paquete llamado **tgt**:
```shell
#### Instalamos el paquete ####
vagrant@maquina1:~$ sudo apt install tgt
```

Una vez instalado vamos a configurar el fichero llamado **/etc/tgt/targets.conf** donde, como el nombre indica, tenemos que indicar los target que vamos a tener, en este caso solo vamos a tener uno que va a ser el volumen logico llamado *v_grupo1*:
```shell
#### Configuramos el fichero /etc/tgt/targets.conf ####
vagrant@maquina1:~$ sudo nano /etc/tgt/targets.conf

<target iqn.prueba-iSCSI.com:target1>
    backing-store /dev/grupo1/v_grupo1
</target>
```

Una vez hecho esto debemos de reinciar el servicio **tgt**, y una vez hecho esto debe de haberse detectado el disco iSCSI, para comprobarlo podemos usar el siguiente comando:
```shell
#### Vemos si el disco esta en linea ####
vagrant@maquina1:~$ sudo tgtadm --lld iscsi --op show --mode target
Target 1: iqn.prueba-iSCSI.com:target1
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
    LUN information:
        LUN: 0
            Type: controller
            SCSI ID: IET     00010000
            SCSI SN: beaf10
            Size: 0 MB, Block size: 1
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: null
            Backing store path: None
            Backing store flags: 
        LUN: 1
            Type: disk
            SCSI ID: IET     00010001
            SCSI SN: beaf11
            Size: 524 MB, Block size: 512
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: rdwr
            Backing store path: /dev/grupo1/v_grupo1
            Backing store flags: 
    Account information:
    ACL information:
        ALL
```

De esta forma ya tendremos el servidor configurado y listo para que los clientes puedan usar el disco por lo que vamos a pasar a la configuracion del iniciador.

### Iniciador

En la *maquina2* debemos de instalar un paquete llamado **open-iscsi** el cual nos va a permitir usar el disco target que tenemos en la *maquina1*:
```shell
vagrant@maquina2:~$ sudo apt install open-iscsi
```

Una vez instalado el paquete vamos a configurar el fichero llamado **/etc/iscsi/iscsid.conf** en el cual le vamos a indicar que lea los targets de forma automatica, para ello debemos de añadir la linea siguiente:
```shell
vagrant@maquina2:~$ sudo nano /etc/iscsi/iscsid.conf

...
# To request that the iscsi initd scripts startup a session set to "automatic".
node.startup = automatic
#
# To manually startup the session set to "manual". The default is manual.
# node.startup = manual
...
```

Hecho esta modificación debemos de reiniciar el servicio de **open-iscsi** para que tenga efecto y, una vez hecho ese reinicio debemos de comprobar si el target esta hecho correctamente, para ello podemos usar el siguiente comando:
```shell
#### Reiniciamos los servicios ####
vagrant@maquina2:~$ sudo systemctl restart open-iscsi

#### Comprobamos los targets ####
vagrant@maquina2:~$ sudo iscsiadm -m discovery -t st -p 192.168.100.100
192.168.100.100:3260,1 iqn.prueba-iSCSI.com:target1
```

Como vemos ya nos esta inidicando el target que tenemos en la dirección ip que le hemos indicado, que es la ip de *maquina1* y nos indica el disco que tenemos en target. Ahora vamos a proceder a conectarnos a este target, para ello debemos de usar el siguiente comando para poder conectarnos a ese target:
```shell
#### Nos conectamos al target que le indicamos junto a la ip de la maquina ####
vagrant@maquina2:~$ sudo iscsiadm -m node -T iqn.prueba-iSCSI.com:target1 --portal "192.168.100.100" --login
Logging in to [iface: default, target: iqn.prueba-iSCSI.com:target1, portal: 192.168.100.100,3260] (multiple)
Login to [iface: default, target: iqn.prueba-iSCSI.com:target1, portal: 192.168.100.100,3260] successful.

#### Usamos lsblk para ver que tenemos ese target en nuestra maquina ####
vagrant@maquina2:~$ lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0 19.8G  0 disk 
├─sda1   8:1    0 18.8G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0 1021M  0 part [SWAP]
sdb      8:16   0  500M  0 disk 
```

Como vemos ya tenemos el target de 500MB en nuestra maquina, por lo que vamos a proceder a formatearlo con un sistema de fichero **ext4** y lo vamos a montar en nuestra maquina, por lo que vamos a seguir los siguiente pasos:
```shell
#### Usamos gdisk para particionarlo ####
vagrant@maquina2:~$ sudo gdisk /dev/sdb 
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-1023966, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-1023966, default = 1023966) or {+-}size{KMGTP}: 
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.

#### Lo formateamos en ext4 ####
vagrant@maquina2:~$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 510956 1k blocks and 128016 inodes
Filesystem UUID: e296c077-5886-436f-97c2-b27077ac08fb
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729, 204801, 221185, 401409

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

#### Lo montamos en /mnt ####
vagrant@maquina2:~$ sudo mount /dev/sdb1 /mnt/

vagrant@maquina2:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0 19.8G  0 disk 
├─sda1   8:1    0 18.8G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0 1021M  0 part [SWAP]
sdb      8:16   0  500M  0 disk 
└─sdb1   8:17   0  499M  0 part /mnt
```

## Tarea 2

Utiliza systemd mount para que el target se monte automáticamente al arrancar el cliente.

En esta parte de la práctica vamos a ver como es la configuración para que se monte automáticamente el target al arrancar el cliente utilizando systemd mount.

Para ello debemos de irnos a la *maquina1* para configurar el fichero **/etc/tgt/targets.conf** y debemos de añadir la siguiente linea:
```shell
#### Modificamos el fichero /etc/tgt/targets.conf ####
vagrant@maquina1:~$ sudo nano /etc/tgt/targets.conf

#### Reiniciamos el servicio tgt ####
vagrant@maquina1:~$ sudo systemctl restart tgt
```

Una vez tengamos esta configuración debemos de irnos a la *maquina2* y ahi debemos de crear una nueva unidad de systemd en la que le vamos a indicar que se monte el target de forma automatica, para ello debemos de crear la siguiente unidad:
```shell
#### Creamos la unidad de systemd ####
vagrant@maquina2:~$ sudo nano /etc/systemd/system/prueba-sSCSI.mount

[Unit]
Description=Montaje

[Mount]
What=/dev/sdb1
Where=/mnt
Type=ext4
option=_netdev

[Install]
WantedBy=multi-user.target

#### Creamos la unidad de automount de systemd ####
vagrant@maquina2:~$ sudo nano /etc/systemd/system/mnt.automount

[Unit]
Description=Automount

[Automount]
Where=/mnt

[Install]
WantedBy=multi-user.target


#### Reiniciamos los servicios ####
vagrant@maquina2:~$ sudo systemctl daemon-reload
```

### fstab

Tambien podemos añadir la unidad al fichero de **fstab** en el que indicamos que unidades se van a montar en el arranque del sistema indicando el UUID del volumen, el punto de montaje, el tipo de formato y las opciones.

Asi, si queremos hacerlo por esta via solo debemos de añadir la siguiente linea a nuestro fichero **/etc/fstab**:
```shell
#### Buscamos el uuid de nuestro volumen ####
vagrant@maquina2:~$ lsblk -f | grep sdb1
└─sdb1 ext4         e296c077-5886-436f-97c2-b27077ac08fb

#### Añadimos la siguiente linea a nuestro fichero fstab ####
...
UUID=e296c077-5886-436f-97c2-b27077ac08fb       /mnt    ext4    _netdev 0       0
...
```

Ahora que tenemos la linea añadida debemos de comprobar si se monta correctamente, en este caso lo mejor no es reiniciar y ver si se monta ya que eso podria dar un problema y hacer que la maquina no inicie correctamente por lo que debemos de hacer usar el comando **mount** de la siguiente manera:
```shell
#### Comprobamos que no esta montado ####
vagrant@maquina2:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0 19.8G  0 disk 
...
sdb      8:16   0  500M  0 disk 
└─sdb1   8:17   0  499M  0 part 

#### Usamos mount para montar todos los volumenes del fichero fstab ####
vagrant@maquina2:~$ sudo mount -a

#### Comprobamos que se haya montado correctamente ####
vagrant@maquina2:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0 19.8G  0 disk 
...
sdb      8:16   0  500M  0 disk 
└─sdb1   8:17   0  499M  0 part /mnt
```

## Tarea 3





