---
title: "Guia Hugo"
date: 2020-10-14T17:15:19+02:00
draft: true
---


# GoHugo

## ¿Que es GoHugo?

![Logo de hugo](https://docs.altinn.studio/technology/hugo/hugo-logo.png?width=300 "Logo de Hugo")

[Hugo](https://gohugo.io/getting-started/quick-start/) es un generador de sitio estático escrito en Go. Hugo ha visto un gran aumento en las características y el rendimiento gracias al desarrollador principal actual Bjørn Erik Pedersen y otros colaboradores. Hugo es de codigo abierto bajo la licencia de [Apache License 2.0](https://en.wikipedia.org/wiki/Apache_License).

Al ser capaz de generar la mayoría de los sitios web en segundos (a <1 ms por página), Hugo es reconocido como "el framework más rápido del mundo para la creación de sitios web" gracias no solo a que se ha creado con **Go**, sino también a los esfuerzos concienzudos de sus desarrolladores para comparar y aumentar actuación. Su velocidad y su conjunto de características en evolución llevaron a un gran aumento en su popularidad.

Hugo toma archivos de datos, paquetes i18n , configuración, plantillas para diseños, archivos estáticos y contenido escrito en Markdown o Org-mode y renderiza un sitio web estático. Algunas características notables son soporte multilingüe, procesamiento de imágenes, formatos de salida personalizados y códigos cortos.

## Primeros pasos con Hugo

Lo primero que debemos de hacer es instalar el paquete de *hugo* en nuestro sistema debian, para ello podemos, o bien descargarlo desde repositorios, aunque esta opcion no es muy recomendable ya que la version de repositorios es bastante antigua o bien podemos descargar el paquete desde su repositorio oficial de github en este [enlace](https://github.com/gohugoio/hugo/releases). En mi caso he descargado e instalado la version **0.75.1**.

Una vez hayamos instalado el paquete de hugo podemos ver su version con el siguiente comando:

```shell
hugo version
```

Asi podemos verificar la version que hemos instalado, una vez instalado ya podemos iniciar nuestro primer sitio estatico con hugo, para ello usamos el siguiente comando:

```shell
hugo new site "nombre-sitio"
```

Con el anterior comando creamos un directorio llamado "nombre-sitio" que contiene todos los elementos necesarios para empezar a crear nuestro sitio estatico.

## Añadir tema

Lo primero que haremos es añadir un tema a nuestra sitio estatico, para ello nos vamos al siguiente [enlace](https://themes.gohugo.io/) donde podremos encontrar una gran cantidad de temas para nuestro sitio estatico, en mi caso he elegido el tema llamado [cactus](https://themes.gohugo.io/hugo-theme-cactus/). 

Una vez tengamos el tema ya elegido y descargado en nuestro equipo, movemos el tema a la carpeta de nuestro sitio web que se llama ***themes*** y ahi ponemos nuestro tema elegido.

Como veremos existe un fichero llamado ***content.toml*** en el cual vamos a definir los parametros de nuestra pagina como pueden ser el tema que vamos a usar, el nombre de nuestro sitio estatico, la url que va a tener, lenguaje de nuestro sitio, etc. En este caso solo vamos a definir de momento el tema de la siguiente forma:

```toml
theme = "cactus"
```

Tenemos que tener en cuenta que debemos de leer la documentacion del tema que nosotros vayamos a elegir ya que en dicha documentacion nos dan algunos parametros para el fichero *config.toml* que, si es verdad que no son necesarias, dan algunas funciones a nuestro sitio que quedan muy curiosas y muy chulas.

## Añadir contenido a nuestro sitio

Ahora que tenemos todo preparado es hora de añadir el contenido a nuestro sitio estatico, para ello podemos hacer de dos maneras distintas:

* De forma manual: Podemos crear los fichero manualmente, para ello debemos de guardar el fichero con la siguiente estrutura de directorios:
    ```shell
    content/<categoria>/<fichero>.md
    ```
* Mediante el comando *new*: Podemos usar un comando del paquete hugo que nos permite, desde el directorio de nuestro sitio, crear el contenido directamente, para ello usamos el siguiente comando:
    ```shell
    hugo new <categoria>/<fichero>.md

    #### El fichero sera creado con las siguientes lineas ####
    ---
    title: "nombre del post"
    date: fecha de creacion del post
    draft: true
    ---
    ```

De esta forma ya podremos empezar a rellenar con contenido nuestro sitio estatico.

## Iniciar el servidor hugo

Ahora vamos a comprobar nuestro sitio estatico de forma local para poder comprobar que todo lo que hemos hecho esta correcto, para ello nos vamos a la terminal y en el directorio de nuestra pagina estatica ejecutamos el siguiente comando:

```shell
hugo server -D
```

De esta forma nos indicara que en ***http://localhost:1313/*** tendremos nuestra pagina para verla en forma local, si queremos parar nuestra pagina solo debemos presionar **ctrl+C**.

## Generar nuestra pagina estatica

Ahora vamos a generar nuestra pagina estatica para asi poder llevarla hasta un servicio de despliegue donde poder alogarla, para ello vamos a usar el siguiente comando:

```shell
hugo -D
```

De esta forma nos generara un directorio llamado **public** que es la carpeta que va a servir nuestro servicio de despliegue.

## Desplegar en Render

En mi caso he usado el servicio de despliegue llamado [Render](https://render.com/) el cual nos permite desplegar de forma muy sencilla y rapida nuestra pagina estatica generada con hugo, debemos de tener en cuenta que nuestra pagina generada debe de estar guardada en un repositorio ya sea github o gitlab. 

Nos registramos y una vez hecho esto nos dirigimos a un nuevo despliegue donde vamos a elegir como despliegue una pagina estatica y seleccionamos github o gitlab y seleccionamos el repositorio donde tengamos nuestra pagina. Una vez hecho esto le ponemos el nombre que nosotros deseemos, le decimos que sirva la carpeta **public** y que use el comando de construccion de la pagina estatica siguiente:

```shell
hugo --gc --minify

Donde:
    --minify: Soporte para cualquier formato
    --gc: habilitar para ejecutar algunas tareas de limpieza
```

## Script

He realizado el siguiente [script](https://github.com/FranJaviMN/elementos-grado/blob/main/Implantacion/pagina-estatica/script-commit.sh) en el cual subimos los cambios que realicemos en nuestra pagina estatica:

```shell
#!/bin/bash

# Nos vamos al directorio donde tenemos la pagina estatica, en este caso es el directorio /home/francisco/Documentos/Implantacion/pagina_hugo
cd /home/francisco/pagina_hugo

# Generamos los elementos necesarios de nuestra pagina estatica.
hugo -D

# Añadimos todos los ficheros nuvos que hemos creado, en caso de que no haya ningun fichero que añadir no añadira nada.
git add *

# Nos pide que introduzcamos la descripcion del commit que vamos subir a nuestro repositorio en github
echo "Dime el mensaje del commit:"
read commit
# Crea el commit con el mensaje que hemos metido
git commit -m "$commit"

# Subimos los cambios a nuestro repositorio.
git push
```





