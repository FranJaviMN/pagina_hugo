---
title: "Prueba Django"
date: 2020-12-10T14:08:25+01:00
draft: true
---

# Despliegue de aplicaciones python

## Tarea 1 Entorno de desarrollo

Vamos a desarrollar la aplicación del tutorial de django 3.1. Vamos a configurar tu equipo como entorno de desarrollo para trabajar con la aplicación, para ello:

1. Vamos a desarrollar la aplicación del tutorial de django 3.1. Vamos a configurar tu equipo como entorno de desarrollo para trabajar con la aplicación, para ello: https://github.com/josedom24/django_tutorial.

Por lo que nos vamos a ese repositorio y lo que vamos a hacer es crear un **fork** a nuestro repositorio para posteriormente clonarlo en nuestra maquina:
```shell
francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial$ git clone git@github.com:FranJaviMN/django_tutorial.git
```

2. Crea un entorno virtual de python3 e instala las dependencias necesarias para que funcione el proyecto (fichero requirements.txt).

Para ello necesitamos un entorno virtual, solo seguimos los siguientes pasos:
```shell
#### Creamos el entorno virtual llamado trabajo-despliegue ####
francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial$ python3 -m venv ./trabajo-despliegue

#### Ahora iniciamos nuestro entorno virtual e instalamos todas las dependencias del fichero requirements.txt ####
francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial$ source trabajo-despliegue/bin/activate

#### Clonamos las dependencias ####
(trabajo-despliegue) francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial/django_tutorial$ pip install -r requirements.txt

#### Comprobamos que son las mismas ####
(trabajo-despliegue) francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial/django_tutorial$ pip list
Package       Version
------------- -------
asgiref       3.3.0  
Django        3.1.3  
pip           18.1   
pkg-resources 0.0.0  
pytz          2020.4 
setuptools    40.8.0 
sqlparse      0.4.1  
```

3. Comprueba que vamos a trabajar con una base de datos sqlite (django_tutorial/settings.py). ¿Cómo se llama la base de datos que vamos a crear?

Solo debemos de visualizar el fichero y buscar el nombre de la base de datos, en este caso su nombre es **db.sqlite3**

4. Crea la base de datos: python3 manage.py migrate. A partir del modelo de datos se crean las tablas de la base de datos.

Para ello solo debemos de ejecutar el siguiente comando: *python3 manage.py migrate* que nos creara la base de datos a partir del fichero llamado **manage.py**

5. Crea un usuario administrador:

Para ello debemos de usar el comando *python3 manage.py createsuperuser* y ya lo creamos con un nombre y una contraseña.

6. Ejecuta el servidor web de desarrollo y entra en la zona de administración (\admin) para comprobar que los datos se han añadido correctamente.

Para ello nos dirijimos a donde esta el fichero llamado *app.py* y ejecutamos el fichero:
```shell
(trabajo-despliegue) francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial/django_tutorial$ python3 manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
November 12, 2020 - 17:34:54
Django version 3.1.3, using settings 'django_tutorial.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

Al entrar en la url http://127.0.0.1:8000/admin tendremos un login y luego entraremos en la zona de administracion:

![Zona admin](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/tutorial_django/admin-zone-dj.png)

7. Crea dos preguntas, con posibles respuestas.

Para ello solo debemos de dirigirnos hacia la ventana donde dice *Questions* y ahi debemos de añadir 2 preguntas con sus posibles respuestas.

![Zona de preguntas creadas](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/tutorial_django/pregunta-admin.png)

8. Comprueba en el navegador que la aplicación está funcionando, accede a la url \polls.

Para ello entramos, con el servidor activado, a la url llamada http://127.0.0.1:8000/polls y nos saldra una pagina con las preguntas que hemos creado en la zona admin, si entramos en ellas veremos que podemos votar por las respuestas que nosotros consideremos.

![polls de djnago](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/tutorial_django/preguntas-polls.png)

## Tarea 2: Entorno de producción

Vamos a realizar el despliegue de nuestra aplicación en un entorno de producción, para ello vamos a utilizar una instancia del cloud, sigue los siguientes pasos:

1. Instala en el servidor los servicios necesarios (apache2). Instala el módulo de apache2 para ejecutar código python.

Para ello debemos de instalar los paquetes de **apache2** y el paquete llamado **libapache2-mod-wsgi-py3** que es el modulo que vamos a usar para que apache2 pueda ejecutar el codigo python:

```shell
#### Instalamos los paquetes necesarios ####
debian@practica-django:~$ sudo apt install apache2 libapache2-mod-wsgi-py3
```

2. Clona el repositorio en el DocumentRoot de tu virtualhost.

Nosotros vamos a usar como documentroot el directorio **/srv/**, por lo que si queremos clonar necesitaremos el paquete de **git** asi que procedemos a instalarlo:
```shell
#### Instalamos git ####
debian@practica-django:~$ sudo apt install git

#### Clonamos el repositorio ####
debian@practica-django:/srv$ sudo git clone https://github.com/FranJaviMN/django_tutorial.git
```

3. Crea un entorno virtual e instala las dependencias de tu aplicación

Para ello necesitamos los paquetes para poder crear entornos virtuales python, por lo que debemos de instalar el siguiente paquete:
```shell
#### Instalamos los paquetes necesarios ####
debian@practica-django:~$ sudo apt install python3-venv

#### Creamos el entorno virtual ####
debian@practica-django:~$ python3 -m venv ./trabajo-despliegue

#### Instalamos las dependencias del programa en el entorno virtual ####
(trabajo-despliegue) debian@practica-django:~$ pip install -r /srv/django_tutorial/requirements.txt
```

4. Instala el módulo que permite que python trabaje con mysql:

Para ello debemos de instalar el paquete llamado **python3-mysqldb**(lo instalamos en la maquina de prueba, no en el entorno virtual)
```shell
#### En la maquina de prueba ####
debian@practica-django:~$ sudo apt install python3-mysqldb

#### En nuestro entorno virtual instalamos el paquete para conectar con mysql ####
(trabajo-despliegue) debian@practica-django:~$ pip install mysql-connector-python
```

5. Crea una base de datos y un usuario en mysql.

Para ello entramos como root en mysql y ahi creamos un usuario llamado *fran* con una contraseña *usuario* y tambien creamos la base de datos a la que llamaremos django:
```shell
#### Instalamos mariadb ####
debian@practica-django:~$ sudo apt install mariadb-server mariadb-client

#### Entramos como superusuario ####
debian@practica-django:~$ sudo mysql -u root -p

#### Creamos la base de datos django ####
MariaDB [(none)]> create database django;

#### Creamos un usuario llamado fran de contraseña usuario ####
MariaDB [(none)]> create user 'fran'@'localhost' identified by 'usuario';

#### Le damos todos los privilegios sobre la base de datos django ####
MariaDB [(none)]> GRANT ALL PRIVILEGES ON django.* to 'fran'@'localhost';
```

De esta forma ya tendremos nuestra base de datos con usuario *fran*.

6. Configura la aplicación para trabajar con mysql, para ello modifica la configuración de la base de datos en el archivo settings.py:

Para ello nos debe de quedar una sintaxis como la siguiente:
```python
DATABASES = {
    'default': {
        'ENGINE': 'mysql.connector.django',
        'NAME': 'django',
        'USER': 'fran',
        'PASSWORD': 'usuario',
        'HOST': 'localhost',
        'PORT': '',         
    }
}
```

Ahora solo debemos de migrar la base de datos como hicimos anteriormente, pero esta vez se migrara a la base de datos de mysql:
```shell
#### Entramos en el entorno virtual python ####
debian@practica-django:~$ source trabajo-despliegue/bin/activate

#### Migramos la base de datos ####
(trabajo-despliegue) debian@practica-django:~$ python3 /srv/django_tutorial/manage.py migrate

#### Comprobamos en nuestra base de datos si esta migrada correctamente ####
MariaDB [django]> show tables;
+----------------------------+
| Tables_in_django           |
+----------------------------+
| auth_group                 |
| auth_group_permissions     |
| auth_permission            |
| auth_user                  |
| auth_user_groups           |
| auth_user_user_permissions |
| django_admin_log           |
| django_content_type        |
| django_migrations          |
| django_session             |
| polls_choice               |
| polls_question             |
+----------------------------+
12 rows in set (0.001 sec)
```

7. Crea un usuario administrador: python3 manage.py createsuperuser.

Ahora, como en el ejercicio anterior, vamos a proceder a crear un usuario administrador, para ello usamos el entorno virtual de python y usamos el siguiente comando:
```shell
(trabajo-despliegue) debian@practica-django:~$ python3 /srv/django_tutorial/manage.py createsuperuser

Username (leave blank to use 'debian'): fran
Email address: 
Password: 
Password (again): 
```

De esta forma ya tendremos creado nuestro superusuerio con nombre fran.

8. Configura un virtualhost en apache2 con la configuración adecuada para que funcione la aplicación. El punto de entrada de nuestro servidor será django_tutorial/django_tutorial/wsgi.py.

Para ello vamos a usar el virtualhost por defecto que viene creado en apache2, partiendo de esa base nuestro fichero de configuracion debe de tener el siguiente contenido:
```shell
#### Contenido del fichero 000-default.conf
<VirtualHost *:80>

        ServerName www.prueba-django.org
        ServerAdmin webmaster@localhost
        DocumentRoot /srv/django_tutorial

        WSGIDaemonProcess prueba_django user=www-data group=www-data processes=1 threads=5 python-path=/srv/django_tutorial:/home/debian/trabajo-despliegue/lib/python3.7/site-packages
        WSGIScriptAlias / /srv/django_tutorial/django_tutorial/wsgi.py

        <Directory /srv/django_tutorial>
        WSGIProcessGroup prueba_django
        WSGIApplicationGroup %{GLOBAL}
        Require all granted
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

#### Para entrar dejamos que cualquier host pueda acceder a la pagina ####
debian@practica-django:/srv/django_tutorial/django_tutorial$ sudo nano settings.py

y aqui debemos de tener lo siguiente:

...
ALLOWED_HOSTS = ['*']
...
```

De esta forma si entramos en la pagina desde nuestro cliente veremos que nos sale ya la zona de login y polls pero sin su oja de estilos, este problema lo vamos a solucionar en el siguiente apartado.

9.  asegurarte que el contenido estático se está sirviendo: ¿Se muestra la imagen de fondo de la aplicación? ¿Se ve de forma adecuada la hoja de estilo de la zona de administración?

Para poder servir el contenido estatico para la url **/polls**, en nuestro fichero de configuracion de nuestro virtualhost y crear un alias como el siguiente para poder servir el contenido estatico:
```shell
#### COntenido en el fichero del virtualhost ####
...
            Alias /static/ /srv/django_tutorial/polls/static/

                <Directory /srv/django_tutorial/polls/static>
                Require all granted
                </Directory>
...
```

Ahora si nosotros entramos en **/polls** veremos que se estará sirviendo el contenido estatico correctamente. Ahora debemos de hacer lo mismo pero para **/admin**, para ello, vamos a copiar el contenido del directorio **trabajo-despliegue/lib/python3.7/site-packages/django/contrib/admin/static/admin** al directorio **/srv/django_tutorial/polls/static** por lo que, al entrar en la pagina /admin vamos a ver la hoja de estilos correspondiente a la pagina admin y pagina polls.

![hoja de estilos admin](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/tutorial_django/admin-estilos.png)


10. Desactiva en la configuración (fichero settings.py) el modo debug a False. Para que los errores de ejecución no den información sensible de la aplicación.

Para ello nos vamos al fichero **/srv/django_tutorial/django_tutorial/settings.py** y dentro buscamos una linea que se llama de la siguiente forma:
```python
...
# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = False
...
```

De tal forma ya tendremos nuestra pagina funcionando perfectamente.

## Tarea 3: Modificación de nuestra aplicación

Vamos a realizar cambios en el entorno de desarrollo y posteriormente vamos a subirlas a producción. Vamos a realizar tres modificaciones (entrega una captura de pantalla donde se ven cada una de ellas). Recuerda que primero lo haces en el entrono de desarrollo, y luego tendrás que llevar los cambios a producción:

1. Modifica la página inicial donde se ven las encuestas para que aparezca tu nombre: Para ello modifica el archivo django_tutorial/polls/templates/polls/index.html.

Para ello, en nuestro entorno de desarrollo hemos añadido la siguiente linea:
```python
...
{% if latest_question_list %}
    <p>Esta pagina de preguntas es de Fran</p>
    <ul>
    {% for question in latest_question_list %}
...
```

Entonces al entrar en nuestra pagina en desarrollo tendremos el siguiente contenido en polls:

![pagina polls modificada](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/tutorial_django/fran-polls-desarrollo.png)

Ahora debemos de llevarnos lo que hemos modificado a nuestro entorno de produccion, para ello debemos de subir los cambios a git y desde el entorno de produccion debemos de hacer un pull de los cambios:
```shell
#### Subimos los cambios ####
francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial/django_tutorial$ git commit -am "Cambio en polls"

francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial/django_tutorial$ git push

#### Añadimos los cambios en nuestro entorno de produccion ####
debian@practica-django:/srv/django_tutorial$ git pull
```

Si entramos en nuestra pagina de prueba veremos que el cambio se ha añadido:

![cambio en produccion](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/tutorial_django/prueba-produccion.png)

2. Modifica la imagen de fondo que se ve la aplicación.

Para ello vamos a escoger una imagen para nuestro fondo, da igual cual sea, pero cuando la tengamos tenemos que reemplazarla por **django_tutorial/polls/static/polls/images/background.jpg** con el mismo nombre, de tal forma que al iniciar el servidor en desarrollo veremos la pagina cambiada con un fonco distinto:

![cambio de fondo](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/tutorial_django/cambio-fondo-desarrollo.png)

Una vez lo tengamos nos llevamos los cambios a produccion mediante git:
```shell
#### Añadimos la imagen nueva ####
francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial/django_tutorial$ git add polls/

#### Nuevo commit ####
francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial/django_tutorial$ git commit -am "Nuevo fondo"

francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial/django_tutorial$ git push
```

En nuestro entorno de produccion debemos de traernos los cambios que hemos hecho en desarrollo con **git pull** para poder ver los cambios en la pagina:

![fondo produccion](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/tutorial_django/fondo-produccion.png)

3. Vamos a crear una nueva tabla en la base de datos, para ello sigue los siguientes pasos:

    * Añade un nuevo modelo al fichero polls/models.py:
    ```python
    class Categoria(models.Model):	
  	    Abr = models.CharField(max_length=4)
  	    Nombre = models.CharField(max_length=50)

  	def __str__(self):
  		return self.Abr+" - "+self.Nombre 		
    ```

    * Crea una nueva migración: python3 manage.py makemigrations.
    ```shell
    (trabajo-despliegue) francisco@debian10:~/Documentos/Implantacion/Python/Django tutorial/django_tutorial$ python3 manage.py makemigrations
    ```

    * En la maquina, hacemos un pull y hacemos la nueva migracion:
    ```shell
    (trabajo-despliegue) debian@django-prueba:/srv/django_tutorial$ python3 manage.py migrate
    ```

De tal forma que, si entramos en la pagiande /admin veremos una nueva pestaña llamada **Categorias**:

![imagen categorias](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Implantacion/tutorial_django/categorias-admin.png)