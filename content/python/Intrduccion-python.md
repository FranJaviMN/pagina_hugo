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

## 3. Tipos de datos

En python tenemos distintos tipos de datos que podemos usar como son los siguientes:

* **Numeros enteros**: Podemos usar numeros enteros como pueden ser 8, 10, 200...
* **Numeros reales**: Tambien podemos usar numeros reales como 10.7, 137.78...
* **Cadenas de texto**: Podemos usar cadenas de texto que podemos mostrar por pantalla, usar variables...

### Variables

Una de las partes mas importantes es el uso de **variables** en las cuales podemos almacenar distintos tipos de datos para poder usarlos mas adelante o, tambien podemos guardar datos los cuales se iran modificando a lo largo del programa
```python
## Ejemplos de variables

>>> cadena_texto = "hola mundo"
>>> print(cadena_texto)
hola mundo

>>> cadena_numeros = 88
>>> print(cadena_numeros)
88
```

### Operadores

Tambien podemos hacer uso de distintos operadores para realizar operaciones como pueden los siguientes operadores:

#### **Operadores aritmeticos**: Podemos usar tanto sumas(+) como restas(-) para realizar operaciones aritmeticas, tambien podemos multiplicar(\*), podemos dividr(/), podemos hacer divisiones enteras(//), podemos hacer el modulo de un numero(%) o podemos hacer pontencias de un numero(\*\*).

* Sumas y restas
```python
### Ejemplos de sumas

>>> print(3+9)
12

>>> print(3+34553.23)
34556.23

### Ejemplos de restas
>>> print(2321414-34553.23)
2286860.77

>>> print(2321414-334344314553.23)
-334341993139.23

## Tambien podemos unir cadenas de texto

>>> print("hola que tal" + " estas amigo?")
hola que tal estas amigo?
```

* Multiplicaciones y divisiones
```python
### Ejemplo de multiplicaciones

>>> print(3434*2)
6868

>>> print(3*2.99)
8.97

### Ejemplos de divisiones

>>> print(3/2.99)
1.0033444816053512

>>> print(3/2344)
0.001279863481228669
```

* Divisiones enteras, modulos y potencias
```python
### Ejemplo divisiones enteras

>>> print(80//2)
40
>>> print(80//2.24)
35.0

### Ejemplos de modulos

>>> print(80%2.24)
1.5999999999999925

print(80%2443)
80

### Ejemplos de potencias

>>> print(80**2)
6400

>>> print(2**10)
1024
```

#### **Operadores booleanos**: Una variable booleana es una variable que sólo puede tomar dos posibles valores: **True (verdadero) o False (falso)**.

En Python **cualquier variable** (en general, cualquier objeto) puede considerarse como una variable booleana. En general los elementos nulos o vacíos se consideran False y el resto se consideran True.

Pero, ¿Que valores se consideran falsos?:
* False
* Cualquier numero que sea 0 o 0.0...
* Cualquier secuencia vacia, ya sean tuplas o diccionarios.

De este modo podemos tener los siguientes ejemplos:

* Ejemplos de valores booleanos:
```python
### Ejemplos de booleanos.

>>> bool(0)
False
>>> bool(1)
True
>>> bool("")
False
>>> bool(None)
False
>>> bool(())
False
>>> bool([])
False
>>> bool({})
False
>>> bool(34)
True
>>> bool(3222)
True
>>> bool(322.3)
True
```

#### **Operadores logicos**: Los operadores lógicos son unas operaciones que trabajan con valores booleanos. Existen distintos tipos de booleanos:

* and: Es el ***y^*** lógico. Este operador da como resultado **True** si y sólo si sus dos operandos son **True**:
```python
>>> True and True
True
>>> True and False
False
>>> False and True
False
>>> False and False
False
```

* or: Es el ***o*** lógico. Este operador da como resultado True si algún operando es True:
```python
>>> True or True
True
>>> True or False
True
>>> False or True
True
>>> False or False
False
```

**Nota: En el lenguaje cotidiano, el "o" se utiliza a menudo en situaciones en las que sólo puede darse una de las dos alternativas. Por ejemplo, en un menú de restaurante se puede elegir "postre o café", pero no las dos cosas (salvo que se pague aparte, claro). En lógica, ese tipo de "o" se denomina "o exclusivo" (xor).**

* not: Es la negación lógica. Este operador da como resultado True si y sólo si su argumento es False:
```python
>>> not True
False
>>> not False
True
```
