---
title: "Integracion Continua"
date: 2021-01-21T17:34:08+01:00
draft: true
---

# Práctica: Introducción a la integración continua

Debes realizar una de las siguientes tareas, utilizando una herramienta de CI/CD: GitHub Actions, GiLab CI/CD, CircleCI, Jenkins, … (No uses Travis CI).

En mi caso vamos a usar **CodeShip**, codeship es una plataforma alojada de integración y entrega continua. Se encuentra entre tu repositorio de código fuente (por ejemplo, GitHub, GitLab o Bitbucket) y el entorno de alojamiento (por ejemplo, Amazon Web Services) y prueba e implementa automáticamente cada cambio en tu plataforma.

# Primeros pasos

Lo primero que debemos de hacer es generar con nuestro generador de paginas estaticas, un nuevo proyecto en el cual solo vamos a añadir unos cuantos ficheros markdown. Este proyecto debemos de subirlo a nuestro github, en mi caso el nombre del repositorio es **https://github.com/FranJaviMN/pagina_hugo_2**.

Una vez ya lo tengamos aqui debemos de irnos a nuestra pagina de surge y crearnos una cuenta.

En mi caso el generador que voy a usar es **hugo** del cual tengo un post en este blog donde hablo un poco sobre este generador y la forma de instalarlo en nuestra maquina y empezar a usar nuestra pagina estática. [Enlace hacia post](https://franjavimn.onrender.com/implantacion/guia-hugo/)

## Instalamos surge

En nuestra maquina debemos de tener instalado **surge** para poder seguir con el ejercicio, para ello debemos de tener instalado el gestor de paquetes llamado **npm** y una vez tengamos ya instalado el gestor de paquete **npm** instalamos surge que va a ser donde vamos a tener nuestra pagina estatica que vamos a crear con integracion continua:
```shell
#### Instalamos surge ####
francisco@debian10:~$ sudo npm install --global surge

#### Iniciamos surge y nos registramos ####
francisco@debian10:~/.ssh$ surge 

   Welcome to surge! (surge.sh)
   Login (or create surge account) by entering email & password.

          email: franjaviermn17100@gmail.com
       password: 

   Running as franjaviermn17100@gmail.com (Student)
```

Una vez lo tengamos podemos desplegar el proyecto para que asi nos de una url o le podemos poner la que queramos. Cuando ya tengamos esto debemos de irnos a la pagina de [CodeShip](https://www.cloudbees.com/). Cuando lo tengamos solo debemos de seleccionar nuestro repositorio en que queremos realizar la integracion. y ya podemos empezar con los pasos que debemos de seguir en la integración

## Pasos en CodeShip

### Test de comprobación

Lo primero que debemos de hacer es irnos a la configuración de nuestro proyecto de CodeShip, cuando estemos debemos de añadir un nuevo test a este el cual verifica si en la maquina en la que se va a realizar nuestra integracion cuenta con el software apropiado y con una apropiada version para que asi pueda funcionar sin problemas. Como nuestro generador de paginas estatica esta escrito en el lenguaje **Go**, necesitamos una versión en la que sepamos que el generador de paginas estaticas funcione por lo que usamos este test:
```shell
# Install a custom Go version, https://golang.org/
#
# Add at least the following environment variables to your project configuration
# (otherwise the defaults below will be used).
# * GO_VERSION
#
# Include in your builds via
# source /dev/stdin <<< "$(curl -sSL https://raw.githubusercontent.com/codeship/scripts/master/languages/go.sh)"
GO_VERSION=${GO_VERSION:="1.6.2"}
# strip all components from PATH which point toa GO installation and configure the
# download location
CLEANED_PATH=$(echo $PATH | sed -r 's|/(usr/local\|tmp)/go(/([0-9]\.)+[0-9])?/bin:||g')
CACHED_DOWNLOAD="${HOME}/cache/go${GO_VERSION}.linux-amd64.tar.gz"
# configure the new GOROOT and PATH
export GOROOT="/tmp/go/${GO_VERSION}"
export PATH="${GOROOT}/bin:${CLEANED_PATH}"
# no set -e because this file is sourced and with the option set a failing command
# would cause an infrastructure error message on Codeship.
mkdir -p "${GOROOT}"
wget --continue --output-document "${CACHED_DOWNLOAD}" "https://storage.googleapis.com/golang/go${GO_VERSION}.linux-amd64.tar.gz"
tar -xaf "${CACHED_DOWNLOAD}" --strip-components=1 --directory "${GOROOT}"
# check the correct version is used
go version | grep ${GO_VERSION}
```
### Añadir las variables de entorno necesarias

Despues de haber agregado el test que comprueba la version de **go** que tiene es la adecuada, debemos de irnos a nuestra maquina y generar un token de surge para asi poder crear nuestra pagina estatica, para ello solo debemos de poner **surge token** y de esta forma nos genera un token que debemos de meter en las variables de entorno de CodeShip.

Para poder meter variables de entorno en CodeShip debemos de irnos a la pestaña de **Environment** y ahi debemos de añadir dos nuevas variables, una para nuestro correo electronico con el que hemos ingresado en surge.sh y otra con el token que nos a generado surge.

Debemos de tener en cuenta que las variables deben de tener un nombre en especifico ya que, como podemos ver mas adelante en el fichero de deploy, en ningun momento indicamos las variables de entorno a la hora de desplegar nuestra pagiana, asi pues la variable de entorno del email debe llamarse **SURGE_LOGIN** y la variable de entorno del token debe de llamarse **SURGE_TOKEN**.

### Fichero de Deploy

Cuando tengamos esto debemos de añadir un nuevo deploy en la pestaña con el mismo nombre, en este deploy vamos a instalar los paquetes necesarios para poder hacer el despliegue de forma automatica, para ello debemos de irnos a la pestaña **deploy** y añadir el siguiente script con el cual vamos a aprovisionar de forma adecuada la maquina que se va a encargar de desplegar la pagina estatica por lo que tenemos que instalarle ciertos paquetes que no tiene la maquina por defecto como son surge o hugo, que lo instalamos mediante un paquete **.deb**:

```shell
# Install pygments for syntax highlighting
pip install Pygments

# Instalamos surge
npm install -g surge

# Instalamos el paquete de hugo desde el fichero .deb descargado
wget https://github.com/gohugoio/hugo/releases/download/v0.75.1/hugo_0.75.1_Linux-64bit.deb
sudo dpkg -i hugo_0.75.1_Linux-64bit.deb

# Usamos el repositorio que hemos clonado y, si tuviera el directorio ./public lo borra
cd ~/clone/
rm -rf ./public

# Construimos el sitio con el comando hugo
hugo -D

# Desplegamos en surge
surge --project ./public/ --domain {nuestro dominio}

Donde mi dominio es: franjmn.surge.sh
```

Donde tenemos el apartado del despliegue, la opcion **--domain** es donde indicamos el dominio que hemos elegido o nos ha dado surge.

Hecho esto nos vamos a nuestro proyecto de CodeShip y ahi realizamos el *build* de la integración y esta empezará con el contenido de nuestro repositorio y, si todo ha ido bien, veremos que en nuestro repositorio tendremos un *tic* verde y el *build* de la integracion a terminado correctamente ya tendremos todo desplegado y en funcionamiento.

Aqui vemos como seria la integración cuando ha funcionado:

![integracion bien](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/pagina-estatica/deploy-bien.png)

Y aqui vemos que en nuestro repositorio aparece el tic verde:

![tic verde](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/pagina-estatica/tic-verde.png)

Asi quedaria el despliegue mediante surge y usando integración continua con CodeShip.

![pagina surge](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/pagina-estatica/surge-pagina.png)
