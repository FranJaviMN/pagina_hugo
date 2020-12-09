---
title: "Mongodb"
date: 2020-12-09T13:46:40+01:00
draft: true
---

# Instalar mongodb en una maquina debian 10

En mi caso vamos a instalar debian 10 en una maquina virtual con vagrant, el codigo de vagrant que he usado es el siguiente:
```ruby
Vagrant.configure("2") do |config|
  config.vm.define :mongodb do |mongodb|
    mongodb.vm.box = "debian/buster64"
    mongodb.vm.hostname = "mongodb"
    mongodb.vm.network :public_network, :bridge=>"wlp2s0"
  end
end
```

Cuando tengamos la maquina lista lo que vamos a hacer es instalar unas herramientas antes de empezar:
```shell
#### Actualizamos las listas de repositorios ####
vagrant@mongodb:~$ sudo apt update

#### Instalamos las herramientas necesarias ####
vagrant@mongodb:~$ sudo apt install gnupg -y
```

## Descargar mongodb para debian 10

AHora que ya tenemos las herramientas anteriores lo que debemos de hacer es añadir un nuevo repositorio a nuestra maquina ya que los paquetes de mongodb no se encuentran en los repositorios de debian, por lo que ejecutamos los siguientes comandos:
```shell
#### Importamos la clave publica ####
vagrant@mongodb:~$ wget https://www.mongodb.org/static/pgp/server-4.4.asc -qO- | sudo apt-key add
OK

#### Añadimos el nuevo repositorio ####
vagrant@mongodb:~$ sudo nano /etc/apt/sources.list.d/mongodb-org.list

añadimos el siguiente repositorio:

deb http://repo.mongodb.org/apt/debian buster/mongodb-org/4.4 main
```

## Instalar mongodb para debian 10

Ya que hemos hecho los pasos de añadir el repositorio de mongodb para debian lo que debemos de hacer es actualizar la lista de paquetes e instalar los paquetes que vamos a necesitar para poder empezar a usar mongodb:
```shell
#### Actualizamos la lista de paquetes ####
vagrant@mongodb:~$ sudo apt update

#### Instalamos mongodb en nuestra maquina virtual ####
vagrant@mongodb:~$ sudo apt install mongodb-org -y

#### Iniciamos mongodb y lo habilitamos para que inicie solo siempre ####
vagrant@mongodb:~$ sudo systemctl enable --now mongod
Created symlink /etc/systemd/system/multi-user.target.wants/mongod.service → /lib/systemd/system/mongod.service.

#### Comprobamos que se ha inciciado correctamente ####
vagrant@mongodb:~$ sudo systemctl status mongod
● mongod.service - MongoDB Database Server
   Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2020-12-01 09:41:20 GMT; 22s ago
```

Ahora para comprobar si esta funcionando bien ejecutamos el comando de **mongo** para ver si funciona. Una vez entremos vamos a crear un nuevo usuario sobre la base de datos llamada **admin**.
```shell
#### Conectamos a nuestra base de datos admin ####
vagrant@mongodb:~$ mongo

#### Usamos la base de datos admin ####
> use admin
switched to db admin

#### Creamos el usuario cliente ####
db.createUser({user: "admin", pwd: "********", roles: [{role: "root", db: "admin"}]})
Successfully added user: {
	"user" : "admin",
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		}
	]
}
```

Ahora que ya hemos creado nuestro usuario administrador, salimos de la linea de comando de mongo y esta vez volvemos a entrar pero con el usuario que hemos creado anteriormente, para ello solo debemos de usar el siguiente comando:
```shell
#### Entramos en mongo con el usuario cliente ####
vagrant@mongodb:~$ mongo -u admin
MongoDB shell version v4.4.2
Enter password: 
> 
```

Hecho esto vamos a proceder a crear en nuestro mongodb una nueva coleccion, que es como se le llama a las bases de datos en mongo, y la vamos a poblar con algunos elementos de prueba. Para ello vamos a crear una coleccion llamada nba en la cual vamos a tener el siguiente fichero .json que le vamos a insertar a nuestra coleccion:
```shell
#### Descargamos el fichero .json ####
vagrant@mongodb:~$ wget https://gitlab.com/FranJaviMN/ejercicio_json/-/raw/master/playerdata.json

#### Entramos en mongo y creamos la coleccion junto a un nuevo usuario ####
> use nba
switched to db nba

> db.createUser({user: "cliente", pwd: "cliente", roles: [{role: "readWrite", db: "nba"}]})
Successfully added user: {
	"user" : "cliente",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "nba"
		}
	]
}

#### Inyectamos el fichero json a nuestra coleccion ####
vagrant@mongodb:~$ mongoimport --db nba --collection league --type json --file playerdata.json
```

## Acceso remoto a mongo

Ahora que ya tenemos un usuario y una coleccion creada vamos a entrar remotamente a nuestra base de datos mongodb, para ello debemos de ir al servidor de mongo y editar el fichero llamado **/etc/mongod.conf** editando el parametro *bindIp*:
```shell
#### Editamos el fichero mongod.conf ####
vagrant@mongodb:~$ sudo nano /etc/mongod.conf
...
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0
...

#### Reiniciamos el servicio de mongo ####
vagrant@mongodb:~$ sudo systemctl restart mongod.service
```

Ahora desde nuestro cliente vamos a conectar con el siguiente comando:
```shell
#### Desde la maquina postgres vamos a conectar a la maquina con mongodb ####
vagrant@postgres:~$ mongo --host 192.168.1.120 -u cliente
```

Como vemos hemos usado la maquina donde tenemos el servidor postgres para realizar la prueba y como vemos en la imagen sioguiente ha sido un exito, ya tenemos acceso remoto a la maquina con mongodb.

![cliente en mongodb](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/base%20de%20datos/mongo/cliente-mong.png)




