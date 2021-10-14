<img src="https://i.ibb.co/VM5MzBT/craftech-logo3.png=150x" width="250" height="250">

#### Prueba 2 - Despliegue de una aplicación Django y React.js

Elaborar el deployment dockerizado de una aplicación en django (backend) con frontend en React.js contenida en el repositorio. Es necesario desplegar todos los servicios en un solo docker-compose.

Se deben entregar los Dockerfiles pertinentes para elaborar el despliegue y justificar la forma en la que elabora el deployment (supervisor, scripts, docker-compose, kubernetes, etc)

Subir todo lo elaborado a un repositorio (github, gitlab, bitbucket, etc). En el repositorio se debe incluir el código de la aplicación  y un archivo README.md con instrucciones detalladas para compilar y desplegar la aplicación, tanto en una PC local como en la nube (AWS o GCP).
---------------------------------------------------------------
DESARROLLO:


En primer lugar se procede a construir los Dockerfiles tanto del frontend como del backend.
Empezando por la construcción del Dockerfile del backend, para ello, lógicamente, creamos el archivo Dockerfile dentro del directorio /backend incluido en el repositorio brindado. Dentro de él se usaron los siguientes parámetros: 
- FROM: indicando la imagen en la que se basa el contenedor, en este caso Python 3.7 (como buena practica siempre especificar la versión puntual de la imagen )
- ENV PYTHONUNBUFFERED: variable de entorno que permitirá que Docker nos muestre la salida y errores en la consola.
- WORKDIR: indica el directorio de trabajo se ubicara en /code, así, los comandos que ejecutemos se ejecutaran en el mismo.
- COPY requirements.txt:  el archivo "requirements.txt" será utilizado por el comando pip install .
- RUN: permite correr los comandos que le brindemos, además aquí se usa el archivo previamente copiado (requirements.txt) para instalar las dependencias especificadas en el mismo. 
- COPY . . : copia el directorio actual dentro del directorio actual del contenedor, en este caso /code.
- CMD: para ejecutar el comando a correr una vez construido el contenedor.
- EXPOSE: nos permite exponer el puerto indicado.

Luego se procede a crear el Dockerfile para el frontend. De manera análoga creamos el archivo, y los parámetros utilizados son, básicamente, los  mismos que se explicaron. En esta ocasión además de utilizar el node como base para React, utilizamos nginx para implementar multistage, técnica que nos permite implementar múltiples contenedores y que interactúen entre ellos.. 
Adicionalmente hay que configurar variables de entorno en el archivo .env para lo que corresponde al postgresql.

Una vez finalizada la creación de los Dockerfile pertinentes se debe proceder a construir los mismos. Para ello se debe ejecutar por separado y por consola, situándose en el directorio donde se encuentran ambos Dockerfile el siguiente comando:

$ docker build -t <nombreContenedor:etiqueta>
cabe destacar que el parametro -t (tag) indica la etiqueta. De otro modo se asignara un ID alfanumérico.
Para ejecutar el contenedor construido ejecutar:
$docker run -d -it <puertoExterno:puertoInterno> <nombreContenedor:etiqueta>
El parámetro -d (dettach) permite correr el contenedor en segundo plano, mientras que -it (interactive terminal) habilita la interacción con el contenedor.

Una vez construidos y en funcionamiento los contenedores se procede a la creación del archivo "docker-compose.yaml", que permitirá correr la aplicación completa. El mismo esta conformado por la siguiente estructura (para facilitar la explicación solo se desarrollara un ítem):

- services: 
    En esta sección se declaran los servicios que correremos dentro del Dockercompose. Nos permitirá correr varios contenedores usando un solo archivo y automáticamente los pone en la misma red.

    frontend:
        build:
            context: ./frontend
            dockerfile: ./Dockerfile
            Situamos el directorio de trabajo y especificamos el dockerfile.

        container_name: craftech-front
            Nombre de contenedor

        ports: 
            - target: 80
              published: 8081
                      Se escpecifican los puertos externos e internos.

    backend:
    ....
       depends_on:
            - db
        Indica que antes de construir el contenedor se debe instalar db para proseguir.

        networks: 
            webapp-net: 
            Nos permite crear una red de docker, porque necesitamos que los contenedores se ejecuten en la misma red, que se puedan ver de ellos y conectarse el uno con el otro

    db:
        volumes:
            - ./data/db:/var/lib/postgresql/data
        Permite que los datos que almacenemos persistan.

        environment:
            - POSTGRES_DB=postgres
            - POSTGRES_USER=postgres
            - POSTGRES_PASSWORD=postgres
            Variables de entorno para el postgres


Una vez creado el archivo pertinente se procede a la ejecución del mismo con:
$ docker-compose up --build
Con ello queda disponible la aplicación de manera local.

Se selecciono Amazon Web Service para desplegar la aplicación. Para ello se debe crear una instancia EC2,la cual se podrá ejecutar en la capa gratuita para evitar costos. De manera adicional se adjunta el documento (de creación propia) con los pasos a seguir para activar alarmas de consumo y evitar gastos de facturación. En cuanto a la configuración de reglas de entrada  de la instancia se permitió el trafico entrante en  los puertos TCP http en 80 y 8080, además del puerto ssh en el 22 para poder interactuar con la instancia a desplegar. Una vez finalizada la configuración, se procede a crear y lanzar la instancia (las demás configuraciones se dejaran por defecto para poder seguir utilizando la capa gratuita ofrecida por AWS). 
Para poder interactuar con la misma se utilizo el protocolo SSH. Para ello antes se debe generar una nueva llave de seguridad desde el asistente proporcionado por EC2 cuando se realiza la configuración, esta es la que permite el acceso a la instancia. Una vez obtenida la clave debemos configurar su propietario ejecutando: 
$chown 400 <claveObtenida>.pem
Ahora si podremos acceder vía SSH a la instancia ejecutando:
$ssh -i <claveObtenida>.pem ec2-user@<ipInstancia>
Una vez que iniciamos sesión debemos actualizar dependencias  e instalar y configurar docker y docker-compose :
$sudo yum update
$sudo yum install docker
$sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$sudo chmod +x /usr/local/bin/docker-compose
$sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
$ sudo usermod -a -G docker ec2-user

Para iniciar el servicio de docker en la instancia se usa:
$sudo service docker start
Y por ultimo para desplegar los contenedores, al igual que de manera local, se usa:
$sudo docker-compose up --build

Una vez realizados estos pasos podremos conectarnos desde cualquier host a nuestra aplicacion usando un navegador con:
<ipInstancia>:<puertoExterno>

Para la prueba es : ENVIAR POR CORREO

REFERENCIAS:

https://www.youtube.com/watch?v=E6jVbOlEVjU
https://hub.docker.com/_/postgres
https://docs.docker.com/samples/django/
https://www.youtube.com/watch?v=s8po-frUVwg     
https://github.com/blackadress/docker-django-postgres-nginx/tree/master/docker_django
https://blackhold.nusepas.com/2020/12/04/caso-real-de-despliegue-con-docker-compose/
https://platzi.com/blog/django-docker/
https://dev.to/gaabgonca/desplegando-contenedores-de-docker-en-aws-ec2-a-traves-de-un-boton-7nh