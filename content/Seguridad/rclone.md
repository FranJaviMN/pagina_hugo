---
title: "Rclone"
date: 2020-10-05T19:55:23+02:00
draft: true
---

# rclone en linux

## ¿Qué es rclone?

se trata de una herramienta en línea de comando para sincronizar archivos y directorios desde la computadora con los proveedores más importantes de alojamiento de contenidos en la nube. También permite efectuar copias dentro de nuestro propio sistema de archivos.

## Practica con rclone

A continuación vamos a realizar distintas practicas con la herramienta de rclone en nuestro ordenador, para ello vamos a tener en total 4 tareas distintas.

* Tarea 1: Instala rclone en tu equipo.

* Tarea 2: Configura dos proveedores cloud en rclone (dropbox, google drive, mega, …)

* Tarea 3: Muestra distintos comandos de rclone para gestionar los ficheros de los proveedores cloud: lista los ficheros, copia un fichero local a la nube, sincroniza un directorio local con un directorio en la nube, copia ficheros entre los dos proveedores cloud, muestra alguna funcionalidad más,…

* Tarea 4: Monta en un directorio local de tu ordenador, los ficheros de un proveedor cloud. Comprueba que copiando o borrando ficheros en este directorio se crean o eliminan en el proveedor.

### Tarea 1

Para la instalación de rclone tenemos 2 opciones de instalación para esta herramienta:

* Opción 1: En la opcion uno la instalación de la herramienta de rclone la vamos a realizar mediante los repositorios oficiales de Debian, para ello nos vamos a la linea de comandos y escribimos lo siguiente:

```shell
    francisco@debian10:~$ sudo aptitude install rclon
```

* Opción 2: En la siguiente opción, en vez de usar los repositorios oficiales de Debian, vamos a instalar la versión que esta en la pagina oficial de rclone, para ello primero descargamos el fichero .deb:

```shell
    francisco@debian10:~/Descargas$ wget https://downloads.rclone.org/v1.53.1/rclone-v1.53.1-linux-amd64.d

    y a continuación instalamos el paquete descargado:

    francisco@debian10:~/Descargas$ sudo dpkg -i rclone-v1.53.1-linux-amd64.d
```

### Tarea 2

En esta tarea lo que debemos de hacer es configurar dos proveedores cloud con nuestro rclone, en mi caso voy a usar Google Drive y DropBox.

Una vez tengamos instalado rclone, nos dirigimos a la linea de comandos y en ella escribimos lo siguiente:
El siguiente comando nos permite la configuración de nuestro rclone:

```shell
    francisco@debian10:~$ rclone config
```

Una vez hayamos iniciado el comando nos apareceran las distintas opciones que nos da rclone, nosostros necesitamos crear un nuevo “remote” para ello pulsamos la letra que nos indica el menu que nos aparece y le decimos el nombre que va a tener, en mi caso “Drive”.

```shell
    No remotes found - make a new one 
    n) New remote 
    s) Set configuration password 
    q) Quit config 
    n/s/q> n 
    name> Drive
```

Una vez lo hayamos hecho debemos de elegir el proveedor cloud que nosotros vayamos a usar, en mi caso usare drive, que es el numero 12, y tambien usare dropbox, que es el numero 8, pero cada uno de los proveedores se debe configurar a parte, asi que elegimos solo uno, que sera Drive.

```shell
    Type of storage to configure. 
    Enter a string value. Press Enter for the default (""). 
    Choose a number from below, or type in your own value 
    1 / A stackable unification remote, which can appear to merge the contents of several remotes 
    \ "union" 
    2 / Alias for a existing remote 
      \ "alias" 
    3 / Amazon Drive 
      \ "amazon cloud drive" 
    4 / Amazon S3 Compliant Storage Providers (AWS, Ceph, Dreamhost, IBM COS, Minio) 
      \ "s3"
    . . .
    Storage> 12
```

Una vez elegido el proovedor nos pediran el nombre del cliente y su contraseña aunque estos campos los podemos dejar en blanco si lo deseamos.

```shell
    Enter a string value. Press Enter for the default (""). 
    client_id> 
    Google Application Client Secret 
    Leave blank normally. 
    Enter a string value. Press Enter for the default (""). 
    client_secret> 
    Scope that rclone should use when requesting access from drive. 
    Enter a string value. Press Enter for the default ("").
```

Una vez realizado el los campos anteriores nos pedirá que tipo de acceso que vamos a tener a nuestro proveedor cloud, en mi caso quiero que el acceso a este proveedor sea total, para ello le indicamos el numero que que esta en el menú que nos aparece.

```shell
    Choose a number from below, or type in your own value 
    1 / Full access all files, excluding Application Data Folder. 
      \ "drive" 
    2 / Read-only access to file metadata and file contents. 
      \ "drive.readonly" 
      / Access to files created by rclone only. 
    3 | These are visible in the drive website. 
      | File authorization is revoked when the user deauthorizes the app. 
      \ "drive.file" 
      / Allows read and write access to the Application Data folder. 
    4 | This is not visible in the drive website. 
      \ "drive.appfolder" 
      / Allows read-only access to file metadata but 
    5 | does not allow any access to read or download file content. 
     \ "drive.metadata.readonly" 
    scope> 1
```

Una vez hecho esto nos pedirá si queremos editar las opciones avanzadas, a lo que nosotros le responderemos que no.

```shell
    Edit advanced config? (y/n) 
    y) Yes 
    n) No 
    y/n> n
```

Una vez hecho el paso anterior nos pedirá si queremos que nuestro servicio cloud se configure de forma automática, a lo que le respondemos que si y nos debería de salir una pantalla de nuestro navegador para que rellenemos nuestra credenciales de nuestra cuenta del proveedor que nosotros hayamos elegido.

```shell
    Use auto config? 
    * Say Y if not sure 
    * Say N if you are working on a remote or headless machine or Y didn't work 
    y) Yes 
    n) No 
    y/n> y 
    If your browser doesn't open automatically go to the following link: http://127.0.0.1:53682/auth 
    Log in and authorize rclone for access 
    Waiting for code... 
    Got code
```

Y por ultimo nos pedirá si queremos que nuestra cuenta sea configurada como una cuenta de equipo, a lo que le respondemos que no.

```shell
    Configure this as a team drive? 
    y) Yes 
    n) No 
    y/n> n 
    -------------------- 
    [Drive] 
    scope = drive 
    token = 
    -------------------- 
    y) Yes this is OK 
    e) Edit this remote 
    d) Delete this remote 
    y/e/d> y
```

Y ya habríamos acabo de configurar nuestro proveedor de cloud con rclone.

### Tarea 3

En esta tarea vamos a mostrar algunos de los comandos mas utiles que podemos usar con la herramienta de rclone:

* Listar ficheros: Para poder listar ficheros podemos hacerlo con el comando:

```shell
“rclone ls (nombre_servicio):[PATH]” Dentro de el comando ls existen variantes:

Listar los objetos y directorios de una forma sencilla: rclone lsf (nombre_servicio):[PATH]. Ej: rclone lsf Drive:Sistemas

Listar solo los directorios: rclone lsd (nombre_servicio):[PATH]. Ej: rclone lsd Drive:

Listar la fecha de modificacion de los ficheros: rclone lsl (nombre_servicio):[PATH]. Ej: rclone lsl DropBox:
```

* Copiar ficheros a nuestro servicio con rclone: para poder copiar ficheros desde nuestro equipo hacia el servicio cloud que tengamos usamos:

```shell
rclone copy (PATH fichero) (Servicio:PATH). 
Ej: rclone copy /home/francisco/Documentos/PLANTILLA.odt DropBox:
```

* Sincronizar Directorios con nuestro servicio: Con la sincronización lo que conseguimos es que el contenido de un directorio se copie a la ruta en nuestro servicio que le hayamos indicado por lo que, si estamos trabajando y hemos creado y modificado varios archivos y queremos guardarlos en nuestro servicio Cloud, solo debemos de sincronizar el directorio donde tengamos todos estos ficheros con una ruta que tengamos en nuestro servicio. Para realizar esta acción usamos el comando:

```shell
rclone sync (PATH a sincornizar) (Servicio:PATH) 
Ej: rclone sync ./prueba/ DropBox:pruebas/rutas
```

* Crear directorios en nuestro servicio. Podemos crear directorios que no existan en nuestro servicio desde rclone, para ello usamos el siguiente comando:

```shell
rclone mkdir (Servicio:ruta)
por ejemplo queremos crear la carpeta prueba1 en la carpeta Pruebas:
rclone mkdir DropBox:Pruebas/prueba1
```

### Tarea 4

En la siguiente tarea lo que debemos hacer es montar en un directorio local , lo ficheros de nuestro servicio cloud. Si hacemos esto lo que conseguimos es que en nuestro ordenador tengamos ese directorio montado para poder añadir o modificar cualquier fichero o directorio que nosotros veamos y al hacerlo, todo cambio realizado aparecerá en nuestro proveedor cloud. Para poder realizar estas acciones necesitamos el siguiente comando:

```shell
rclone mount (servicio:PATH a montar) (PATH donde lo montamos)
Ej: rclone mount Drive:Sistemas/Pruebas ./Documentos/Sistemas/Pruebas
```