<img src="https://i.ibb.co/VM5MzBT/craftech-logo3.png=150x" width="250" height="250">

#### Prueba 3 - CI/CD

Dockerizar un nginx con el index.html default.
Elaborar un pipeline que ante cada cambio realizado sobre el index.html buildee la nueva imagen y la actualize en la plataforma elegida. (docker-compose, swarm, kuberenetes, etc.)
Para la creacion del CI/CD se puede utilizar cualquier plataforma (CircleCI, Gitlab, Github, Bitbucket.)

**Requisitos y deseables:**

La solución al ejercicio debe mostrarnos que usted puede: 

Automatizar la parte del proceso de despliegue.
usar conceptos de CI para aprovisionar el software necesario para que los entregables se ejecuten
use cualquier herramienta de CI de su elección para implementar el entregable
-------------------------------------------------
DESARROLLO: 

Para el desarrollo del pipeline se selecciono como herramienta Github Actions, y para desplegar la pagina con el index se usa Github Pages. 
En primer lugar deberemos situarnos en nuestro directorio (PARTE3),  crear uno llamado .github/workflows y uno con extension.yaml, en este caso push.yaml: 
$mkdir -p .github/workflows
$touch .github/workflows/push.yaml 

El archivo push.yaml esta compuesto de la siguiente manera- las acciones aquí usadas se descargaron del market de github actions (cabe aclarar también que se explican los ítems solo una vez para evitar redundancia):

name: ci
on:
  push:
            Definimos que cuando se realice un push se ejecute el trabajo que definimos (ci()).
    branches:
      - 'main'
jobs:
  docker:
    runs-on: ubuntu-latest
            Se especifica donde se ejecutará el trabajo.
    steps:
            Se definen los pasos, las acciones a ejecutar.
      -
        name: Checkout
        uses: actions/checkout@v2
            Actions es le repositorio donde están las acciones por defecto de github, checkout es el nombre de la acción y checkout es la versión.            
      -
        name: Set up QEMU
        uses: docker/setup-qemu-action@v1
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      -
        name: Login to DockerHub
        uses: docker/login-action@v1
        with:
            Con este parámetro le pasamos las opciones especificas para la acción definida.
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
                Los dos secretos usados aqui deberemos configurarlos en github. Para ello deberemos agregar los secrets en la pestaña de configuración (settings) del repositorio, tanto DOCKER_PASSWORD como DOCKER_USERNAME.
      -
        name: Build and push
            Aquí definimos otra acción adicional, la cual consiste en construir la imagen y subirla a dockerhub.
        uses: docker/build-push-action@v2
        with:
          context: ./PARTE3/
          push: true
          tags: maxiorellano/craftech:latest, maxiorellano/craftech:${{ github.sha }}

Una vez construido el archivo yaml, como así también el index.html básico, lo unico que deberemos hacer es realizar un push en nuestro repositorio ($git push origin main). En consecuencia solo revisamos la solapa "Actions" en github en el navegador, donde podremos observar que se esta ejecutando los trabajos, y finalizamos.
Por otro lado se uso Github Pages para hacer que nuestro index este disponible, para que luego de subidos los cambios al repositorio se encuentren disponibles. Para este caso la url para acceder al index es : https://maxiorellano.github.io/craftech/PARTE3/index.html

REFERENCIAS:
https://www.youtube.com/watch?v=MNBf-ylhtK0&t=80s
https://github.com/docker/build-push-action#usage
https://docs.github.com/es/actions/deployment/deploying-with-github-actions
https://github.com/marketplace/actions/kubectl-simple
https://github.com/pablokbs/prueba-gha/blob/master/.github/workflows/cicd.yml
