---
title: "Correo Ovh"
date: 2021-02-11T12:32:11+01:00
draft: true
---

# Práctica: Servidor de correos

Instala y configura de manera adecuada el servidor de correos en tu máquina de OVH, para tu dominio iesgnXX.es. El nombre del servidor de correo será mail.iesgnXX.es (Este es el nombre que deberá aparecer en el registro MX)

Para ello debemos de añadir un nuevo resgistro MX en nuestro dns de **OVH** por lo que debemos de dirigirnos a nuestro panel de control del DNS de nuestro OVH y tenemos que añadir una entrada como la siguiente:

1. Primero una entrada CNAME que apunte a nuestro servidor, en mi casi lo vamos a llamar **mail.iesgn13.es** y va a apuntar a nuestra maquina **omega.iesgn13.es**

![CNAME mail OVH](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/CNAME-mail-ovh.png)

2. Una vez tengamos nuestro CNAME configurado, vamos a añadir el registro **MX** a nuestro DNS de ovh, para ello debemos de tenerlo de la siguiente manera:

![MX mail OVH](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/MX-mail-ovh.png)

3. Una vez lo tengamos, necesitamos configurar una entrada para el protocolo **SPF** pero antes de hacerlo vamos a ver que es. 
**SPF** es una protección contra la falsificación de direcciones en el envío de correo electrónico. Identifica, a través de los registros de nombres de dominio (DNS), a los servidores de correo SMTP autorizados para el transporte de los mensajes. Este convenio busca ayudar para disminuir abusos como el spam y otros males del correo electrónico. Una vez tengamos ya un concepto de lo que es y para que se usa el protocolo **SPF**, vamos a añadirle a nuestro DNS de OVH y tenemos que añadir un registro como el siguiente:

![SPF mail OVH](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/SPF-mail-ovh-1.png)

Y a continuacion podemos poner mas datos, aunque este paso no es necesario ya que el registro ya apunta a una direccion ip y nosotros la estamos volviendio a poner:

![SPF mail ovh no](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/SPF-mail-ovh-2.png)

# Gestión de correos desde el servidor

## Tarea 1

Documenta una prueba de funcionamiento, donde envíes desde tu servidor local al exterior. Muestra el log donde se vea el envío. Muestra el correo que has recibido. Muestra el registro SPF.

Para ello, en nuestra maquina de OVH debemos de instalar el servidor de correos **Postfix** en nuestro servidor, para ello debemos de instalar el paquete llamado **postfix** por lo que lo instalamos junto a la herramienta llamada **bsd-mailx** que es un paquete con algunas utilidades para los correos electronicos.
Una vez instalado solo debemos de seguir los pasos que nos pide y poner en nuestro dominio que es **iesgn13.es**:
```shell
#### Instalamos postfix y bsd-mailx ####
debian@omega:~$ sudo apt install postfix bsd-mailx -y

#### El nombre de dominio debe de ser iesgnXX.es ####
iesgn13.es

#### Para ver nuestro nombre de dominio que tiene postfix vemos el fichero en /etc/mailname ####
debian@omega:~$ cat /etc/mailname 
iesgn13.es
```

Ahora que lo tenemos, vamos a hacer una prueba de funcionamiento donde enviemos al exterior un correo, en mi caso me lo voy a mandar a mi mismo, para ello usamos la utilidad llamada **mail** seguido de la direccion de correo electronico:
```shell
#### Mandamos el correo ####
debian@omega:~$ mail franjaviermn17100@gmail.com
Subject: Hola
Como estas?
Cc: 

#### Vemos el log para ver que se ha enviado bien ####
...
Jan 26 10:41:34 omega postfix/smtp[15270]: 3120A82629: to=<franjaviermn17100@gmail.com>, relay=gmail-smtp-in.l.google.com[64.233.167.26]:25, delay=0.8, delays=0.02/0.01/0.49/0.27, dsn=2.0.0, status=sent (250 2.0.0 OK  1611657694 b4si17228448wri.94 - gsmtp)
...
```

Y en nuestro correo tendremos el mensaje que hemos enviado en nuestra bandeja de entrada:

![mensaje ovh](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/mensaje-mail-iesgn13.png)

Y si le damos a ver el mensaje original, veremos que tendremos mas informacion del mensaje y podremos ver como el protocolo **SPF** lo ha pasado sin problemas:

![spf pass](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/spf-pass.png)

## Tarea 2

Documenta una prueba de funcionamiento, donde envíes un correo desde el exterior (gmail, hotmail,…) a tu servidor local. Muestra el log donde se vea el envío. Muestra cómo has leído el correo. Muestra el registro MX de tu dominio.

Para ello lo que vamos a hacer es contestar el correo que nos hemos enviado anteriormente para asi hacer la prueba de envio desde el exterior, para ello desde nuestra ventana de correo le respondemos con cualquier mensaje de hola y asi, cuando vayamos a nuestra maquina de ovh tendremos que ver si nos ha llegado el mensaje, para ello usamos la utilidad **mail**:
```shell
#### Despues de haber respondido el mensaje usamos mail en nuestra maquina ####
You have new mail in /var/mail/debian
debian@omega:~$ mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
 N  2 franjaviermn17100  Tue Jan 26 10:51   71/3425  Re: Hola
& 2

#### Seleccionamos el mensaje 2 y nos aparecera toda la informacion y el mensaje en si ####
Message 2:
From franjaviermn17100@gmail.com  Tue Jan 26 10:51:35 2021
X-Original-To: debian@iesgn13.es
...
References: <20210126104134.3120A82629@omega.iesgn13.es>
In-Reply-To: <20210126104134.3120A82629@omega.iesgn13.es>
From: =?UTF-8?Q?Francisco_Javier_Martin_Nu=C3=B1ez?= <franjaviermn17100@gmail.com>
Date: Tue, 26 Jan 2021 11:51:23 +0100
Subject: Re: Hola
To: Debian <debian@iesgn13.es>
Content-Type: multipart/alternative; boundary="00000000000031b26c05b9cb7099"

--00000000000031b26c05b9cb7099
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

Estoy muy bien

El mar, 26 ene 2021 a las 11:41, Debian (<debian@iesgn13.es>) escribi=C3=B3=
...

#### Vemos el log y veremos los siguientes registros ####
Jan 26 10:51:35 omega postfix/smtpd[15401]: connect from mail-wr1-f54.google.com[209.85.221.54]
Jan 26 10:51:35 omega postfix/smtpd[15401]: B9A2F82629: client=mail-wr1-f54.google.com[209.85.221.54]
Jan 26 10:51:35 omega postfix/cleanup[15411]: B9A2F82629: message-id=<CA+kZvsjEh-Wz7LrfpNFy9uV+_KoL+t2Wfi6unQN3FnqkURRBog@mail.gmail.com>
Jan 26 10:51:35 omega postfix/qmgr[10907]: B9A2F82629: from=<franjaviermn17100@gmail.com>, size=3323, nrcpt=1 (queue active)
Jan 26 10:51:35 omega postfix/local[15412]: B9A2F82629: to=<debian@iesgn13.es>, relay=local, delay=0.03, delays=0.02/0.01/0/0, dsn=2.0.0, status=sent (delivered to mailbox)
```

# Uso de alias y redirecciones

## Tarea 3

Vamos a comprobar como los procesos del servidor pueden mandar correos para informar sobre su estado. Por ejemplo cada vez que se ejecuta una tarea cron podemos enviar un correo informando del resultado. Normalmente estos correos se mandan al usuario root del servidor, para ello:
```shell
#### Entramos en el usuario root ####
debian@omega:~$ sudo su -

#### Vamos a asignar el correo a root ####
root@omega:~# crontab -e

MAILTO = root
```

Asi, cuando una accion del cron se ejecute en nuestro sistema debe de llegar a nuestro usuario **root** un correo informando de que se ha realizado una accion en el cron. Para ell vamos a crear un nuevo cron con el que vamos a ver si nos manda el correo a nuestro usuario root, para ello vamos a añadir la siguiente tarea cron a nuestro crontab:
```shell
#### Entramos en nuestro crontab ####
root@omega:~# crontab -e

#### Añadimos la siguiente linea ####
20 11 * * * apt -y update
```

La nueva tarea cron que hemos añadido hara una actualizacion de la lista de paquetes de nuetra maquina todos los dias a las 12:10am, asi que, cuando se cumpla veremos que nos debe de llegar a nuestro usuario root un mensaje de correos diciendo que se ha realizado el cron:
```shell
#### Entramos en nuestro usuario root y vemos los mensajes ####
root@omega:~# mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
"/var/mail/root": 1 message 1 new
>N  1 root@iesgn13.es    Tue Jan 26 11:20   31/1045  Cron <root@omega> apt -y update

#### Lo abrimos y vemos que es la actualizacion ####
Message 1:
From root@iesgn13.es  Tue Jan 26 11:20:02 2021
X-Original-To: root
From: root@iesgn13.es (Cron Daemon)
To: root@iesgn13.es
Subject: Cron <root@omega> apt -y update
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <MAILTO=root>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/root>
X-Cron-Env: <PATH=/usr/bin:/bin>
X-Cron-Env: <LOGNAME=root>
Date: Tue, 26 Jan 2021 11:20:02 +0000 (UTC)


WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Hit:1 http://security.debian.org/debian-security buster/updates InRelease
Hit:2 http://deb.debian.org/debian buster InRelease
Hit:3 http://deb.debian.org/debian buster-updates InRelease
Reading package lists...
Building dependency tree...
Reading state information...
All packages are up to date.
```

Ahora vamos que vemos que lo envia al usuario root, vamos a hacer que nos lo envie a nuestro correo de gmail, para ello debemos de crear un alias hacia nuestro correo electronico pero, ¿qué es un alias en correo electronico?.

Cuando se define un alias para un determinado usuario se redirige el correo que llegue a otro usuario de la misma máquina. Los alias de correo se utilizan principalmente para gestionar el correo de las “cuentas de administración” y se definen en el fichero **/etc/aliases**. En dicho fichero vemos que los correos llegan a las cuentas de root generalmente, pero queremos que lo que llegue a root nos llegue a nuestro usuario real llamado **fran**, para ello debemos de tener el siguiente contenido:
```shell
#### Contenido del fichero aliases ####
postmaster:    root
root: fran

#### Una vez modificado usamos la opcion newaliases ####
debian@omega:~$ sudo newaliases
```

Y vamos a comprobar con el cron que de verdad nos llegan los correos al usuario fran, para ello modificamos el cron y lo ponemos a otra hora:
```shell
#### Modificamos el cron ####
35 11 * * * apt -y update
```

Y ahora esperamos a que nos llegue el correo una vez llegue la hora que le hemos indicado.

```shell
fran@omega:~$ mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
"/var/mail/fran": 1 message 1 new
>N  1 root@iesgn13.es    Tue Jan 26 11:35   31/1045  Cron <root@omega> apt -y update
& 
```

Y como vemos nos ha llegado de forma correcta, ahora debemos de conseguir que nos llegue a nuestro correo electronico de gmal, para ello lo que debemos de hacer es una **redireccion** para que los correos que llegue a ese usuario los envie a nuestro correo, para ello debemos de crear un fichero llamado **.forward** en la carpeta home del usuario en cuestión, en este caso en **fran** con el correo electronio en su interior o una lista de correos y, al realizarse una tarea cron nos debe de llegar un correo a nuestro correo de gmail:

![correo gmail cron](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/correo%20cron.png)


# Para luchar contra el SPAM

## Tarea 5

Configura de manera adecuada Postfix para que tenga en cuenta el registro SPF de los correos que recibe. Muestra el log del correo para comprobar que se está haciendo el testeo del registro SPF.

Para ello debemos de hacer que nuestro correo verifique los correos que llegue a nuestra maquina, para ello vamos a necesitar un paquete adicional llamado **postfix-policyd-spf-python** el cual vamos a instalar en nuestra maquina con su correo postfix. Cuando lo instalemos debemos de indicar en el fichero **/etc/postfix/master.cf** que cree un socket UNIX para la comprobación:
```shell
#### Instalamos el paquete necesario ####
debian@omega:~$ sudo apt install postfix-policyd-spf-python

#### Añadimos la siguiente linea en el fichero /etc/postfix/master.cf ####
policyd-spf  unix  -       n       n       -       0       spawn     user=policyd-spf argv=/usr/bin/policyd-spf
```

Ahora debemos de indicar en el fichero **/etc/postfix/main.cf** que haga la comprobación en ese socke unix, por lo que debemos de añadir las siguientes lineas:
```shell
#### Modificamos el fichero /etc/postfix/main.cf ####
policyd-spf_time_limit = 3600
smtpd_recipient_restrictions = check_policy_service unix:private/policyd-spf
```

Ahora vamos a reiniciar el servicios de postfix y vamos a enviar un correo a nuestro servidor para ver si hace la comprobacion spf:
```shell
#### Miramos en el log de /var/log/mail.log ####
Jan 26 17:09:35 omega postfix/smtpd[22650]: connect from mail-wr1-f42.google.com[209.85.221.42]
Jan 26 17:09:35 omega policyd-spf[22657]: prepend Received-SPF: Pass (mailfrom) identity=mailfrom; client-ip=209.85.221.42; helo=mail-wr1-f42.google.com; envelope-from=franjaviermn17100@gmail.com; receiver=<UNKNOWN> 
Jan 26 17:09:35 omega postfix/smtpd[22650]: 9138A82630: client=mail-wr1-f42.google.com[209.85.221.42]
Jan 26 17:09:35 omega postfix/cleanup[22660]: 9138A82630: message-id=<CA+kZvsieXaGEX86PzAuFN5XDq1fbUfmZHH0-Km=Z+1dZt60HXw@mail.gmail.com>
Jan 26 17:09:35 omega postfix/qmgr[22592]: 9138A82630: from=<franjaviermn17100@gmail.com>, size=3496, nrcpt=1 (queue active)
Jan 26 17:09:35 omega postfix/local[22661]: 9138A82630: to=<debian@iesgn13.es>, relay=local, delay=0.32, delays=0.31/0.01/0/0, dsn=2.0.0, status=sent (delivered to mailbox)
Jan 26 17:09:35 omega postfix/qmgr[22592]: 9138A82630: removed
Jan 26 17:09:35 omega postfix/smtpd[22650]: disconnect from mail-wr1-f42.google.com[209.85.221.42] ehlo=2 starttls=1 mail=1 rcpt=1 bdat=1 quit=1 commands=7
```

Ya tenemos el verificador de spf en nuestro servidor de correos postfix configurado.

## Tarea 6

Configura un sistema antispam. Realiza comprobaciones para comprobarlo.

Para ello vamos a instalar el sistema antispam llamado **SpamAssassin** por lo que debemos de instalar los paquetes **spamassassin** y **spamc**:
```shell
#### Instalamos los paquetes necesarios ####
debian@omega:~$ sudo apt install spamc spamassassin
```

Una vez instalados los paquetes que necesitamos tenemos que añadir las siguientes lineas en el fichero **/etc/postfix/master.cf** que iremos añadiendo poco a poco:
```shell
#### Añadimos la siguiente linea para que queda algo como lo siguiente ####
smtp      inet  n       -       y       -       -       smtpd
-o content_filter=spamassassin

#### Añadimos esta linea quedanos algo parecido a lo siguiente ####
submission inet n       -       y       -       -       smtpd
-o content_filter=spamassassin

#### Al final del fichero debemos de añadir la siguiente linea ####
spamassassin unix -     n       n       -       -       pipe
  user=debian-spamd argv=/usr/bin/spamc -f -e /usr/sbin/sendmail -oi -f ${sender} ${recipient}
```

Una vez hayamos configurado el fichero **/etc/postfix/master.cf** debemos de reiniciar el servicio de postfix para ver si esta todo correcto, que deberia de estarlo. 

Ahora vamos a pasar a la configuracion de **Spamassassin**, lo primero va a ser irnos al fichero **/etc/spamassassin/local.cf** y debemos de descomentar una linea como la siguiente:
```shell
#### Linea a descomentar ####
rewrite_header Subject *****SPAM*****
```

Esto lo hacemos para que marque como SPAM los mensajes que lo sean. Y ahora vamos a habilitar y reiniciar los distintos servicios:
```shell
#### Reiniciamos el servicios de postfix ####
debian@omega:~$ sudo systemctl restart postfix.service

#### Habilitamos el servicio de spamassassin ####
debian@omega:~$ sudo systemctl enable spamassassin.service

#### Iniciamos el servicios de spamassassin ####
debian@omega:~$ sudo systemctl start spamassassin.service
```

Y a la hora de enviar un mensaje con spam debe de salirnos los mensajes siguiente en el log de postfix:
```shell
Jan 26 18:01:57 omega spamd[24888]: spamd: identified spam (999.9/5.0) for debian-spamd:112 in 0.2 seconds, 3818 bytes.
Jan 26 18:01:57 omega spamd[24888]: spamd: result: Y 999 - DKIM_SIGNED,DKIM_VALID,DKIM_VALID_AU,FREEMAIL_FROM,GTUBE,HTML_MESSAGE,RCVD_IN_MSPIKE_H2,SPF_PASS,URIBL_BLOCKED scantime=0.2,size=3818,user=debian-spamd,uid=112,required_score=5.0,rhost=::1,raddr=::1,rport=38494,mid=<CA+kZvsg23JEMj32M6Frs1bc1UZ_2kSNeEpGKzP+wQ5wcdkQN-A@mail.gmail.com>,autolearn=no autolearn_force=no
Jan 26 18:01:57 omega postfix/pipe[25543]: B6A05826C7: to=<debian@iesgn13.es>, relay=spamassassin, delay=0.61, delays=0.32/0.01/0/0.29, dsn=2.0.0, status=sent (delivered via spamassassin service)
```

Asi, a la hora de ver nuestro correo lo veremos asi:
```shell
#### vemos nuestro correo ####
debian@omega:~$ mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
"/var/mail/debian": 2 messages 1 new 2 unread
 U  1 franjaviermn17100  Tue Jan 26 17:55   79/4205  *****SPAM***** hola
>N  2 franjaviermn17100  Tue Jan 26 18:01  102/5378  *****SPAM***** Fwd: hola
```

Vemos que tiene la cadena *****SPAM***** en el mensaje que el considera spam.

## Tarea 7

Configura un sistema antivirus. Realiza comprobaciones para comprobarlo.

Para ell vamos a usar como sistema de antivirus llamado clamAV, para ello debemos de instalar los siguientes paquetes en nuestro sistema:
```shell
#### Instalamos los paquetes necesarios ####
debian@omega:~$ sudo apt install clamav clamav-freshclam clamsmtp

#### Reiniciamos el servicio llamado clamsmtp.service ####
debian@omega:~$ sudo systemctl restart clamsmtp.service

#### Reiniciamos el servicio de postfix ####
debian@omega:~$ sudo systemctl restart postfix
```

Una vez hayamos hecho los pasos anteriores debemos de irnos al fichero de configuracion de **/etc/clamsmtpd.conf** y ahi debemos de buscar las lineas **OutAddress:** y la linea **Listen:** y debemos de dejarlo de la siguiente forma:
```shell
#### Fichero /etc/clamsmtpd.conf ####
OutAddress: 10026
...
Listen: 127.0.0.1:10025
```

Una vez hecho el anterior cambio debemos de irnos a nuestro fichero **/etc/postfix/main.cf** y vamos a añadir las siguientes lineas al final de este fichero:
```shell
#### Lineas a añadir ####
content_filter = scan:127.0.0.1:10025
receive_override_options = no_address_mappings
```

Ahora nos dirigimos al fichero **/etc/postfix/master.cf** y debemos de añadir la lineas en la que le indicamos a nuestro servicio de correos donde debe de realizar la consulta sobre si es o no es un correo con algun virus, por lo que le vamos a añadir las siguientes lineas:
```shell
#### Añadimos las lineas siguientes a nuestro fichero ####
scan      unix  -       -       n       -       16      smtp
        -o smtp_send_xforward_command=yes
...
127.0.0.1:10026      inet  n       -       n       -       16      smtpd
        -o content_filter=
        -o receive_override_options=no_unknown_recipient_checks,no_header_body_checks
        -o smtpd_helo_restrictions=
        -o smtpd_client_restrictions=
        -o smtpd_sender_restrictions=
        -o smtpd_recipient_restrictions=permit_mynetworks,reject
        -o mynetworks_style=host
        -o smtpd_authorized_xforward_hosts=127.0.0.0/8
```

Asi, al realizar la configuración vamos a reiniciar los servicios de **clamsmtp.service** y el servicio de **postfix** asi, ahora si recibimos un correo que nuestro antivirus detecte como un virus, lo descartará y no lo tendremos en nuestra bandeja de entrada. Para poder comprobar si funciona correctamente vamos a enviar un mensaje de prueba para ver si lo detecta como virus, para ello seguimos los pasos:
```shell
#### En nuestro correo enviamos el siguiente mensaje ####
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H* 

#### Miramos los log de postfix para ver que hace con ese correo ####
Jan 27 17:45:32 omega postfix/smtpd[17782]: connect from mail-wr1-f42.google.com[209.85.221.42]
Jan 27 17:45:32 omega policyd-spf[17789]: prepend Received-SPF: Pass (mailfrom) identity=mailfrom; client-ip=209.85.221.42; helo=mail-wr1-f42.google.com; envelope-from=franjaviermn17100@gmail.com; receiver=<UNKNOWN> 
...
Jan 27 17:45:32 omega spamd[24888]: spamd: clean message (-0.1/5.0) for debian-spamd:112 in 0.2 seconds, 2794 bytes.
Jan 27 17:45:32 omega spamd[24888]: spamd: result: . 0 - DKIM_SIGNED,DKIM_VALID,DKIM_VALID_AU,FREEMAIL_FROM,HTML_MESSAGE,RCVD_IN_MSPIKE_H2,SPF_PASS,TVD_SPACE_RATIO scantime=0.2,size=2794,user=debian-spamd,uid=112,required_score=5.0,rhost=::1,raddr=::1,rport=38584,mid=<CA+kZvsjbkMPbcUYhSfzzbGYCtVTZ07bVNT1+YvB=Ce2Ln9J_fQ@mail.gmail.com>,autolearn=ham autolearn_force=no
Jan 27 17:45:32 omega postfix/pipe[17795]: 9C5A0826CE: to=<debian@iesgn13.es>, relay=spamassassin, delay=0.51, delays=0.27/0.01/0/0.23, dsn=2.0.0, status=sent (delivered via spamassassin service)
Jan 27 17:45:32 omega postfix/qmgr[17773]: 9C5A0826CE: removed
...
Jan 27 17:45:32 omega clamsmtpd: 100002: accepted connection from: 127.0.0.1
Jan 27 17:45:32 omega spamd[24887]: prefork: child states: II
Jan 27 17:45:32 omega postfix/smtpd[17803]: connect from localhost[127.0.0.1]
Jan 27 17:45:32 omega postfix/smtpd[17803]: ECB66826CE: client=localhost[127.0.0.1]
Jan 27 17:45:32 omega postfix/smtp[17801]: D79E0826D0: to=<debian@iesgn13.es>, relay=127.0.0.1[127.0.0.1]:10025, delay=0.09, delays=0.01/0.02/0.07/0.01, dsn=2.0.0, status=sent (250 Virus Detected; Discarded Email)
Jan 27 17:45:32 omega postfix/qmgr[17773]: D79E0826D0: removed
```

Y en este log podemos ver como entra en juego el servicio de **spamassassin** y detecta que no contiene spam, por lo que no lo marca como spam asi que pasa y entra en la comprobacion del antivirus **clamAV** y detecta que es un virus, por lo que lo descarta y no lo encontraremos en nuestra bandeja de correos en nuestro usuario.

# Gestión de correos desde un cliente

## Tarea 8

Configura el buzón de los usuarios de tipo Maildir. Envía un correo a tu usuario y comprueba que el correo se ha guardado en el buzón Maildir del usuario del sistema correspondiente. Recuerda que ese tipo de buzón no se puede leer con la utilidad mail.

Para ello debemos de configurar el buzon de correos Maildir, para ello debemos de irnos al fichero de configuración de nuestro postfix y ahi debemos de indicar la siguiente linea:
```shell
#### Añadimos la linea siguiente en /etc/postfix/main.cf #### 
home_mailbox = Maildir/
```

Asi reiniciamos el servicio y cuando nos llegue un correo veremos que se nos creara una carpeta en el usuario en cuestion llamada **Maildir** pero claro, ahora la utilidad de **mail** ya no nos funciona por lo que vamos a instalar ahora otro cliente, por ejemplo **mutt**, para ell seguimos los siguientes pasos:
```shell
#### Instalamos mutt ####
debian@omega:~$ sudo apt install mutt

#### Creamos en el directorio del usuario un fichero .muttrc ####
debian@omega:~$ nano .muttrc

set mbox_type=Maildir
set folder="~/Maildir"
set mask="!^\\.[^.]"
set mbox="~/Maildir"
set record="+.Sent"
set postponed="+.Drafts"
set spoolfile="~/Maildir"
```

De esta forma ya tendremos activado el nuevo cliente y con el fichero .muttrc le estamos diciendo al cliente donde debe de mirar los correos que lleguen y desde donde los debe de enviar, asi, si enviamos un correo de prueba debemos de verlo con solo poner en la terminal el comando **mutt**:

![mutt mensaje](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/mutt-mensaje.png)

Y si queremos enviar un mensaje desde nuestro nuevo cliente mutt, podemos usar la siguiente estructura:
```shell
echo "Cuerpo del mensaje" | mutt -s "Titulo del mensaje" correo@destino.com
```


## Tarea 9

Instala configura dovecot para ofrecer el protocolo IMAP. Configura dovecot de manera adecuada para ofrecer autentificación y cifrado.

Para realizar el cifrado de la comunicación crea un certificado en LetsEncrypt para el dominio mail.iesgnXX.es. En mi caso vamos a usar el protocolo **IMAPS** por lo que vamos a necesitar los certificados de letsEncrypt, para ello vamos a generarlos para el dominio mail.iesgn13.es por lo que vamos a necesitar seguir el tutorial de instalación del [Certbot](https://certbot.eff.org/).

Una vez instalado Certbot en nuestra maquina ejecutamos el siguiente comando para generar los certificados necesarios, debemos de tener en cuenta que si tenemos algun servidor web iniciado en la maquina debemos de parar el servicio para que certbot pueda realizar el challenge:
```shell
#### Generamos los certificados que necesitamos ####
debian@omega:~$ sudo certbot certonly --standalone -d mail.iesgn13.es
```

Asi ya tendremos los certificados generados por lo que ya podemos seguir con la instalacion de **dovecot**, para ell debemos de instalar en nuestra maquina los paquetes llamados **dovecot-imapd, dovecot-pop3d y dovecot-common**. Una vez instalado debemos de configurarlos de la manera que vamos a mostrar a continuacion y le vamos a añadir el protocolo **IMAPS**:
```shell
#### Instalamos los paquetes necesarios ####
debian@omega:~$ sudo apt install dovecot-imapd dovecot-pop3d dovecot-common

#### Configuramos el fichero llamado /etc/dovecot/conf.d/10-auth.conf ####
disable_plaintext_auth = no
...

#### Aseguramos que las lineas esten de esta forma ####
mail_location = maildir:~/Maildir
...
mail_privileged_group = mail
...
mail_access_groups = mail
```

Ahora lo que vamos a hacer es activar ssl en nuestro dovecot con el certificado que hemos generado con LetsEncrypt, para ello debemos de tener en cuenta que las rutas donde se encuentran la clave y el certificado estan en **/etc/letsencrypt/live/{nuestro-dominio}**. Asi nos vamos al fichero **/etc/dovecot/conf.d/10-ssl.conf** y ahi debemos de tener la siguiente configuracion:
```shell
#### Descomentamos las lineas siguientes ####
ssl = yes
...
ssl_cert = </etc/letsencrypt/live/mail.iesgn13.es/fullchain.pem
ssl_key = </etc/letsencrypt/live/mail.iesgn13.es/privkey.pem

#### Reiniciamos el servicio de dovecot ####
sudo systemctl restart dovecot.service
```

Asi, a la hora de entrar en nuestro cliente de correos remoto, como puede ser thunderbird, debemos de poner los siguientes elementos:

![ssl thunderbird](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/ssl-thunderbird.png)

Y a la hora de entrar en nuestro cliente veremos ya los correos de nuestro usuario:

![correos thunderbird](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Servicios/mail-ovh/recibido-thunderbird.png)