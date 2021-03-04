---
title: "Kubernetes Letschat"
date: 2021-03-04T12:11:18+01:00
draft: true
---

# Despliegue de un cluster de kubernetes

En este ejercicio vamos a realizar el despliegue de una cluster de kubernetes en el que vamos a tener un total de 3 nodos, 1 nodo controlador al que le vamos a llamar **Master** y dos nodos *workers* que se van a llamar **Worker1** y **Worker2** respectivamente.

Para el despliegue de nuestro cluster vamos a usar la herramienta de **k3s**

## ¿Qué es K3S?

**K3S** es una solución Kubernetes creada por *Rancher Labs* que nos promete fácil instalación, pocos requisitos y un uso de memoría mínimo.

Para el planteamiento de un entorno Demo/Desarrollo esto se convierte en una gran mejora ya que nos elimina las complicaciones que puede llegar a tener el despliegue de escenarios donde vamos a trabajar con nuestras pruebas, ademas de la gran cantidad de recursos que esta puede llegar a consumir de nuestro equipo, siendo un inconveniente si tenemos un equipo de bajas prestaciones.

## Explicando el escenario

En nuestro caso y como hemos visto antes, vamos a usar tres nodos los cuales vamos a tener montados en unas maquinas virtuales en **openstack** con los nombres **Master**, **Worker1** y **Worker2** respectivamente.
* **Master**: Controla el cluster, tiene llamadas a la API...
* **Workers**: Son los encargados de manejar la carga de trabajo, donde los *pods* son desplegados y donde corren las aplicaciones que tengamos.

Nuestro objetivo final es el despliegue de una aplicacion llamada **lestchat** la cual hace uso de una base de datos **mongodb**. Los fichero de despliegue y servicio los podemos encontrar en el siguiente [repositorio](https://github.com/iesgn/kubernetes-storm/tree/master/unidad3/ejemplos-3.2/ejemplo8)

## Instalacion de Docker

Para ello lo primero que debemos de tener es las maquinas actualizadas, por lo que en cada una de ellas en conveniente usar el siguiente comando para procurar que esten correctamente actualizadas:
```shell
sudo apt update && sudo apt upgrade -y
```

Una vez tengamos los tres nodos ya actualizados vamos a proceder a la instalación de docker en cada uno de los nodos ya que kubernetes se usa sobretodo para la administración de contenedores Docker por lo que vamos a proceder a la instalación:
```shell
#### En cada una de las maquinas debemos de realizar los siguientes comandos para la instalacion de Docker ####
# Master #
debian@master:~$ sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg -y
# Worker1 y Worker2 #
debian@worker1:~$ sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg -y
debian@worker2:~$ sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg -y

#### Obtenemos la clave en los tres nodos ####
debian@master:~$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
debian@worker1:~$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
debian@worker2:~$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

#### Añadimos los repositorios necesarios ####
debian@master:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian buster stable"
debian@worker1:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian buster stable"
debian@worker2:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian buster stable"

#### Instalamos docker ####
debian@master:~$ sudo apt install docker-ce -y
debian@worker1:~$ sudo apt install docker-ce -y
debian@worker2:~$ sudo apt install docker-ce -y
```
## Instalacion de K3S en el nodo master

Una vez tengamos instalado Docker debemos de dirigirnos a nuestra maquina master que es donde vamos a tener el nodo master de nuestro escenario y ahi vamos a empezar a instalar **k3s** para ello debemos de seguir los siguientes pasos teniendo en cuanta que es solo en la maquina master:
```shell
#### Instalamos k3s ####
debian@master:~$ curl -sfL https://get.k3s.io | sh -s - --docker
[INFO]  Finding release for channel stable
[INFO]  Using v1.20.4+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.20.4+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.20.4+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, command exists in PATH at /usr/bin/ctr
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s

#### Comprobamos la nueva unidad de systemd que nos ha creado ####
debian@master:~$ sudo systemctl status k3s
● k3s.service - Lightweight Kubernetes
   Loaded: loaded (/etc/systemd/system/k3s.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2021-03-03 16:47:21 UTC; 1min 12s ago
     Docs: https://k3s.io

#### Comprobamos si el nodo maestro esta funcionando ####
debian@master:~$ sudo kubectl get nodes -o wide
NAME     STATUS   ROLES                  AGE     VERSION        INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                       KERNEL-VERSION          CONTAINER-RUNTIME
master   Ready    control-plane,master   8m34s   v1.20.4+k3s1   10.0.0.12     <none>        Debian GNU/Linux 10 (buster)   4.19.0-14-cloud-amd64   docker://20.10.5
```

Una vez hemos hecho las comprobaciones debemos de extraer el *token* del master que es el que debemos de poner mas adelante en los *worker*, para ello debemos de consultar el fichero **/var/lib/rancher/k3s/server/node-token** y nos mostrara algo como lo siguiente:
```shell
#### Obtenemos el token de nuestro master ####
debian@master:~$ sudo cat /var/lib/rancher/k3s/server/node-token
K105622af8f80bbae94421ada3fd7b6853db903e9f0988b27378248ab5fa19cab51::server:98d2c10ecb5234e5621bce6834f7da5c
```

## Instalando k3s en los worker

Ya tenemos en nuestro nodo **master** toda la cofiguración que necesitamos de momento asi que ahora vamos a proceder a instalar en las maquinas *worker* nuestro **k3s**, para ello debemos de hacer uso del toker que hemos obtenido de nuestra maquina **master** y tambien de la **ip** de nuestra maquina master, en mi caso es una ip interna de dirección **10.0.0.12**.

Teniendo en cuenta los datos anteriores, en cada uno de los *worker* debemos de instalar k3s de la siguiente manera:
```shell
#### Instalamos en cada uno de los worker k3s ####
# Worker1
debian@worker1:~$ curl -sfL http://get.k3s.io | K3S_URL=https://10.0.0.12:6443 K3S_TOKEN=K105622af8f80bbae94421ada3fd7b6853db903e9f0988b27378248ab5fa19cab51::server:98d2c10ecb5234e5621bce6834f7da5c sh -s - --docker

# Worker2
debian@worker2:~$ curl -sfL http://get.k3s.io | K3S_URL=https://10.0.0.12:6443 K3S_TOKEN=K105622af8f80bbae94421ada3fd7b6853db903e9f0988b27378248ab5fa19cab51::server:98d2c10ecb5234e5621bce6834f7da5c sh -s - --docker

#### Verificamos que estan funcionando correctamente
# Worker1
debian@worker1:~$ sudo systemctl status k3s-agent
● k3s-agent.service - Lightweight Kubernetes
   Loaded: loaded (/etc/systemd/system/k3s-agent.service; enabled; vendor
   Active: active (running) since Thu 2021-03-04 07:33:24 UTC; 4min 11s a
     Docs: https://k3s.io

# Worker2
debian@worker2:~$ sudo systemctl status k3s-agent
● k3s-agent.service - Lightweight Kubernetes
   Loaded: loaded (/etc/systemd/system/k3s-agent.service; enabled; vendor 
   Active: active (running) since Thu 2021-03-04 07:35:19 UTC; 2min 46s ag
     Docs: https://k3s.io
```

Una vez instalado vamos a irnos a nuestra maquina **master** donde vamos a verificar que los nodos los hemos añadidio correctamente a nuestro cluster, para ello usamos **kubectl get nodes**:
```shell
#### Comprobamos que los nodos estan añadidos correctamente a nuestro cluster ####
debian@master:~$ sudo kubectl get nodes -o wide
NAME      STATUS   ROLES                  AGE     VERSION        INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                       KERNEL-VERSION          CONTAINER-RUNTIME
worker2   Ready    <none>                 3m56s   v1.20.4+k3s1   10.0.0.11     <none>        Debian GNU/Linux 10 (buster)   4.19.0-14-cloud-amd64   docker://20.10.5
master    Ready    control-plane,master   14h     v1.20.4+k3s1   10.0.0.12     <none>        Debian GNU/Linux 10 (buster)   4.19.0-14-cloud-amd64   docker://20.10.5
worker1   Ready    <none>                 5m58s   v1.20.4+k3s1   10.0.0.8      <none>        Debian GNU/Linux 10 (buster)   4.19.0-14-cloud-amd64   docker://20.10.5
```

## Desplegando nuestra aplicacion lets-chat

Para hacer las pruebas en nuestro cluster vamos a desplegar en nuestro cluster la aplicación de **lets-chat** que debe de tener una base de datos **mongodb** por lo que en total vamos a tener dos fichero **deployment**, uno para la aplicación de **lets-chat** y otro para la base de datos **mongodb**:
```yml
#### Ficher de deployment de lets-chat ####
apiVersion: apps/v1
kind: Deployment
metadata:
  name: letschat
  labels:
    name: letschat
spec:
  replicas: 1 
  selector:
    matchLabels:
      name: letschat
  template:
    metadata:
      labels:
        name: letschat
    spec:
      containers:
      - name: letschat
        image: sdelements/lets-chat
        ports:
          - name: http-server
            containerPort: 8080

#### Fichero de deployment de mongodb ####
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  labels:
    name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      name: mongo
  template:
    metadata:
      labels:
        name: mongo
    spec:
      containers:
      - name: mongo
        image: mongo
        ports:
          - name: mongo
            containerPort: 27017
```

Como podemos ver en los ficheros de **deployment**, vemos que cada uno de ellos solo va a crear un pod que es dondde vamos a estar ejecutando nuestras aplicaciones y vemos como en el fichero de **lets-chat** tenemos expuesto el puerto **8080** y a este puerto le hemos llamado **http-server**

Una vez tengamos esos ficheros vamos a desplegarlos de la siguiente forma:
```shell
#### Desplegamos el fichero deployment de lets-chat ####
debian@master:~$ sudo kubectl apply -f letschat-deployment.yaml

#### Desplegamos el fichero deployment de mongodb ####
debian@master:~$ sudo kubectl apply -f mongodb-deployment.yaml

#### Comprobamos que se han desplegado correctamente ####
debian@master:~/kubernetes-storm/unidad3/ejemplos-3.2/ejemplo8$ sudo kubectl get all -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
pod/mongo-5c694c878b-b67g8      1/1     Running   0          5m26s   10.42.2.3    worker2   <none>           <none>
pod/letschat-7c66bd64f5-9z58n   1/1     Running   0          6m16s   10.42.0.13   master    <none>           <none>

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   15h   <none>

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                 SELECTOR
deployment.apps/mongo      1/1     1            1           5m26s   mongo        mongo                  name=mongo
deployment.apps/letschat   1/1     1            1           6m16s   letschat     sdelements/lets-chat   name=letschat

NAME                                  DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                 SELECTOR
replicaset.apps/mongo-5c694c878b      1         1         1       5m26s   mongo        mongo                  name=mongo,pod-template-hash=5c694c878b
replicaset.apps/letschat-7c66bd64f5   1         1         1       6m16s   letschat     sdelements/lets-chat   name=letschat,pod-template-hash=7c66bd64f5
```

Una vez tengamos nuestros fichero de **deployment** ya desplegados, podemos ver que el pod donde tenemos la aplicación de lets-chat esta reiniciando constantemente y eso se debe a que aun no puede acceder a la base de datos de mongodb por lo que debemos de tener dos ficheros **service** que son los que haran de proxy para que los pods de los distintos **deployment** puedan conectarse entre si:
```yml
#### Fichero lets-chat service ####
apiVersion: v1
kind: Service
metadata:
  name: letschat
spec:
  type: NodePort
  ports:
  - name: http
    port: 8080
    targetPort: http-server
  selector:
    name: letschat

#### Fichero mongodb service ####
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  ports:
  - name: mongo
    port: 27017
    targetPort: mongo
  selector:
    name: mongo
```

Y ahoda solo queda desplegar estos servicios:
```shell
#### Service de lets-chat ####
debian@master:~/kubernetes-storm/unidad3/ejemplos-3.2/ejemplo8$ sudo kubectl apply -f letschat-srv.yaml 
service/letschat created

#### Service de mongodb ####
debian@master:~/kubernetes-storm/unidad3/ejemplos-3.2/ejemplo8$ sudo kubectl apply -f mongo-srv.yaml 
service/mongo created

#### Comprobamos que ambos servicios estan activos ####
debian@master:~/kubernetes-storm/unidad3/ejemplos-3.2/ejemplo8$ sudo kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/mongo-5c694c878b-b67g8      1/1     Running   0          11m
pod/letschat-7c66bd64f5-9z58n   1/1     Running   5          12m

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP          15h
service/mongo        ClusterIP   10.43.67.197    <none>        27017/TCP        4m1s
service/letschat     NodePort    10.43.190.237   <none>        8080:31855/TCP   4s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongo      1/1     1            1           11m
deployment.apps/letschat   1/1     1            1           12m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/mongo-5c694c878b      1         1         1       11m
replicaset.apps/letschat-7c66bd64f5   1         1         1       12m
```

Asi, a la hora de hacer los despliegues de **deployment** y de **service** ya esta nuestro cluster sirviendo la aplicación de lets-chat por lo que debemos de irnos a nuestro navegador y en él debemos de poner la ip de nuestra maquina **master** junto con el puerto que nos ha asignado dentro del rango **30000-40000**, en mi caso me ha asignado el puerto **31855**:
```shell
#### Obtenemos el puerto que nos ha dado ####
debian@master:~$ sudo kubectl describe service/letschat | grep NodePort
Type:                     NodePort
NodePort:                 http  31855/TCP
```

![accediendo a lets-chat sin ingress](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/sistemas/kubernetes/lets-chat-no-ingress.png)

## Usando el fichero ingress.yml

Ahora que ya hemos accedido a nuestra aplicación por medio de la ip de nuestro master, ahora vamos a hacerlo pero esta vez con un proxy que nos va a dar la posibilidad de entrar en la aplicación usando un nombre de dominio como puede ser **www.lets-chat.com**.

Para ello debemos de usar un fichero que le llamaremos **ingress.yml** donde vamos a definir el nombre del host, en este caso va a ser **www.letschat.com**, el puerto en el que va a escuchar y el nombre del servicio:
```yml
#### Fichero ingress.yml ####
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-letschat
spec:
  rules:
  - host: www.letschat.com
    http:
      paths:
      - path: "/"
        backend:
          serviceName: letschat
          servicePort: 8080
```

Asi ahora solo debemos de cargarlo:
```shell
#### Usamos el fichero ingress.yml ####
debian@master:~/kubernetes-storm/unidad3/ejemplos-3.2/ejemplo8$ sudo kubectl apply -f ingress.yaml 
Warning: networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.networking.k8s.io/ingress-letschat created
```

Y ahora debemos de tener en nuestro fichero **/etc/hosts** una linea donde apunte a la ip y al nombre del host Hecho esto solo debemos de entrar en nuestro navegador y poner el nombre del host:

![accediendo a lets-chat con ingress](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/sistemas/kubernetes/lets-chat-ingress.png)

## Escalar varias replicas

Ahotra que ya tenemos nuestra aplicación corriendo podemos ampliar el numero de replicas según necesitemos, para ello vamos a ampliar el numero de replicas a 5 aunque pueden ser el numero que necesitemos, para ello vamos a usar el comando **kubectl scale**:
```shell
#### Escalamos a 5 replicas ####
debian@master:~/kubernetes-storm/unidad3/ejemplos-3.2/ejemplo8$ sudo kubectl scale deployment.apps/letschat --replicas=5
deployment.apps/letschat scaled

#### Comprobamos que se ha escalado correctamente ####
debian@master:~$ sudo kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/mongo-5c694c878b-b67g8      1/1     Running   0          3h13m
pod/letschat-7c66bd64f5-9z58n   1/1     Running   5          3h14m
pod/letschat-7c66bd64f5-bbhsm   1/1     Running   0          177m
pod/letschat-7c66bd64f5-psjz6   1/1     Running   0          177m
pod/letschat-7c66bd64f5-vgwqg   1/1     Running   0          177m
pod/letschat-7c66bd64f5-df4x9   1/1     Running   0          177m

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP          18h
service/mongo        ClusterIP   10.43.67.197    <none>        27017/TCP        3h5m
service/letschat     NodePort    10.43.190.237   <none>        8080:31855/TCP   3h1m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongo      1/1     1            1           3h13m
deployment.apps/letschat   5/5     5            5           3h14m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/mongo-5c694c878b      1         1         1       3h13m
replicaset.apps/letschat-7c66bd64f5   5         5         5       3h14m
```

## Usar otra maquina para manejar nuestro cluster

Para ello debemos de dirigirnos a nuestra maquina master y debemos de copiar el fichero **/etc/rancher/k3s/k3s.yaml** a la maquina donde queramos manejar el cluster con **kubectl**, para ello debemos de dirigirnos a la carpeta **.kube** y buscar un fichero llamado **config** y ahi debemos de pegar el contenido del fichero del master.