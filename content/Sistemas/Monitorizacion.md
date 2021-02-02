---
title: "Monitorizacion"
date: 2021-02-01T19:22:06+01:00
draft: true
---

# Monitorización 

La monitorización de los sistemas que podamos tener en producción es una parte muy importante a la hora de hacer que todos los servicios que ofrecemos con nuestros sitemas funcionen correctamente y, si en el caso de que hubiera un fallo o algun error, gracias a la monitorización de nuestros sitemas, lograr encontrar y solucionar el error.

Hoy dia podemos encontrar una gran cantidad de sistemas de monitorización, ya sean mediante la recolención de metricas, la monitorizacion de los sistemas o usar una gestion de logs centralizada de todos los equipos. En este mar de opciones podemos escoger entre muchos sistemas que nos permiten hacer una monitorización de nuestros equipos.

Debemos de tener en cuenta que a la hora de usar la monitorización debemos de tener 3 cosas fundamentales que son, **un recolector de metricas**, **una base de datos** y **un visualizador**, con estos tres componentes podemos hacer una monitorización muy completa de todos nuestros equipos. 

El recolector, se encarga de recoger todos los datos de nuestra máquina, continuamente, en tiempo real, para así mandarlo a la base de datos, que será quien lo almacene, y por último el visualizador es quien nos muestra una gráfica con los datos.

Entre todas las opciones a nuestras disposición he optado por usar el sistema **Prometheus** y **Grafana**. De estos dos componentes el visualizador va a ser **Grafana** mientras que **Prometheus** va a actuar como base de datos y como recolector.

## Prometheus

Prometheus es un sistema de monitorización de métricas, y generador de alertas, creado por SoundCloud, los que a su vez son ex-empleados de Google. Estos se inspiraron en el sistema de monitorización Borgmon, que se utilizaba para Kubernetes cuando este todavía no era un producto público. Este proyecto forma parte de la Cloud Native Computing Foundation.

Componentes que aporta Prometheus:

* El servidor de Prometheus para consultar y almacenar la series de datos.
* Un Pushgateway para permitir que los trabajos efímeros y por lotes expongan sus métricas a Prometheus.
* Exporters útil para casos donde no es factible instrumentar un sistema dado con métricas Prometheus directamente.
* Un sistema de manejo de alarmado.
* Un sistema de discovery.

Los tipos de métricas utilizadas en Prometheus son las siguientes:

* Counter : es una métrica acumulativa que representa un solo contador cuyo valor puede solamente incrementar o reinicia a cero.
* Gauge : es una métrica que representa un solo valor numeral que puede arbitrariamente subir o bajar.
* Histogram : muestrea las observaciones y las cuenta en categorías configurables. También proporciona una suma de todo los valores observados.

Prometheus se utiliza para monitorizar aplicaciones de kubernete además de utilizarse en conjunto con Grafana para visualizar su monitorización.

## Grafana

Grafana es un software libre basado en licencia de Apache 2.0, que permite la visualización y el formato de datos métricos. Permite crear cuadros de mando y gráficos a partir de múltiples fuentes. Originalmente comenzó como un componente de Kibana y que luego le fue realizado una bifurcación.

Grafana es multiplataforma sin ninguna dependencia y también se puede implementar con Docker. Está escrito en lenguaje Go y tiene un HTTP API completo. Sus caracteristicas mas generales son las siguientes:

* Dispondremos de gráficos elegantes para la visualización de datos. Los gráficos son rápidos y flexibles, con numerosas opciones.
* Pone a nuestra disposición paneles dinámicos y reutilizables.
* Es altamente extensible, podemos utilizar muchos paneles y complementos disponibles en la biblioteca oficial.
* Pondrá a nuestra disposición la autenticación a través de LDAP, Google Auth, Grafana.com y Github.
* Respalda notablemente la colaboración al permitir el intercambio de datos y cuadros de mando entre equipos.
* Está disponible una [demostración](https://play.grafana.org/) en línea para que pruebes Grafana antes de instalarlo en tu equipo.

# Instalación de Grafana + Prometheus

Para ello lo primero que vamos a ver es que maquina va a ser la que tenga el visualizador **Grafana**, en mi caso va a ser la maquina **Dulcinea** que es la que tiene una interfaz de red hacia el exterior.

## Instalacion de Prometheus

La instalación de **Prometheus** es una tarea muy sencilla ya que solo debemos de instalarlo desde los repositorios, en este caso usando una maquina con Debian 10, para ello solo debemos instalar el paquete llamado **prometheus** que nos instalara tambien servicio de **Prometheus Node Exporter**:
```shell
#### Instalamos prometheus ####
debian@dulcinea:~$ sudo apt update && sudo apt install prometheus

#### Comprobamos la version ####
debian@dulcinea:~$ prometheus --version
prometheus, version 2.7.1+ds (branch: debian/sid, revision: 2.7.1+ds-3+b11)
  build user:       pkg-go-maintainers@lists.alioth.debian.org
  build date:       20190608-09:54:04
  go version:       go1.11.6
```

Ahora que tenemos **Prometheus** instalado, si nos vamos a nuestro navegador y ponemos la ip de nuestra maquina y el puerto **9100** veremos que nos llevara a un panel donde podremos hacer consultas sobre las metricas que esta recolectando el recolector:

![dashboard prometheus](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/sistemas/monitorizacion/prometheus-dulcinea.png)

## Instalacion de Grafana

A continuación debemos de instalar **Grafana** y podemos optar por dos caminos, el primero es instalarlo desde repositorio pero nos instalará una version mas antigua de Grafana de la que tenemos en su pagina oficial, o podemos usar la ultima version descargando el paquete manualmente. Yo optare por la segunda opción y lo instalaremos desde la ultima version que este en la [pagina oficial](https://grafana.com/grafana/download?plcmt=top-nav&cta=downloads):
```shell
#### Instalamos los paquetes necesarios ####
debian@dulcinea:~$ sudo apt install libfontconfig1

#### Descargamos el fichero .deb ####
debian@dulcinea:~$ wget https://dl.grafana.com/oss/release/grafana_7.3.7_amd64.deb

#### Instalamos el fichero .deb ####
debian@dulcinea:~$ sudo dpkg -i grafana_7.3.7_amd64.deb

#### Habilitamos el demonio de grafana y lo iniciamos ####
debian@dulcinea:~$ sudo systemctl enable grafana-server
debian@dulcinea:~$ sudo systemctl start grafana-server
```

Una vez lo tengamos listo solo debemos de irnos a nuestro navegador y entrar en la ip de nuestra maquina y al puerto 3000 que es donde escucha **Grafana** y nos aparecera el login que, por defecto es *admin/admin*:

![grafana dulcinea](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/sistemas/monitorizacion/grafana-dulcinea.png)

Una vez lo hayamos instalado debemos de añadir a nuestro **Grafana** un nuevo *datasource* en este caso va a ser **Prometheus** por lo que solo debemos de seguir los pasos que nos indica en las ventanas del *datasource*. 

Una vez lo tengamos vamos a instalar un *dashboard* en el que vamos a tener una gran cantidad de metricas pero tambien las mas generales, para ello debemos de irnos a la zona de importar un nuevo *dashboard* y elegimos la zona donde dice **Importar por id** e introducimos el id *1860*.

Cuando lo hagamos debemos de tener un *dashboard* como el siguiente:

![dashboard grafana](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/sistemas/monitorizacion/dashboard-grafana.png)

## Monitorización de maquinas

Ya tenemos las herramientas listas para monitorear nuestras maquinas en nuestro escenario y tambien nuestra maquina de ovh, por lo que debemos de indicarle a nuestra maquina con grafana que debe de recoger metricas desde las distintas maquinas que tenemos, primero vamos a hacerlo con nuestra maquina de ovh.

### OVH

Debemos de acceder a nuestra maquina de ovh y alli debemos de instalar el recolector de **Prometheus** el cual se encuentra en los repositorios de debian y podemos instalarlo de forma muy sencilla, el paquete que debemos de instalar se llama **prometheus-node-exporter**:
```shell
#### En nuestra maquina de ovh instalamos el paquete ####
debian@omega:~$ sudo apt install prometheus-node-exporter

#### Lo habilitamos e iniciamos ####
debian@omega:~$ sudo systemctl enable prometheus-node-exporter

debian@omega:~$ sudo systemctl start prometheus-node-exporter
```

Una vez tengamos todo esto hecho ya tendremos el recolector recogiendo las metricas de nuestra maquina, por lo que ahora debemos de irnos a la maquina donde tenemos **Prometheus** instalado y alli debemos de modificar el fichero llamado **/etc/prometheus/prometheus.yml** y en la seccion *scrape_configs:* y en *job_name: node* debemos de indicar la ip de la maquina y el puerto por el que va a escuchar:
```yml
- job_name: node
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['localhost:9100','146.59.196.94:9100']
```

Una vez hecho eso reiniciamos el servicio de **Prometheus** en nuestra maquina **Dulcinea** y nos dirigimos a nuestro *dashboard* en cual veremos una pestaña donde pone *host* y ahi elegimos la maquina de ovh para ver sus metricas:

![metricas ovh](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/sistemas/monitorizacion/metricas-ovh.png)

### Freston y Sancho

Para ello solo debemos de siguir los pasos que hemos seguido en la maquina de ovh ya que son la misma distribución por lo que solo debemos de instalar el recolector, habilitarlo, iniciarlo y modificar el fichero en la maquina **Dulcinea**
```shell
#### Instalamos el recolector en freston y sancho####
debian@freston:~$ sudo apt update && sudo apt install prometheus-node-exporter

ubuntu@sancho:~$ sudo apt update && sudo apt install prometheus-node-exporter

#### Lo habilitamos e iniciamos ####
debian@freston:~$ sudo systemctl enable prometheus-node-exporter

debian@freston:~$ sudo systemctl start prometheus-node-exporter

ubuntu@sancho:~$ sudo systemctl enable prometheus-node-exporter

ubuntu@sancho:~$ sudo systemctl start prometheus-node-exporter

```

Hecho esto solo debemos de modificar el fichero de **/etc/prometheus/prometheus.yml** y añadimos la nueva maquina, en este caso en vez de poner la ip, como tenemos el nombre de la maquina en nuestro DNS, solo debemos de poner el nombre de dicha maquina:
```yml
- job_name: node
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['localhost:9100','146.59.196.94:9100','freston:9100','sancho:9100']
```

Una vez hecho eso reiniciamos el servicio de **Prometheus** en nuestra maquina **Dulcinea** y nos dirigimos a nuestro *dashboard* en cual veremos una pestaña donde pone *host* y ahi elegimos la maquina **Freston** o la maquina **Sancho** para ver sus metricas:

![metricas freston](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/sistemas/monitorizacion/metricas-freston.png)

Y aqui podemos ver las metricas de **Sancho**:

![metricas sancho](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/sistemas/monitorizacion/metricas-sancho.png)

### Maquina Quijote

Nuestra maquina **Quijote** es una Centos8 por lo que la instalación del recolector varia un poco respecto a las anteriores instalaciones en las maquinas con Debian y Ubuntu ya que no estan los paquetes de **Prometheus** por lo que tenemos que instalarlos a mano aunque parezca un poco complicado al final no es tan dificil.

Lo primero que debemos de hacer es descargar el paquete de **Prometheus node exporter** el cual podemos encontrar en su repositorio de github, en mi caso la version a descargar es la version **1.0.1**. Una vez descargado debemos de seguir los siguientes pasos que se van a describir a continuación:
```shell
#### Añadimos el usuario node_exporter ####
[root@quijote ~]# useradd -M -r -s /bin/false node_exporter

[root@quijote ~]# id node_exporter
uid=992(node_exporter) gid=988(node_exporter) groups=988(node_exporter)


#### Descargamos el paquete en /tmp/ ####
[root@quijote tmp]# wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz

#### Lo descomprimimos y movemos el binario a /usr/local/bin/ ####
[root@quijote tmp]# tar xzf node_exporter-1.0.1.linux-amd64.tar.gz

[root@quijote tmp]# cp node_exporter-1.0.1.linux-amd64/node_exporter /usr/local/bin/

#### Modificamos los permisos del binario ####
[root@quijote ~]# chown node_exporter:node_exporter /usr/local/bin/node_exporter

#### Creamos una unidad de systemd ####
[root@quijote ~]# nano /etc/systemd/system/node_exporter.service

[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target

#### Reinciamos y habilitamos el nuevo servicio ####
[root@quijote ~]# systemctl daemon-reload

[root@quijote ~]# systemctl enable --now node_exporter.service

[root@quijote ~]# systemctl status node_exporter
● node_exporter.service - Prometheus Node Exporter
   Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; v>
   Active: active (running) since Mon 2021-02-01 19:02:48 CET; 27s ago

#### Comprobamos que este escuchando en el puerto 9100 ####
[root@quijote ~]# ss -altnp | grep 91
LISTEN    0         128                      *:9100                   *:*        users:(("node_exporter",pid=2169,fd=3))                          

#### Lo habilitamos en el firewall ####
[root@quijote ~]# firewall-cmd --add-rich-rule 'rule family="ipv4" port port=9100 protocol=tcp accept' --permanent

[root@quijote ~]# firewall-cmd --reload
```

Hecho esto solo nos queda poner en nuestro fichero de configuración de Dulcinea a nuestra maquina **Quijote**:
```yml
- job_name: node
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['localhost:9100','146.59.196.94:9100','freston:9100','sancho:9100','quijote:9100']
```

De esta forma tendremos en nuestro *dashboard* a nuestra maquina **Quijote**:

![metricas quijote](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/sistemas/monitorizacion/metricas-quijote.png)

