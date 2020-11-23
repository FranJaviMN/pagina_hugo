---
title: "Certificados Digitales"
date: 2020-11-23T14:15:09+01:00
draft: true
---

# Práctica: Certificados digitales. HTTPS

## Tarea 1

1. Una vez que hayas obtenido tu certificado, explica brevemente como se instala en tu navegador favorito.

Para instalarlo en el navegador, en mi caso lo voy a instalar en firefox, solo debemos de descargarnos el certificado desde la pagina [siguiente](https://www.sede.fnmt.gob.es/certificados/persona-fisica/obtener-certificado-con-dnie/descargar-certificado/descarga-chrome).

Una vez lo tengamos nos vamos a la zona de preferencias -> Privacidad y seguridad y ahi buscamos el apartado de certificados, le damos a ver certificados y agregamos el nuestro que nos hemos descargado previamente.

2. Muestra una captura de pantalla donde se vea las preferencias del navegador donde se ve instalado tu certificado.

![certificado](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/certificado-digital.png)

3. ¿Cómo puedes hacer una copia de tu certificado?, ¿Como vas a realizar la copia de seguridad de tu certificado?. Razona la respuesta.

Desde la misma ventana donde vemos los certificados instalados, si le damos al nuestro tiene una opcion en la que nos deja hacer una copia de dicho certificado. Para realizar una copia de seguridad lo copiamos y lo debemos de guardar en distintos sitios donde sepamos que no lo vamos a perder en caso de que lo necesitemos de nuevo, en mi caso lo tengo en varios pendrives y en mi ordenador de sobremesa, a parte de en el portatil.

4. Investiga como exportar la clave pública de tu certificado.

Para ello, a la hora de realizar la copia, debemos de fijarnos que la extension del fichero que vamos a obtener sea **PKCS12** la cual solo guarda la clave publica del certificado.

## Tarea 2

Instala en tu ordenador el software [autofirma](https://firmaelectronica.gob.es/Home/Descargas.html) y desde la página de VALIDe valida tu certificado. Muestra capturas de pantalla donde se comprueba la validación.

Para ello debemos de tener instalado los siguientes paquetes para poder instalar el programa de autofirma, los paquetes a instalar son los siguientes:
```shell
#### Paquetes a instalar ####
sudo apt install default-jdk libnss3-tools openjdk-11-jdk

#### Descargamos el software de autofirma y lo instlamos ####
francisco@debian10:~/Descargas/AutoFirma_Linux$ sudo dpkg -i AutoFirma_1_6_5.deb
```

Solo debemos de dirgirnos a la pagina VALIDe y desde ahi vamos a [verificar certificado](https://valide.redsara.es/valide/validarCertificado/ejecutar.html), le damos al boton donde dice seleccionar certificado y se nos abrira la aplicacion de **Autofirma**, en ella seleccinamos nuestro certificado y rellenamos el campo de seguridad y validamos. Cuando hayamos validado nos debe de aparecer algo como lo siguiente:

![validacion certificado](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/certificado-valido.png)

## Tarea 3

1. Utilizando la página VALIDe y el programa autofirma, firma un documento con tu certificado y envíalo por correo a un compañero.

Ahoara lo que debemos hacer, desde nuestro equipo, abrimos el programa de autofirma y seleccionamos un fichero que queremos firmar, en este caso una fotografia. Para firmarlo lo vamos a hacer mediante nuestro certificado digital, por lo que, cuandio lo firmemos podemos verificar que esta firmado por nosotros en la pagina de VALIDe:

![imagen firma](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/firma-valida-documento.png)

2. Tu debes recibir otro documento firmado por un compañero y utilizando las herramientas anteriores debes visualizar la firma (Visualizar Firma) y (Verificar Firma). ¿Puedes verificar la firma aunque no tengas la clave pública de tu compañero?, ¿Es necesario estar conectado a internet para hacer la validación de la firma?. Razona tus respuestas.

Al recibir un fichero firmado con el certificado de mi compañero, vemos que si nos vamos a la pagina de VALIDe y verificamos la firma vemos que, aun sin tener la clave publica de mi compañero, puede verificar que es el ya que en el propio certificado de nuestro compañero esta tambien la clave publica de este. Al no tener internet no podemos entrar a la pagina de VALIDe por lo que no podremos verificar la firma de nuestro compañero.

![firma compañero](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/firma-celia.png)

Y aqui tenemos la firma verificada:

![firma verificada](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/firma-valida-celia.png)

3. Entre dos compañeros, firmar los dos un documento, verificar la firma para comprobar que está firmado por los dos.

Para ello vamos a usar un fichero de texto y primero lo firmo yo y luego lo firmara nuestro compañero, asi, al entrar en la pagina de VALIDe y verificar la firma del documento veremos que esta firmado por dos personas, en este caso por mi y mi compañero.

![doble firma](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/firma-ambos.png)

## Tarea 4

1. Utilizando tu certificado accede a alguna página de la administración pública )cita médica, becas, puntos del carnet,…). Entrega capturas de pantalla donde se demuestre el acceso a ellas.

Para ello vamos a consultar las becas que estan en vigor, para ello usamos la siguiente [pagina](https://sede.educacion.gob.es/sede/login/inicio.jjsp?idConvocatoria=111) en la cual vamos a elegir como forma de identificacion el certificado digital, lo cual nos llevara a una pagina como la siguiente:

![pagina certificado](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/seleccion-digital.png)

Al darle a usar el certificado digital nos saldra una ventana como la siguiente:

![certificado digital](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/certificado-entrar-pagina.png)

# HTTPS / SSL

**Antes de hacer esta práctica vamos a crear una página web (puedes usar una página estática o instalar una aplicación web) en un servidor web apache2 que se acceda con el nombre tunombre.iesgn.org.**

Para ello vamos a usar una instancia en cloud donde vamos a servir una pagina estatica de prueba.

## Tarea 1

El alumno que hace de Autoridad Certificadora deberá entregar una documentación donde explique los siguientes puntos:

1. Lo primero que debemos de hacer es preparar el directorio donde vamos a tener nuestra autoridad certificadora, para ello nos dirigimos al directorio root y creamos una carpeta llamada **CA**, y dentro de dicha carpeta creamos unsa serie de directirios:
```shell
#### Creamos la carpeta CA ####
root@debian10:~# mkdir CA

#### Dentro de CA creamos los siguiente directorios ####
root@debian10:~/CA# mkdir certsdb certreqs crl private

Donde:
    crl: Es donde los CRL se van a guardar
    private: Donde vamos a guardar nuestra clave privada del CA
    certsdb: Es donde vamos a guardar los certificados firmados
    certreqs: Donde vamos a guardar las copias de los CSRs

#### Cambiamos los permisos de private para protegerlo ####
root@debian10:~/CA# chmod 700 private/

#### Creamos el fichero para la base de datos de nuestros certificados ####
root@debian10:~/CA# touch index.txt
```

Ahora, dentro de la carpeta CA, debemos de copiar como plantilla el fichero openssl.cnf que, o bien puede estar en **/etc/openssl.con**, puede estar en **/usr/lib/ssl/openssl.cnf** o tambien puede estar en **/usr/share/ssl/openssl.cnf**. En mi caso esta plantilla estana en el directorio de **/usr/lib/ssl/openssl.cnf**. Una vez hayamos copiado este fichero en nuestro directorio **CA** vamos a buscar las siguientes lineas que son aquellas que vamos a modificar:
```shell
[ CA_default ]

dir             = /root/CA              # Where everything is kept
certs           = $dir/certs            # Where the issued certs are kept
crl_dir         = $dir/crl              # Where the issued crl are kept
database        = $dir/index.txt        # database index file.

new_certs_dir   = $dir/newcerts         # default place for new certs.

certificate     = $dir/cacert.pem       # The CA certificate
serial          = $dir/serial           # The current serial number
crlnumber       = $dir/crlnumber        # the current crl number
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem # The private key
```

En el apartado anterior estamos añadiendo las rutas donde va a buscar los distintos fichero que va a necesitra para crear nuestra entidad certificadora. A continuacion lo que debemos de modificar son los datos de nuestra autoridad certificadora, en mi caso seran los siguientes:
```shell
[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = ES
countryName_min                 = 2
countryName_max                 = 2

stateOrProvinceName             = State or Province Name (full name)
stateOrProvinceName_default     = SEVILLA

localityName                    = Locality Name (eg, city)
localityName_default            = Alcala de Guadaira

0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = Francis Corp

organizationalUnitName          = Organizational Unit Name (eg, section)
organizationalUnitName_default  = Informatica
```

LLegados a este punto ya solo debemos de comentar algunas lineas que no son interesantes para nuestro trabajo, en mi caso las siguientes:
```shell
#challengePassword              = A challenge password
#challengePassword_min          = 4
#challengePassword_max          = 20

#unstructuredName               = An optional company name
```

Ahora ya tenemos nuestro fichero bien configurado, por lo que vamos a proceder a generar un par de claves y un fichero csr el cual, despues de generar las claves y el .crs, vamos a firmarnos nosotros mismos con nuestra autoridad certificadora:
```shell
root@debian10:~/CA# openssl req -new -newkey rsa:2048 -keyout private/cakey.pem -out careq.pem -config ./openssl.cnf
Generating a RSA private key
....+++++
...+++++
writing new private key to 'private/cakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [ES]:   
State or Province Name (full name) [SEVILLA]:
Locality Name (eg, city) [Alcala de Guadaira]:
Organization Name (eg, company) [Francis Corp]:
Organizational Unit Name (eg, section) [Informatica]:
Common Name (e.g. server FQDN or YOUR name) []:debian10
Email Address []:franjaviermn17100@gmail.com
```

Ya hemos generado nuestro fichero .csr, por lo que debemos de autofirmarlo para que generemos un fichero .crt que es el que vamos a enviar a nuestros clientes como certificado de nuestra autoridad certificadora, para ello usamos el siguiente comando:
```shell
#### Creamos un directorio llamado newcerts ####
root@debian10:~/CA# mkdir newcerts

#### Firmamos nuestro fichero .csr ####
root@debian10:~/CA# openssl ca -create_serial -out cacert.pem -days 365 -keyfile private/cakey.pem -selfsign -extensions v3_ca -config ./openssl.cnf -infiles careq.pem
Using configuration from ./openssl.cnf
Enter pass phrase for private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number:
            6c:bf:83:1d:49:9a:1f:76:17:32:e1:51:13:7b:ff:9b:41:42:bb:0f
        Validity
            Not Before: Nov 23 08:25:04 2020 GMT
            Not After : Nov 23 08:25:04 2021 GMT
        Subject:
            countryName               = ES
            stateOrProvinceName       = SEVILLA
            organizationName          = Francis Corp
            organizationalUnitName    = Informatica
            commonName                = debian10
            emailAddress              = franjaviermn17100@gmail.com
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                10:48:81:98:C5:9E:EA:71:B1:7A:84:44:7B:EB:27:A2:18:8B:64:11
            X509v3 Authority Key Identifier: 
                keyid:10:48:81:98:C5:9E:EA:71:B1:7A:84:44:7B:EB:27:A2:18:8B:64:11

            X509v3 Basic Constraints: critical
                CA:TRUE
Certificate is to be certified until Nov 23 08:25:04 2021 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

Ahora ya solo queda que nuestro compañero nos envie su fichero .csr para que asi lo podamos firmar.


2. Debe recibir el fichero CSR (Solicitud de Firmar un Certificado) de su compañero, debe firmarlo y enviar el certificado generado a su compañero.

Una vez hayamos recibido el fichero .csr de nuestro compañero vamos a proceder a firmalo, para ello el fichero .csr lo movemos al directorio **certreqs** y usamos el siguiente comando:
```shell
root@debian10:~/CA# openssl ca -config openssl.cnf -out certsdb/fran.crt -infiles certreqs/fran.csr
Using configuration from openssl.cnf
Enter pass phrase for /root/CA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number:
            6c:bf:83:1d:49:9a:1f:76:17:32:e1:51:13:7b:ff:9b:41:42:bb:10
        Validity
            Not Before: Nov 23 09:06:10 2020 GMT
            Not After : Nov 23 09:06:10 2021 GMT
        Subject:
            countryName               = ES
            stateOrProvinceName       = SEVILLA
            organizationName          = Francis Corp
            organizationalUnitName    = Informatica
            commonName                = francisco.iesgn.org
            emailAddress              = franjaviermn17100@gmail.com
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                83:75:DA:09:16:5F:66:93:F9:33:91:88:12:02:87:71:E1:F7:62:4B
            X509v3 Authority Key Identifier: 
                keyid:10:48:81:98:C5:9E:EA:71:B1:7A:84:44:7B:EB:27:A2:18:8B:64:11

Certificate is to be certified until Nov 23 09:06:10 2021 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

El fichero .crt generado debemos de enviarlo a nuestro compañero junto al certificado llamado cacert.pem que es el certificado de la autoridad certificadora.

3. ¿Qué otra información debes aportar a tu compañero para que éste configure de forma adecuada su servidor web con el certificado generado?

Debemos de entregarle el fichero **cacert.pem** a nuestro compañero junto con su certificado firmado. Una vez hecho solo tendra que actualizar su pagina a https

1. Crea una clave privada RSA de 4096 bits para identificar el servidor.

En nuestra maquina de prueba llamada **prueba-https** con ip flotante **172.22.200.147** vamos a generar un par de claves para poder identificarnos, para ello vamos a usar el siguiente comando:
```shell
root@prueba-https:~# openssl genrsa 4096 > /etc/ssl/private/fran.key
Generating RSA private key, 4096 bit long modulus (2 primes)
......................................................++++
..................++++
e is 65537 (0x010001)
```

2. Utiliza la clave anterior para generar un CSR, considerando que deseas acceder al servidor tanto con el FQDN (tunombre.iesgn.org) como con el nombre de host (implica el uso de las extensiones Alt Name).

Ahora vamos a usar las claves que hemos generado anteriormente para poder crear nuestro fichero .csr que es el que le enviaremos a nuestro compañero para poder tener en nuestra pagina **https** bajo la firma de la autoridad certificadora de nuestro compañero.

Tenemos que tener en cuenta que a la hora de generar nuestro fichero .csr debemos de introducir los valores que nuestro compañero haya puesto en su fichero de **openssl.cnf**, por lo que quedaria de la siguiente forma:
```shell
root@prueba-https:~# openssl req -new -key /etc/ssl/private/fran.key -out ./fran.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:SEVILLA
Locality Name (eg, city) []:Alcala de Guadaira
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Francis Corp
Organizational Unit Name (eg, section) []:Informatica
Common Name (e.g. server FQDN or YOUR name) []:francisco.iesgn.org
Email Address []:franjaviermn17100@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

3. Envía la solicitud de firma a la entidad certificadora (su compañero).

Ahora solo debemos de hacerle llegar el fichero .csr a nuestro compañero para que este lo pueda firmar y asi poderm generar el fichero .crt y poder seguir con el trabajo.

4. Recibe como respuesta un certificado X.509 para el servidor firmado y el certificado de la autoridad certificadora.

Ahora ya he recibido como respuesta, el fichero .crt y el fichero cacert.pem de nuestro compañero, por lo que tenemos el certificado firmado pues la CA de nuestro compañero y a su vez el certificado de su CA.
```shell
debian@prueba-https:~$ ls
cacert.pem  fran.crt
```

Estos fichero debemos de moverlos al directorio **/etc/ssl/certs/**:
```shell
debian@prueba-https:~$ sudo mv cacert.pem /etc/private/certs/
debian@prueba-https:~$ sudo mv fran.crt /etc/ssl/certs/
```

5. Configura tu servidor web con https en el puerto 443, haciendo que las peticiones http se redireccionen a https (forzar https).

Para ello, en nuestro cloud debemos de tener el puerto 443 de https el cual debemos de habilitar en las reglas de seguridad:

![puerto 443](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/puerto-443.png)

Hecho esto ahora debemos de montar nuestro escenario, primero vamos a montarlo en apache2. Para ello instalamos el paquete de apache2 y para la prueba voy a copiar una pagina estatica a nuestra maquina para poder tener un referente de prueba. Nuestro fichero de configuracion, sin https, es el siguiente:
```shell
<VirtualHost *:80>

        ServerName francisco.iesgn.org

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # Redireccion a https
        #redirect / https://francisco.iesgn.org

</VirtualHost>
```

Asi, al acceder a la url **francisco.iesgn.org** veremos la siguiente pagina:

![pagina sin https](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/pagina-sinhttps.png)

Ahora vamos a confirgurarlo para poder usar https, por lo que debemos de hacer lo siguiente:
```shell
#### Fichero default ####
<VirtualHost *:80>

        ServerName francisco.iesgn.org

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # Redireccion a https
        redirect / https://francisco.iesgn.org

</VirtualHost>

#### Fichero de default-ssl ####

    ...

    ServerName francisco.iesgn.org
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    SSLEngine on
    SSLCertificateFile      /etc/ssl/certs/fran.crt
    SSLCertificateKeyFile /etc/ssl/private/fran.key
    SSLCACertificateFile    /etc/ssl/private/cacert.pem

    ...
```

Ahora solo debemos de activar los siguientes modulos de apache y el fichero **default-ssl**:
```shell
sudo a2ensite default-ssl.conf
sudo a2enmod ssl
sudo systemctl restart apache2
```

Y ahora si entramos en nuestra pagina nos saldra el siguiente mensaje:
![imagen https cuidado](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/problema-https.png)

Esto nos pasa porque no tenemos el certificado de la autoridad certificadora en nuestro navegador, por lo que vamos a añadir dicho certificado:

![pagina certificados](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/importar.png)

![importado](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/certificado.png)

Por lo que ahora si entramos en nuestra pagina ya funcionara https sin problemas:

![pagina https](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/pagina-https.png)

6. Instala ahora un servidor nginx, y realiza la misma configuración que anteriormente para que se sirva la página con HTTPS.

Ahora vamos a hacer lo mismo pero esta vez con el servidor con nginx, para ello vamos a parar el servicio de apache2 para que no haya conflictos:
```shell
debian@prueba-https:~$ sudo systemctl stop apache2.service
```

Una vez hecho esto nos dirigimos al fichero de default de nuestro nginx, y ahi vamos a realizar los cambios para poder servir la pagina con https, para ello vamos a tener la siguiente configuracion en el fichero de nginx:
```shell
#### Fichero de default en nginx ####
...

server {
        listen 80;
        server_name francisco.iesgn.org;
        return 301 https://$host$request_uri;
}

server {
        listen 443;
        server_name francisco.iesgn.org;
        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        ssl on;
        ssl_certificate /etc/ssl/certs/fran.crt;
        ssl_certificate_key /etc/ssl/private/fran.key;

        location / {
                try_files $uri $uri/ =404;
        }
}

...

#### Creamos el enlace simbolico ####
debian@prueba-https:/etc/nginx/sites-available$ ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/

#### Reiniciamos el servicio de nginx ####
debian@prueba-https:/etc/nginx/sites-available$ systemctl restart nginx.service
```

De esta forma ya tendremos nuestro fichero listo, ahora solo debemos de entrar en nuestra pagina con url **francisco.iesgn.org** y veremos que tenemos https gracias a la redireccion forzada y que no nos sale el mensaje de advertencia sino que directamente entramos en la pagina:

![pagina de https con nginx](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Seguridad/certificado/pagina-https-nginx.png)

