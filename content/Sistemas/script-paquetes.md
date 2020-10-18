---
title: "Script Paquetes"
date: 2020-10-18T14:44:56+02:00
draft: true
---

# Script para seleccionar paquetes por repositorio

Tenemos que realizar un script que introduciéndolo como parámetro el nombre de un repositorio, muestre como salida los paquetes de ese repositorio que están instalados en la máquina. Los repositorios pueden ser introducidos de dos formas distintas:

* ./script.sh security.debian.org

o bien

* ./script.sh http://security.debian.org

## Script para la selección de paquetes

He realizado el siguiente script el cual tambien podemos encontrar em el siguiente [enlace](https://github.com/FranJaviMN/elementos-grado/blob/main/sistemas/script/listar-paquetes.sh):

```shell

#!/bin/bash

#En el siguiente script vamos a conseguir la lista de paquetes instalados introduciendo como parametro el repositorio que queremos
#consultar.

#Definimos la variable repo que es donde vamos a guardar el parametro de entrada de nuestro script, en este caso sera el repositorio
#que vamos a consultar.

repo=$1

#En esta parte es donde vamos a difenciar las dos formas de consultar los paquetes que tenemos ya que podemos introdocir el repositorio
#de dos formas distintas.

#Si introducimos el repositorio con http://(nombre-repositorio) comprobara el parametro que hemos introducido Ej: http://deb.debian.org

if [[ $repo == *"http"* ]]; then #comprobamos si empieza por la cadena http://
repo="${repo:7}" #guardamos en la variable repo el parametro sin la cadena inicial http://

for p in $(dpkg -l | grep '^ii' | cut -d ' ' -f 3);
do 
apt-cache showpkg $p | head -3 | grep -v '^Versions' | paste - - ; done | grep $repo | awk '{print $2}' 

#Si la cadena no empieza por http:// no hacemos la comprobacion y entramos en el codigo para la consulta de los paquetes Ej: security.debian.org.

else
for p in $(dpkg -l | grep '^ii' | cut -d ' ' -f 3);
do 
apt-cache showpkg $p | head -3 | grep -v '^Versions' | paste - - ; done | grep $repo | awk '{print $2}'
fi
```

A continuación voy a proceder a explicar el script que he usado:

1. Primero debemos de sacar todos los paquetes instalados en nuestro sistema, para ello he usado ***dpkg -l*** y luego mediando el filtrado con ***grep*** sacar solo los paquetes instalados y quedarme solo con el nombre del paquete con ***cut -d ' ' -f 3***. Quedaria de la siguiente forma:

```shell
dpkg -l | grep '^ii' | cut -d ' ' -f 3
```

2. El anterior comando lo introducimos en un bucle para que asi podamos obtener la informacion detallada con el comando ***apt-cache showpkg*** en el cual podemos ver del repositorio desde donde se ha instalado.

```shell
for p in $(dpkg -l | grep '^ii' | cut -d ' ' -f 3)
```

3. Ahora necesitamos obtener el repositorio desde donde se ha instalado el paquete el cual esta en la salida del comando anterior por lo que esta salida debemos filtrarla, para ello vamos a usar el comando ***head*** con el cual vamos a obtener solo las tres primeras lineas ya que en esas 3 lineas podemos ver el nombre del paquete y podemos ver el repositorio desde donde lo instalamos. Con el comando ***grep -v  '^Versions'*** eliminamos la linea intermedia que comienza con la cadena *Version* y unimos las dos lineas en una sola con ***paste - -***.

4. Ahora que ya tenemos todos los elementos en una sola linea lo que debemos de hacer es filtrar por el repositorio que hemos introducido como parametro en el script mediante ***grep $repo*** y mediante el comando ***awk** vamos a sacar por linea de comandos el nombre de los paquetes que estan instalados en nuestro sistema filtrando por repositorios.