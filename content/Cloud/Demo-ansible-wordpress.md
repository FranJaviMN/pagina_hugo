---
title: "Demo Ansible Wordpress"
date: 2021-01-20T20:18:39+01:00
draft: true
---

# Ansible

**Ansible** es un software que automatiza el aprovisionamiento de software, la gestión de configuraciones y el despliegue de aplicaciones. Está categorizado como una herramienta de **orquestación**, muy útil para los administradores de sistema y *DevOps*.

En otras palabras, Ansible permite a los *DevOps* gestionar sus servidores, configuraciones y aplicaciones de forma sencilla, robusta y paralela.

**Ansible** gestiona sus diferentes nodos a través de SSH y únicamente requiere Python en el servidor remoto en el que se vaya a ejecutar para poder utilizarlo. Usa YAML para describir acciones a realizar y las configuraciones que se deben propagar a los diferentes nodos.

## Demo practica sobre despliegue con Ansible

Para ello vamos a usar el escenario que nos propone nuestro profesor en el [repositorio siguiente](https://github.com/josedom24/automatizacion_iaw).

Le hemos realizado un fork a nuestros repositorios y lo hemos clonado en nuestra maquina con el fin de realizar este escenario ya que se necesita de unas modificaciones para su funcionamiento.

El escenario consiste en el despliegue de una maquina en nuestro entorno de openstack la cual tenga un servidor web corriendo wordpress, esto implica la instalacion de un servidor web como es apache2, una base de datos que sera una mariadb y la instalacion de wordpress.

### Instalamos ansible

Para ello y para que no interfiera con la paqueteria de nuestra maquina, vamos a crear en un entorno virtual de python ya que ansible esta escrito en ese lenguaje, para ello vamos a necesitar el paquete llamado **python3-venv**. He de recalcar que lo estamos haciendo en una maquina con un sistema Debian 10 buster.
```shell
#### Instalamos el paquete de python3-venv ####
francisco@debian10:~$ sudo apt install python3-venv

#### Creamos el entorno virtual que lo llamaremos ansible ####
francisco@debian10:~/Documentos/Cloud-computing/Ansible$ python3 -m venv ansible

#### Ejecutamos el entorno virtual e instalamos ansible ####
(ansible) francisco@debian10:~/Documentos/Cloud-computing/Ansible$ pip install ansible
```

De esta forma ya tendremos instalado lo que necesitamos, ahora vamos a necesitar nuestro nuevo repositorio clonado en nuestra maquina.

### Modificacion del escenario

Lo primero que debemos de modificar de nuestro escenario es al host al que este va dirigido, esto lo encontraremos en el fichero llamado **host** que, como vemos en el fichero **ansible.cfg**, es un inventario donde vamos a tener las maquinas que vamos a usar, en nuestro caso solo vamos a usar una sola maquina por lo que debemos de ponerle la ip de la maquina a la que vamos a aprovisionar, en mi caso la ip es **172.22.200.145**
```
[servidores_web]
nodo1 ansible_ssh_host=172.22.200.145 ansible_python_interpreter=/usr/bin/python3
```

Una vez hecho este cambio debemos de tener en cuenta que debemos de tener conectividad ssh a la maquina en cuestion y sobretodo que sea mediante claves ssh, por lo que debemos de asegurarnos la conexion a la maquina antes de empezar:
```shell
#### Probamos el ssh ####
francisco@debian10:~/.ssh$ ssh debian@172.22.200.145
...
debian@prueba-ansible:~$
```

### Empezamos el escenario

Lo primero que vamos a hacer es, mediante las herramientas de ansible, es instalar en la maquina destino un paquete llamado **python3-apt** ya que es una dependencia importante que debemos de tener si queremos usar este escenario de ansible:
```shell
#### Desde el entorno virtual, instalamos el paquete ####
(ansible) francisco@debian10:~/Documentos/Cloud-computing/Ansible/automatizacion_iaw$ ansible all -m apt -a "pkg=python3-apt" -b

#### Iniciamos el playbook ####
(ansible) francisco@debian10:~/Documentos/Cloud-computing/Ansible/automatizacion_iaw$ ansible-playbook site.yaml
```

Este proceso puede tardar un tiempo pero deberia de ir como la seda y no dar ningun problema. Cuando acabe, solo debemos de irnos al navegador y poner la ip del servidor al cual hemos aprovisionado y veremos un mensaje como el siguiente:

![mensaje hola ansible](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/ansible/prueba-ansible-hola.png)

Ahora vamos a añadirle a este inicio un enlace a la zona donde instalaremos wordpress, para ello solo nos debemos de ir al fichero que se encuentra en **roles/apache2/templates** y añadirle el enlace hacia **/wordpress** y volvemos a ejecutar el playbook:
```shell
#### Ejecutamos de nuevo el playbook ####
(ansible) francisco@debian10:~/Documentos/Cloud-computing/Ansible/automatizacion_iaw$ ansible-playbook site.yaml
```

Si nos fijamos es mucho mas rapido que la primera vez y esto se debe a algo evidente, ansible, al intentar realizar las acciones que le decimos en los roles que, cuando instala algo ya ve que esta instalado por lo que directamente pasa a la siguiente accion, por eso solo vemos que ha cambiado una cosa al ejecutar el playbook, el fichero .html:
```shell
PLAY RECAP ******************************************************************************************************************************************
nodo1                      : ok=16   changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Por eso ahora al entrar en la ip veremos que tenemos el enlace hacia la instalacion de wordpress:

![ansible cambiado wordpress](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/ansible/wordpress-ansible.png)