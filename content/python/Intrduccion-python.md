---
title: "Introducción a Python"
date: 2022-06-14T21:00:01+01:00
draft: true
---

# Introducción

En esta entrada vamos a repasar una pequeña introducción a la programación con **python**, para ello vamos a seguir una serie de pautas como son el entorno donde vamos a hacer todas las pruebas y ejercicios, editores de codigo que vamos a usar y el pequeño "temario" que vamos a seguir para avanzar de la forma mas ordenada posible.

	1. Introducción a Python3.
	2. Instalación de python
	3. Tipos de datos.
	4. Manejando variables.
	5. Estructuras de control.
	6. Introducción a las listas.
	7. Introducción a los diccionarios.
	8. Trabajando con librerias.
	
## 1. Introducción a python

Python es un lenguaje de programación interpretado cuya filosofía hace hincapié en una sintaxis que favorezca un código legible.

Se trata de un lenguaje de programación multiparadigma, ya que soporta orientación a objetos, programación imperativa y, en menor medida, programación funcional. Es un lenguaje interpretado, usa tipado dinámico y es multiplataforma.

pero, ¿por qué elegir pyhton como lenguaje de programación? Para empezar, si nunca se ha programado y, como en mi caso, este es tu primer legunaje de programación y lo vas a usar para poder entrar en el mundo de la programación es de los mejores ya que es un **lenguaje facil de aprender** y tiene **una sintaxis muy limpia y legible**, lo que nos ayudara a comprender mejor como se escribe un código.

Como hemos dicho al principio, es un **lenguaje interpretado por lo que no es necesario que usemos un compilador para poder ejecutar el código**. Tambien podemos usarlo para distintas aplicaciones, ya sea para programas de escritorio, aplicaciones web o para scripts.

Tambien podemos tener en cuenta que es un leguaje muy demandado ya que se usa en muchas empresas de gran calibre. Tenemos gran cantidad de modulos a nuestra disposición lo que nos dara mayor funcionalidad.

Y una de las cosas mas interesantes es que, en la gran mayoria de sistemas operativos nuevos ya viene preinstalado en el, por lo que no debemos de preocuparnos por instalarlo.


## 2. Entorno de pruebas e instalación

Para esta entrada vamos a usar como maquina de pruebas una maquina virtual creada en un cluster de VMware, se trata de una maquina con CentOS 7 que esta actualizada a la ultima versión y que tiene instalada los paquetes necesarios y el paquete que mas nos importa en esta entrada, el paquete de python3.

A continuación vamos a ver como instalar python en las distintas distribuciones de Linux y en Windows:

### Instalación en Ubuntu o Debian
```bash
## Actualizamos la lista de paquetes
root@pruebas:~# apt update

## Instalamos el paquete de Python3
root@pruebas:~# apt install python3

## Comprobamos la version que hemos instalado
root@pruebas:~# python3 --version
Python 3.6.9
```

### Instalación en CentOS
```bash
## Actualizamos la lista de paquetes
[root@sb-cert01 ~]# yum update

## Instalamos el paquete de python3
[root@sb-cert01 ~]# yum install python3

## Comprobamos la version de python
[root@sb-cert01 ~]# python3 --version
Python 3.6.8
```

### Instalación en Windows 10

Para ello lo que debemos de hacer es dirigirnos a este [enlace a la pagina de python](https://www.python.org/downloads/) y ahi, podemos descargar la versión que necesitemos, en nuestro caso lo mas recomendable es instalar la ultima versión en nuestro sistema, para ello solo debemos de descargar el paquete y ejecutar el fichero **.exe** y seguir los pasos del instalador.
