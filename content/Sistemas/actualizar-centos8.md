---
title: "Actualizar Centos8"
date: 2020-11-19T13:23:35+01:00
draft: true
---

# De centos7 a centos8

En este documento vamos a detallar como vamos a hacer una actualizacion desde centos7 a centos8. Para ello lo recomendable es hacer una copia de nuestra maquina que vamos a actualizar y guardar esa copia para que, en caso de que cometamos algun error y la maquina deje de funcionar, poder restaurarla para volver a intentar. En mi caso, antes de empezar a trabajar con la maquina que queremos tener actualizada, he realizado primero la actualizacion en una maquina de prueba con las mismas caracteristicas y paquetes que la maquina que queremos actualizar. En mi caso la maquina actualizar es **Quijote** que tiene cnetos 7 y la version del kernel **3.10.0-1160.2.2.el7.x86_64**.

## Pasos a seguir

### Preparando el sistema

Lo primero que debemos de hacer en nuestra maquina de prueba es instalar los repositorios de **epel-release** en el caso de que no lo tengamos instalado aun, para ello usamos el siguiente comando:
```shell
[root@quijote ~]# yum install epel-release
```

Ademas de tener instaladas la utilidades basicas del gestor de paquetes **yum**:
```shell
[root@quijote ~]# yum install yum-utils
```

Y ahora con la herramienta **rpmconf**, que se encarga de buscar archivos **.rpmnew**, **.rpmsave** y **.rpmorigfiles** y nos pregunta que debe de hacer con ellos, mantener la version actual, usar la version actual o consultar a **diff**:
```shell
#### Instalamos rmpconf ####
[root@quijote ~]# yum install rpmconf

#### Usamos el comando rpmconf -a para buscar todos los archivos####
[root@quijote ~]# rpmconf -a
```

Y nos deshacemos de los paquetes que no necesitemos con la herramienta llamada package-cleanup:
```shell
[root@quijote ~]# package-cleanup --orphans

[root@quijote ~]# package-cleanup --leaves
```

### Instalando DNF

Llegados a este punto debemos de instalar el gestor de paquetes que centos8 usa por defecto que es el gestor llamado **DNF**, despues de instalar dnf es recomendable desinstalar yum para seguir trabajando con dnf:
```shell
#### Instalamos el gestor de paquetes dnf ####
[root@quijote ~]# yum install dnf

#### Desinstalamos yum ####
[root@quijote ~]# dnf remove yum yum-metadata-parser

#### Borramos la carpeta de yum ####
[root@quijote ~]# rm -Rf /etc/yum

#### Actualizamos con dnf ####
[root@quijote ~]# dnf update
```

### Actualizar a centos8

Una vez en este punto debemos de habilitar los repositorios de centos8, para ello usamos el siguiente comando:
```shell
[root@quijote ~]# dnf install http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-repos-8.2-2.2004.0.1.el8.x86_64.rpm \
http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-release-8.2-2.2004.0.1.el8.x86_64.rpm \
http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-gpg-keys-8.2-2.2004.0.1.el8.noarch.rpm

```

Y hacemos lo mismo con el repositorio de **epel-release**
```shell
[root@quijote ~]# dnf upgrade -y epel-release
```

A continuacion creamos el cache de dnf con la opcion **makecache**:
```shell
[root@quijote ~]# dnf makecache
```

Y borramos todos los archivos temporales:
```shell
[root@quijote ~]# dnf clean all
```

Ahora debemos de eliminar todas las versiones del kernel que tenemos en nuestro equipo, para ello vamos a usar el comando rpm:
```shell
#### Borramos los kernel ####
[root@quijote ~]# rpm -e `rpm -q kernel`
grubby fatal error: unable to find a suitable template
grubby: doing this would leave no kernel entries. Not writing out new config.

Donde vemos que:
    -e:Indicamos que vamos a borrar.
    -q: Hacemos una busqueda de los paquetes "kernel".

#### Borramos los paquetes conflictivos ####
[root@quijote ~]# rpm -e --nodeps sysvinit-tools
```

Ahora procedemos a actualizar a la nueva version de centos8, para ello usamos el siguiente comando:
```shell
[root@quijote ~]# dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync
```

Cuando lo hacemos veremos que nos fallan algunos paquetes que tienen conflicto con otros, para ello vamos a borrar esos paquetes conflictivos que en mi caso son los siguientes:
```shell
[root@quijote ~]# rpm -e --justdb python36-rpmconf-1.0.22-1.el7.noarch rpmconf-1.0.22-1.el7.noarch

[root@quijote ~]# rpm -e --justdb --nodeps python3-setuptools-39.2.0-10.el7.noarch

[root@quijote ~]# rpm -e --justdb --nodeps vim-minimal

#### Y hacemos un upgrade de los paquetes ####
[root@quijote ~]# dnf upgrade --best --allowerasing rpm
```

Despues de todo volvemos a actualizar a la nueva version de centos8. Cuando lo hayamos hecho solo debemos de instalar el nuevo kernel de centos8 con el siguiente comando:
```shell
[root@quijote ~]# dnf install -y kernel-core
```

E instalamos los paquetes minimos del sistema:
```shell
[root@quijote ~]# dnf -y groupupdate "Core" "Minimal Install" --allowerasing --skip-broken
```

Ahora solo nos queda reiniciar la maquina y comprobar que tenemos cento8 en vez de centos7. Para ello debemos de dirigirnos al fichero **/etc/redhat-release** y si visualizamos su contenido debe de salirnos algo comoe esto:
```shell
[root@quijote ~]# cat /etc/redhat-release 
CentOS Linux release 8.2.2004 (Core) 
```

Y ya tendriamos centos8, si queremos podemos ver tambien que repositorios esta usando dnf:
```shell
[root@quijote ~]# dnf repolist
repo id                                               repo name
AppStream                                             CentOS-8 - AppStream
BaseOS                                                CentOS-8 - Base
epel                                                  Extra Packages for Enterprise Linux 8 - x86_64
epel-modular                                          Extra Packages for Enterprise Linux Modular 8 - x86_64
extras                                                CentOS-8 - Extras
```

Y vemos que esta usando los repositorios de centos8.

## Fuentes consultadas para la actualizacion
Cómo subir de versión de Centos 7 a Centos 8: [enlace](https://www.ochobitshacenunbyte.com/2020/04/17/como-subir-de-version-de-centos-7-a-centos-8/)

How to Upgrade CentOS 7 to CentOS 8: [enlace](https://www.tecmint.com/upgrade-centos-7-to-centos-8/)


