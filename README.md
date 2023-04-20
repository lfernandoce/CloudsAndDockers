#Sesión 01: Introducción a los contenedores en ECS: EC2 y fargate
Introducción
En ésta primer sesión de work pondremos en práctica los conceptos del funcionamiento de los contenedores y docker aprendidos durante el prework, mostraremos como construir una aplicación web dentro de un contenedor y desplegarlo usando los servicios de nube de AWS.

Objetivos
Los objetivos especificos para el work de esta sesión son:

Practicar con Docker
Comprender el funcionamiento de los contenedores en la nube
Descubrir el servicio ECS y sus componentes
Lanzar un web service completamente administrador desde una imagen almacenada en ECR
Requisitos
Para poder realizar las siguientes actividades es necesario lo siguiente:

Una computador con acceso a Internet
Una cuenta de usuario de AWS
Permisos de acceso para usar los servicios:
AWS Cloud9
AWS ECR
AWS ECS
Actividades
En esta sesión realizaremos las siguientes actividades:

Haciendo un Hello world desde un contenedor
Interactuamos con los contenedores en el shell
Exponer un contenedor para consumirlo en la red
Crear una imagen de contenedor desde un entorno AWS Cloud9
Crear un repositorio de imágenes de contenedor en AWS ECR
Publicar una imagen de contenedor en repositorio ECR
Desplegar una imagen de contenedor en Clúster AWS ECS
Hello from Docker
En esta actividad realizaremos ejercicios para aprender a usar el clásico hello wold dentro de un contenedor docker.

Usaremos un entorno de desarrollo en AWS Cloud9 para simplificar el uso de docker en un ambiente ya pre configurado.

Debemos ir al módulo AWS Cloud9, y hacemos clic en Crear Entorno, nombramos el entorno usando un distintivo especifico para tu proyecto.

Crear entorno cloud9 inicio

En el tipo de entorno usaremos New EC2 instance para usar una instancia EC2 del tipo e2-micro la cual esta dentro del free tier. Usaremos Amazon Linux 2 como la plataforma donde correrá el entorno de desarrollo cloud9.

Crear entorno cloud9 ec2

En las opciones de red usamos la opción AWS System Manager (SSM) para el tipo de conexión con el entorno, y las opciones predeterminadas para crear el ambiente en el VPC predeterminado y sin ninguna preferencia sobre la subred que usaremos.

Crear entorno cloud9 red

Para las etiquetas de este recurso se recomienda usar:

owner: email
environment: development
cost-center: bam
Esperamos a que se aprovisionen los recursos de Cloud9, hacemos clic en el nombre de nuestro entorno para ver los detalles.

Abrir entorno cloud9

Una vez que se ha creado el entorno cloud9, hacemos clic en Open in Cloud9 para abrir el URl con el entorno de desarrollo web.

Interfaz entorno cloud9

En Cloud9, vamos a la Terminal y ahi hacemos las siguientes validaciones para ver que ya tengamos docker configurado y listo para usarse.

Verificamos la ruta donde esta instalado docker con el comando which:

$ which docker
Usando el comando id vusuario sea miembro del grupo de sistema docker:

$ id
Verificamos que nos podamos conectar con el cliente docker al servidor o demonio Docker.

$ docker info
Ahora si, usamos el comando docker run para ejecutar una aplicación dentro de un contenedor,

$ docker run hello-world
Ahora usamos el comando docker ps para listar los contenedores:

$ docker ps -a
La opción -a lista todos los contenedores, los que ya terminaron su ejecución y los que están activos.

Finalmente, usamos el comando docker images paral listar las imágenes de los contenedores que hemos descargado:

$ docker images
Interactúa con un contenedor
Ahora vamos a ver una forma diferente de ejecutar un conteneedor en la cual pasamos un parámetro a su ejecución, por ejemplo:

$ docker run busybox echo "hello from busybox"
El comando anterior ejecuta un contenedor usando la imágen de busybox, el cual es un sistema Linux reducido que tiene comandos básicos del sistema, en este caso se le pasa el parámetro en donde se le dice que use el comando echo para que imprima un mensaje personalizado.

Ahora usamos el siguiente comando para listar los contenedores que hemos ejecutado:

$ docker ps -a
Ejecutamos un nuevo contenedor busybox pero con un mensaje diferente:

$ docker run busybox echo "Hola BEDU"
Veamos que pasa con los contenedores en el listado:

$ docker ps -a
Ejecutemos un contenedor con busybox y nos conectamos a su shell:

$ docker run -it busybox sh
La opción -it solicita una pseudoTTY para interacturar con el shell sh, el cual se pasa como parámetro a la ejecución del contenedor.

Dentro del shell del contenedor busybox hacemos un listado rápido y salimos:

$ ls
$ cd bin
$ ls
$ exit
Exposición de un contenedor
En esta actividad mostraremos como exponer un contenedor de manera que pueda ser consumido en la red.

Primero debemos hacer un listado de las imágenes locales para saber cuales hemos descargado:

$ docker images
Ahora descargamos la imagen de el servidor web nginx desde un repositorio o registry de imágenes:

$ docker pull nginx
Listamos nuevamente las imagenes:

$ docker images
Como podemos ver ya tenemos la imagen de nginx en local lista para usarse.

Para ejecutar un contenedor usando una imagen recien descargada usamos docker run, por ejemplo:

$ docker run --rm -it nginx
Describimos las opciones usadas en el comando de arriba:

--rm: especifica que se limpie el contenedor después de que el comando salga.
-it: especifica que se abra un pseudoTTY con stdin.
En Cloud9 crea una nueva terminal y listemos los contenedores para ver el estado de el servicio que acabamos de ejecutar:

$ docker ps -a
Como se puede ver, en este caso el contenedor de nginx esta con un estado de UP y expone un servicio en el puerto TCP/80.

Este contenedor se ejecuta en primer plano, para cancelar su ejecución, use Ctrl+C.

Ahora ejecutamos el contenedor en segundo plano con un mapeo de puertos dinámico y con un nombre de contenedor personalizado:

$ docker run -d -P --name miweb nginx
En este ejercicio se usa la opción -P la cual sirve para publicar todos los puertos expuestos hacia puertos aleatorios.

Listamos los contenedores para identificar el puerto al que se mapeo el servicio:

$ docker ps
Hacemos una solicitud web usando el cliente curl en el localhost y el puerto 49153:

$ curl http://localhost:49153
Ahora detenemos la ejecución del contenedor usando su nombre personalizado:

$ docker stop miweb
Ahora veremos como ejecutar un contenedor usando un mapeo de puertos estático:

$ docker run -p 8888:80 nginx
En este ejercicio se usa la opción -p para hacer un mapeo estático de los puertos de red del contenedor y en el que se expondrá en la red.

Creación de imagen de contenedor
Para los ejercicios de esta actividad vamos a usar el proyecto de docker-curriculum el cual incluye código que nos puede ayudar a realizar nuestras prácticas de docker de forma sencilla, este proyecto permite construir dos tipos de contenedores, uno para un sitio web estático y otro para una aplicación web con el framework de desarrollo web Flask.

Para los ejercicios de esta sesión construiremos la versión de la aplicación web Flask, así que lo primero que debemos hacer es, desde nuestro entorno de trabajo Cloud9 vamos a la terminal y clonamos el repositorio de github:

$ git clone https://github.com/prakhar1989/docker-curriculum.git
Entramos al directorio del proyecto de la aplicación flask:

$ cd docker-curriculum/flask-app
Los archivos relevantes para esta sesión son los siguientes:

app.py - Programa principal python.
requirements.txt - Definición de dependencias para aplicación python.
Dockerfile - Instrucciones para construir imagen de contendor docker
Mostramos el contenido del archivo Dockerfile para ver lo que vamos a construir:

$ cat Dockerfile
Antes de empezar a crear la imagen definamos su nombre en una variable de entorno del shell para poder reutilizarlo en los siguientes pasos:
