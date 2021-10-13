<img src="https://i.ibb.co/VM5MzBT/craftech-logo3.png=150x" width="250" height="250">

#### Prueba 1 - Diagrama de Red

Produzca un diagrama de red (puede utilizar lucidchart) de una aplicación web en GCP o AWS y escriba una descripción de texto de 1/2 a 1 página de sus elecciones y arquitectura.

El diseño debe soportar:

- Cargas variables
- Contar con HA (alta disponibilidad)
- Frontend en Js
- Backend con una base de datos relacional y una no relacional
- La aplicación backend consume 2 microservicios externos

 
El diagrama debe hacer un mejor uso de las soluciones distribuidas.
---------------------------------------------------------------
Se selecciono AWS para la prueba. El diseño presentado permite las cargas variables gracias a Elastic Load Balancing(https://aws.amazon.com/es/elasticloadbalancing/?whats-new-cards-elb.sort-by=item.additionalFields.postDateTime&whats-new-cards-elb.sort-order=desc), ya que segun su especificacion "... proporciona balanceo de carga básico en varias instancias de Amazon EC2 y funciona tanto en el nivel de solicitud como en el nivel de conexión. El balanceador de carga clásico está diseñado para aplicaciones que se crearon dentro de la red EC2-Classic.", en otras palabras sera el encargado de redireccionar el trafico segun la solicitud; pues sera necesario utilizar uno debido a que la especificacion requeria alta disponibilidad, la cual se lograra desplegando instancias EC2 (las cuales alojaran tanto  frontend como backend ) en distintas zonas de disponibilidad (Availiability Zones) para tratar de garantizar la disponibilidad en caso de inconvenientes . REVISAR!!!!. Por otro lado el encargado de garantizar y gestionar alta disponibilidad es el Auto Scaling, mediante el agregado de instancias EC2 adicionales segun se requiera. En cuanto a la base de datos no relacional sera implementada mediante Amazon DynamoDB, y para la relacional Amazon RDS, las cuales cumplen con las necesidades a satisfacer.


---------------------------------------------------------------
#### Prueba 2 - Despliegue de una aplicación Django y React.js

Elaborar el deployment dockerizado de una aplicación en django (backend) con frontend en React.js contenida en el repositorio. Es necesario desplegar todos los servicios en un solo docker-compose.

Se deben entregar los Dockerfiles pertinentes para elaborar el despliegue y justificar la forma en la que elabora el deployment (supervisor, scripts, docker-compose, kubernetes, etc)

Subir todo lo elaborado a un repositorio (github, gitlab, bitbucket, etc). En el repositorio se debe incluir el código de la aplicación  y un archivo README.md con instrucciones detalladas para compilar y desplegar la aplicación, tanto en una PC local como en la nube (AWS o GCP).
---------------------------------------------------------------

En primer lugar se procede a construir los Dockerfiles tanto del frontend como del backend.

Primero se construyo el Dockerfile del backend, para ello, logicamente, creamos el archivo Dockerfile dentro del directorio /backend incluido en el repositorio brindado. Dentro del mismo se uso los siguientes parametros: 
- FROM: indicando la imagen en la que se basa el contenedor, en este caso Python 3.7 (como buena practica siempre especificar la version puntual de la imagen )
- ENV PYTHONUNBUFFERED: variable de entorno que permitira que Docker nos muestre la salida y errores en la consola.
- WORKDIR: indica el directorio de trabajo se ubicara en /code, asi, los comandos que ejecutemos se haran en el mismo.
- COPY requirements.txt:  el archivo "requirements.txt" sera utilizado por el comando pip install.
- RUN: permite correr los comandos que le brindemos, ademas aqui se usa el archivo previamente copiado (requirements.txt) para instalar las dependencias especificadas en el mismo. 
- COPY . . : copia el directorio actual dentro del directorio actual del contenedor, en este caso /code.
- CMD: para ejecutar el comando a correr una vez construido el contenedor.
- EXPOSE: nos permite exponer el puerto indicado.

Luego se procede a crear el Dockerfile para el frontend. De manera analoga creamos el archivo, y los parametros utilizados son, basicamente, los  mismos que se explicaron. En esta ocasion ademas de utilizar el node como base para React, utilizamos nginx para implementar multistage, tecnica que nos permite implementar multiples contenoderes y que interactuen entre ellos.. 
Adicionalmente hay que configurar variables de en el archivo .env para lo que corresponde al postgresql.

Una vez finalizada la creacion de los Dockerfile pertinentes se debe proceder a construir los mismos. Para ello se debe ejecutar por separado y por consola, situandose en el directorio donde se encuentran ambos Dockerfile el siguiente comando:

$ docker build -t <nombreContenedor:etiqueta>
cabe destacar que el parametro -t (tag) indica la etiqueta. De otro modo se asignara un ID alfanumerico.
Para ejecutar el contenedor construido ejecutar:
$docker run -d -it <puertoExterno:puertoInterno> <nombreContenedor:etiqueta>
El parametro -d (dettach) permite correr el contenedor en segundo plano, mientras que -it (interactive terminal) habilita la interaccion con el contenedor.

Una vez construidos y en funcionamiento los contenedores se procede a la creacion del archivo "docker-compose.yaml", que permitira correr la aplicacion completa. El mismo esta conformado por la siguiente estructura (para facilitar la explicacion solo se desarrollara un item):

- services: 
    En esta seccion se declaran los servicios que correremos dentro del Dockercompose. Nos permitira correr varios contenedores usando un solo archivo y automaticamente los pone en la misma red.

    frontend:
        build:
            context: ./frontend
            dockerfile: ./Dockerfile
            Situamos el directorio de trabajo y especificamos el dockerfile

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



Una vez creado el archivo pertinente se procede a la ejecucion del mismo con:
$ docker-compose up --build
Con ello queda disponible la aplicacion de manera local.

Se selecciono Amazon Web Service para desplegar la aplicacion. Para ello se debe crear una instancia EC2 de tipo  TIPOOOOO ,la cual se podra ejecutar en la capa gratuita para evitar costos. De manera adicional se adjunta el documento "NOMBRE DE DOCUMENTO" (de creacion propia) con los pasos a seguir para activar alarmas de consumo y evitar gastos de facturacion. En cuanto a la configuracion de reglas de entrada  de la instancia se permitio el trafico entrante en  los puertos TCP http en 80 y 8080, ademas del puerto ssh en el 22 para poder interactuar con la instancia a desplegar. Una vez finalizada la configuracion, se procede a crear y lanzar la instancia (las demas configuraciones se dejeran por defecto para poder seguir utilizando la capa gratuita ofrecida por AWS). 
Para poder interactuar con la misma se utilizo el protocolo SSH. Para ello antes se debe generar una nueva llave de seguridad desde el asistente proporcionado por EC2 cuando se realiza la configuracion, esta es la que permite el acceso a la instancia. Una vez obtenida la clave debemos configurar su propietario ejecutando: 
$chown 400 <claveObtenida>.pem
Ahora si podremos acceder via SSH a la instancia ejecutando:
$ssh -i <claveObtenida>.pem ec2-user@<ipInstancia>
Una vez que iniciamos sesion debemos actualizar dependencias  e instalar y configurar docker y docker-compose :
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
Para la prueba es : 

LINKS UTILIZADOS:
https://www.youtube.com/watch?v=E6jVbOlEVjU
https://hub.docker.com/_/postgres
https://docs.docker.com/samples/django/
https://www.youtube.com/watch?v=s8po-frUVwg		
https://github.com/blackadress/docker-django-postgres-nginx/tree/master/docker_django
https://blackhold.nusepas.com/2020/12/04/caso-real-de-despliegue-con-docker-compose/
https://platzi.com/blog/django-docker/
https://dev.to/gaabgonca/desplegando-contenedores-de-docker-en-aws-ec2-a-traves-de-un-boton-7nh




#### Prueba 3 - CI/CD

Dockerizar un nginx con el index.html default.
Elaborar un pipeline que ante cada cambio realizado sobre el index.html buildee la nueva imagen y la actualize en la plataforma elegida. (docker-compose, swarm, kuberenetes, etc.)
Para la creacion del CI/CD se puede utilizar cualquier plataforma (CircleCI, Gitlab, Github, Bitbucket.)

**Requisitos y deseables:**

La solución al ejercicio debe mostrarnos que usted puede:

Automatizar la parte del proceso de despliegue.
usar conceptos de CI para aprovisionar el software necesario para que los entregables se ejecuten
use cualquier herramienta de CI de su elección para implementar el entregable


