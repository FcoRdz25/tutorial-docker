
# Curso de Docker

### 1.- Iniciar, estatus y detener demonio de docker
```
$ systemctl start  docker
$ systemctl status docker
$ systemctl stop   docker
$ systemctl enable docker

### Otra opcion:
$ service docker start
$ service docker status
$ service docker stop
```

### 2.- Comandos Simples de Dockers:
```
$ docker ps
$ docker --version

//Muestra todos los comandos disponibles
$ docker
```

### 3.- Crear un contenedor
```
$ docker run hello-world
```

### 4.- Ver imagenes que tenemos en local
```
$ docker images

//Ver todos los contenedores parados
$ docker ps -a

//Mostrar el ultimo contenedor arrancado
$ docker ps -l

//Mostrar los ultumos 4
$ docker ps -n 4

//Ver el tamano
$ docker ps -a -s

```

### 4b.- Crear un contenedor interactivo
```
$ docker run -it ubuntu

//Dentro de Ubuntu
$ uname -a

//Salir de la imagen
$ exit
```

### 5.- Volver a entrar dentro del contenedor
```
//Primero vemos que contenedores hay
$ docker ps -a

//Podemos tomar los primeros 4 digitos de "Container_ID"
$ docker start -i cff9
```

### 6.- Crear/Corriendo un contenedor en Background
```
$ docker run -d nginx
```

### 7.- Docker tags y pulls
```
//Descargar la imagen con la etiqueta "trusty"
$ docker pull ubuntu:trusty

//Instalar y correr contenedor 
$ docker run -it ubuntu:trusty bash
```

> Recordando:
```
////Borrar contenedores
//usando el ID
$ docker rm ID_Contenedor
$ docker rm 7671

//Usando el nombre
$docker rm boring_curie

//Ver imagenes
$ docker images

//Borrar imagen
$ docker rmi ID_Imagen  
//No se puede borrar imagen si hay un contenedor usandolo

//Forza el borrado
$ docker rmi -f ID_Imagen  
```

### 8.- Docker exec: ejecutar comandos contra contenedores
```
//Dando nombre al contenedor
$ docker run -it --name mi_ubuntu ubuntu bash 

//Ejecutando un comando en el contenedor
$ docker exec -it mi_ubuntu echo hola  

//Entrando a contenedor 
$ docker exec -it mi_ubuntu bash  

//Descargar imagen sin crear contenedor
$ docker pull python 

//Ver las imagenes
$ docker images 

//Crea contenedor
$ docker run -it --name mi_python python 

//Entra en el contenedor en bash
$ docker exec -it mi_python bash

NOTAS:
//te muestra las opciones
$ docker image  

// te muestra las opciones
$ docker container  
```

### 9.- Docker Logs y Docker Kill
```
//Corre en background (no muestra nada)
$ docker run -d ubuntu sh -c "while true; do date; done" 

//Muestra todo lo impreso en pantalla
$ docker logs ID_contenedor

//Mostrar los ultimos 10
$ docker logs ID_contenedor --tail 10 

//Mostrar sin salirte
$ docker logs ID_contenedor -f 

//Para un contenedor
$ docker stop ID_contenedor 

//Matar contenedor
$ docker kill ID_contenedor 
```


### 10.- Docker Top, Docker Stats
```
//Top
$ docker top ID_contenedor 

//info de CPU% y Mem usada y limite
$ docker stats ID_contenedor  
```

### 11.- Docker inspect
```
//Muestra todo
$ docker inspect ID_contenedor

//La podemos mandar a un archivo para verla mas facil la info.
$ docker inspect ID_contenedor > informacion.txt 

//Tambien sirve para imagenes
$docer inspect ID_imagen
```

## REDES

### 12.- Gestionar Puertos para acceder al contenedor. Ejemplo
con NGINX
```
$ docker pull nginx
$ docker images

//IP de forma automatica/aleatoria
$ docker run -d -P nginx  //Mapea la ip del contenedor a la maquina principal
```
>Resultado el PORTS = 0.0.0.0:32768->80/tcp.
El "0.0.0.0" indica que se puede acceder desde
cualquier id de la maquina principal, 
Ahora se puede haceder asi: "localhost:32768"

### 13.- Nosotros damos el puerto
```
$ docker run -d --name nginx2 -p 8080:80 nginx
```

### 14.- Redes en Docker
```
$ docker network
$ docker network list

//Inspeccionar un red
$ docker network list
$ docker network inspect bridge > bridge.txt

///Instalar "ping"
$ apt-get install iputils-ping

// Correr un contenedor con mongo
docker run -d --name mongo3 -p 27018:27017 mongo

//Entrar a un contenedor mongo que esta corriendo
$ docker exec -it mongo3 bash

//Acceder a un contenedor con mongo desde otro
$ mongo --host 172.17.0.2 --port 27017
```

### 15.- Crear una nueva red
```
$ docker network ls
$ docker network create --help

///Crea una nueva red llamada red1
$ docker network create red1   
$ docker network ls

//Tools para ver info. de las redes
$ sudo apt install network-manager
$ nmcli con //Conexiones que tenemos dentro del OS
$ ip a 

/// Crear una red nueva con otra subnet
$ docker network create --subnet=192.168.0.0/16 red2

$ docker network create --subnet=172.30.0.0/16 --ip-range=172.30.0.0/24
```

### 16.- Asociar contenedores a una red 
```
$ docker run -it --name ubuntuA --network red1 ubuntu
$ docker run -d --name nginx5 --network red1 nginx

//Conectar un contenedor a una red diferente "en caliente"
$ docker netowork connect red2 ubuntuA

//Desconectar un contenedor de una red
$ docker network disconnect red2 ubuntuA
```

>Comandos Utiles:
```
//Contar la cantidad de contenedores
$ docker ps -a | wc -l

$ docker ps --help

//Muestra solos los id de todos los contenodores
$ docker ps -aq  

//Borra todos los contenedores
$ docker rm `docker ps -aq`  
```

### 17.- Enlazar contenedores con --link, con imagen busybox
```
$ docker run -it --rm --name b1 busybox 
// --rm Al salir borra el contenedor

//Ver mi ip
$ cat /etc/hosts

//Creando un nuevo enlace entre dos contenedores
$ docker run -it --rm --name b3 --link b1:maquina1 busybox

//Ya debe de apareser ahi mi ip y el del b1/maquina1
$ cat /etc/hosts  

```
> NOTA: b1 no puede hacer "ping b3", solo se puede usando su ip!!


### 18.- Enlazar contenedor Redes personalizadas con imagen Mysql

```
//"-e MYSQL_PASSWORD" es una variable de entorno, su valor sera "secret"
$ docker run -d --name mysql_server --network red1 -e MYSQL_PASSWORD=secret mysql

//En otra ventana
$ docer exec -it mysql_server bash
$$ mysql -u root -p 
>pass: secret

//salimos de lo anterior
$ exit

//Ahora si entramos como modo cliente
$ docker run -it --rm --name mysql_client --network red1 mysql bash

//Desde este cliente nos conectamos al mysql server
$ mysql -h mysql_server -u root -p
```

> RECORDATORIO: Al crear contenedores con "redes personalizadas" la crea su propio dns y todos estos contenedores se pueden ver entre si.


### 19.- Otro ejemplo de enlazar contenedores (WordPress y Mysql)
```
//Bajamos la version 5.7 de mysql
$ docker pull mysql:5.7

//Creamos contenedor de mysql
$ docker run -d --name mysql_wp --rm --network red1 -e MYSQL_ROOT_PASSWORD=secret mysql:5.7

//WorkPress
$ docker run -d --name wp --rm --network red1 -e WORDPRESS_DB_HOST=mysql_wp 
-e WORDPRESS_DB_PASSWORD=secret  -p 8080:80 wordpress
```


## Volumenes
Conceptos de volumenes
-Los volumenes son el mecanismo preferido para persistir la informacion 
y datos en contenedores


### 20.- Crear un volumen en un contenedor  ( -v )
```
/var/lib/docker          //Aqui guarda todo docker
/var/lib/docker/volumes  //aqui se guardan los volumenes

$ docker run -it -v /datos --name ubuntu1 ubuntu bash  //dentro de ubuntu estara la carpeta "datos"

//estamos dentro del contenedor
$ ls -l //Veremos que aparese la carpeta datos

//visualizar informacion de volumenes
$ docker volume ///muestra opciones
```

### 21.- Crear un directorio compartido con el host
```
//Creamos un directorio
$ mkdir dir1 
$ docker run -it -v ~/dir1:/dir1 --name ubuntu2 ubuntu
```

### 22.- Compartir volumenes entre contenedores
```
$ docker run -it -v /datos --name ubuntu4 ubuntu
$ docker run -it --name ubuntu5 --volumes-from ubuntu4 ubuntu bash
```
> NOTA: tiene que haber almenos un contenedor apuntando a un volumen
si no el volumen queda inutilisable


### 23.- Crear un volumen independiente
```
$ cd /var/lib/docker/volumes
$ docker volume create vol1
$ ls
$ docker run -it --name ubuntu7 -v vol1:/dir1 ubuntu bash

$ docker run -it --name ubuntu8 -v vol1:/datos:ro ubuntu bash //En solo lectura

//NOTA: Se puede cambiar el nombre de la carpeta dentro del contenedor pero sigue apuntando a la misma carpera del host
```

### 24.- Creando y usando un volumen con apache
```
$ docker volume create v1
$ docker run -d --name apache1 -p 80:80 -v v1:/usr/local/apache2/htdocs/ httpd

//Borrar un volumen
$ docker volume ls
$ docker volume prune //borra los volumenes sin uso
```


## CREAR Y GESTIONAR IMAGENES

### 25.- Modificamos un contenedor y creamos una nueva imagen
```
$ docker run -it --name ubuntu1 ubuntu bash
$ apt-get install wget
$ wget http://www.google.com
$ exit

//Vemos los cambios que sufrio el contenedor
$ docker diff ubuntu1  

//Docker commit. Crear una imagen manualmente
$ docker commit ubuntu1 mi_ubuntu  

//Crea una imagen llamada mi_ubuntu por defualt le pone la etiqueta "latest"

//Se le puede poner tambien un tag
$ docker commit ubuntu1 mi_ubuntu:1.0 

$ docker images
```


# Dockerfile
### Ejemplo Basico (hello World):
```
https://github.com/docker-library/hello-world/blob/b715c35271f1d18832480bde75fe17b93db26414/amd64/hello-world/Dockerfile
```

### 26.- Crear una imagen de un Dockerfile
```
$ mkdir imagen_python
$ cd imagen_python
$ vi Dockerfile	

FROM ubuntu
RUN apt-get update
RUN apt-get install -y python  //con -y no tendra nada interactivo
```

> NOTA: en un "Dockerfile" no puede haber nada interactivo!!

### Construyendo imagen y contenedor
```
//// -t "" nombre de la imagen, "." directorio actual (path, url , etc)
$ docker build -t imagen_python .  
$ docker images 

$ docker run -it imagen_python python
```

### Viendo Cambios en imagen
```
$ docker images
$ docker image 
$ docker image history imagen_python

$ vi Dockerfile
```

### 27.- Otro ejemplo y agregando Tag a la imagen
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
```
### Construyendo y corriendo 
```
$ docker build -t imagen_python:v1 .
$ docker images
$ docker run -it --rm imagen_python:v1 bash
```

## CMD
### 28.- Usando CMD
> NOTA: Dentro un Dockerfile solo puede haber un "CMD"

```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
CMD echo "Welcome to this container"
```
### Contruyendo
```
$ docker build -t image:v1 .
$ docker images
$ docker run -it --rm image:v1

$ docker rmi imagen:v1
```

### 28b.- Otro ejemplo de CMD
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
CMD ["echo","Welcome to this container"]        ///no usa la bash
```
### Construyendo
```
$ docker build -t image:v1 .
$ docker images
$ docker run -it --rm image:v1
$ docker image history image:v1
```

### 28c.- Otro ejemplo de CMD
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
CMD /bin/bash
```
### Construyendo
```
$ docker build -t image:v1 .
$ docker images
$ docker run -it --rm image:v1
$ docker image history image:v1
```

### 28d.- Otro ejemplo de CMD
> Nota: La forma correcta es usar: CMD ["comando"]
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
CMD ["/bin/bash"]
```

### Construyendo
```
$ docker build -t image:v1 .
$ docker images
$ docker run -it --rm image:v1
$ docker image history image:v1
```

## ENTRYPOINT

### 29.- ENTRYPOINT
> NOTA: Similar al CMD , pero con CMD se puede sustituir el comando de inicio
por ejemplo:

```
$ docker run -it --rm image:v1 ls
$ docker run -it --rm image:v1 df -h
```

### Usando ENTRYPOINT:
``` 
$vi Dockerfile
```
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
ENTRYPOINT ["/bin/bash"]
```
```
$ docker build -t image:v2 .
$ docker run -it --rm image:v2

$ docker run -it --rm image:v1 ls <<<< no va dejar ejecutar el "ls"
```

### 29b.- Otro ejemplo de ENTRYPOINT
```
$vi Dockerfile
```
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
ENTRYPOINT ["df"]
```
```
$ docker build -t image:v2 .
$ docker run -it --rm image:v2
//Ahora realiza el domando "df" y se sale del contenedor

$ docker run -it --rm image:v2 -h
//realiza el domando "df -h" y se sale del contenedor
```

## WORKDIR
### 30.- Ejemplo de Workdir
```
$vi Dockerfile
```
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
RUN mkdir /datos
WORKDIR /datos
RUN touch f1.txt
RUN mkdir /datos1
WORKDIR /datos1
RUN touch f2.txt
ENTRYPOINT ["/bin/bash"]
```

## COPY y ADD
### 30.- Copia los archivos del HOST a la imagen (usando COPY)
```
$vi Dockerfile
```
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
RUN mkdir /datos
WORKDIR /datos
RUN touch f1.txt
RUN mkdir /datos1
WORKDIR /datos1
RUN touch f2.txt

##COPY##
COPY index.html .
COPY app.log /datos

##ENTRYPOINT##
ENTRYPOINT ["/bin/bash"]
```
```
$ docker build -t image:v4 .
$ docker run -it --rm image:v4
```

### 31.- ADD

```
$vi Dockerfile
```
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
RUN mkdir /datos
WORKDIR /datos
RUN touch f1.txt
RUN mkdir /datos1
WORKDIR /datos1
RUN touch f2.txt

##COPY##
COPY index.html .
COPY app.log /datos

##ADD##
ADD docs docs
ADD f* /datos/
ADD f.tar .     <<< EN DOCKES se desenpaquetan

##ENTRYPOINT##
ENTRYPOINT ["/bin/bash"]
```

### Como empaquetar y desempaquetar un archivo?
```
//Empaquetar
$ tar cvf archivo.tar /archivo/mayo/*

//Desempaquetar
tar xvf archivo.tar
```

## ENV

### 32.- Variables de entorno
```
//Ejemplo 1
$ docker run -it --rm image:v4
$ env
>>Muestra la variables
$ echo $HOME

//Ejemplo 2
$ docker run -it --rm --env x=10 image:v4
$ env
$ echo $x

//Ejemplo 3
$ docker run -it --rm --env x=`pwd` image:v4
$ env
$ echo $x
>> /root/imagen_python
```

### 32b.- Ejemplo de ENV en Dockerfile
```
$vi Dockerfile
```
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
RUN mkdir /datos
WORKDIR /datos
RUN touch f1.txt
RUN mkdir /datos1
WORKDIR /datos1
RUN touch f2.txt

##COPY##
COPY index.html .
COPY app.log /datos

##ADD##
ADD docs docs
ADD f* /datos/
ADD f.tar .     <<< EN DOCKES se desenpaquetan

##ENV##
ENV dir=/data dir1=/data1
RUN mkdir $dir && mkdir $dir1

##ENTRYPOINT##
ENTRYPOINT ["/bin/bash"]
```


## ARG
### 33.- Usando ARG
```
$vi Dockerfile
```
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
RUN mkdir /datos
WORKDIR /datos
RUN touch f1.txt
RUN mkdir /datos1
WORKDIR /datos1
RUN touch f2.txt

##COPY##
COPY index.html .
COPY app.log /datos

##ADD##
ADD docs docs
ADD f* /datos/
ADD f.tar .     <<< EN DOCKES se desenpaquetan

##ENV##
ENV dir=/data dir1=/data1
RUN mkdir $dir && mkdir $dir1

##ARG##
ARG dir2
RUN mkdir $dir2

##ENTRYPOINT##
ENTRYPOINT ["/bin/bash"]
```
### Construyendo
```
$ docker build -t image:v6 .
>> Da un error.

//Forma correcta "pasando arg dir1"
$ docker build -t image:v6 --build-arg dir2=/data2  .
$ docker run -it --rm image:v6 
```


### 34.- Ejemplo usando scripts
```
$ vi add_user.sh
adduser $user_docker

//Lo convertimos a ejecutable
$ chmod +x add_user.sh

$vi Dockerfile
```
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
RUN mkdir /datos
WORKDIR /datos
RUN touch f1.txt
RUN mkdir /datos1
WORKDIR /datos1
RUN touch f2.txt

##COPY##
COPY index.html .
COPY app.log /datos

##ADD##
ADD docs docs
ADD f* /datos/
ADD f.tar .     <<< EN DOCKES se desenpaquetan

##ENV##
ENV dir=/data dir1=/data1
RUN mkdir $dir && mkdir $dir1

##ARG##
ARG dir2
RUN mkdir $dir2
ARG user
ENV user_docker $user
ADD add_user.sh /datos1
RUN /datos1/add_user.sh


##ENTRYPOINT##
ENTRYPOINT ["/bin/bash"]
```

```
$ docker build -t image:v6 --build-arg dir2="/data2" --build-arg user=spock .
$ docker run -it --rm image:v6 

$ cat /etc/passwd
$ ls /home/
```

## EXPOSE
### 35.- Exponiendo puertos
```
$vi Dockerfile
```
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python
RUN echo 1.0 >> /etc/version && apt-get install -y git \
    && apt-get install -y iputils-ping 
RUN mkdir /datos
WORKDIR /datos
RUN touch f1.txt
RUN mkdir /datos1
WORKDIR /datos1
RUN touch f2.txt

##COPY##
COPY index.html .
COPY app.log /datos

##ADD##
ADD docs docs
ADD f* /datos/
ADD f.tar .     <<< EN DOCKES se desenpaquetan

##ENV##
ENV dir=/data dir1=/data1
RUN mkdir $dir && mkdir $dir1

##ARG##
#ARG dir2
#RUN mkdir $dir2
#ARG user
#ENV user_docker $user
#ADD add_user.sh /datos1
#RUN /datos1/add_user.sh

##EXPOSE##
RUN apt-get install -y apache2
EXPOSE 80
ADD entrypoint.sh /datos1

##CMD##
CMD /datos1/entrypoint.sh

##ENTRYPOINT##
#ENTRYPOINT ["/bin/bash"]
```

```
$ vi entrypoint.sh
apachectl start
/bin/bash

$ chmod +x entrypoint.sh
$ ls -l
```
```
$ docker build -t image:v7 .
$ docker run -it --rm image:v7  << Forma incorrecta

$ docker run -it --rm -p 8080:80 image:v7  
```

## VOLUME en Dockerfile
### 36.- VOLUME en Dockerfile
```
$ vi Dockerfile
```
```
...
...
...

##VOLUME##
ADD paginas /var/www/html   <<< agrega y sustituye
VOLUME ["/var/www/html"]   <<< creamos un volumen basado en ese dir

##CMD##
CMD /datos1/entrypoint.sh

##ENTRYPOINT##
#ENTRYPOINT ["/bin/bash"]
```

```
$ docker volume ls ///El volumen se crea hasta que se crea el contenedor
$ docker run -it --rm -p 8080:80 --name c1 image:v8

///En otra ventana
$ docker volume ls
$ ls /var/lib/docker/volumes/numero/_data/

$ docker run -it --rm -p 9080:80 --volumes-from c1 --name c2 image:v8 
```

>RECORDAR ... si se salen todos los contenedores se pierde el volumen.


### Ejemplo Creando contenedor de mongodb
```
$ docker run -d --name mymongo  -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=pass mongo
```
