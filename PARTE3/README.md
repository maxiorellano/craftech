
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
Primero deberemos situarnos en nuestro directorio y crear uno llamado .github/workflows y uno con extension.yaml, en este caso push.yaml: 
$mkdir -p .github/workflows
$touch .github/workflows/push.yaml 




on: push
    Definimos que cuando se realice un push se ejecute el trabajo que definimos (deploy())
name: deploy
  jobs:
    deploy:
      name: deploy to cluster
      runs-on: latest 
            Se especifica donde se ejecutara el trabajo
      steps:
        Se definen los pasos, las acciones a ejecutar
      - uses: actions/checkout@master
            Actions es le repositorio donde estan las acciones por defecto de github, checkout es el nombre de la accion y master es la version.
      - name: build and push to docker
            Aqui definimos otra accion adicional, la cual consiste en construir la imagen y subirla a dockerhub
        uses: docker/build-push-action@v1
        with: Con esto le pasamos las opciones especificas para la accion definida.
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
                Los dos secretos usados aqui deberemos configurarlos en github. Para ello deberemos agregar los secrets en la pestaña de configuracion del repositorio, tanto DOCKER_PASSWORD como DOCKER_USERNAME
          repository: ${{ github.repository}}
          tag_with_ref: true
          tag_with_sha: true
          tags: ${{ github.sha}}


checkout descarga el repositorio y comprueba si compila

https://www.youtube.com/watch?v=MNBf-ylhtK0&t=80s
https://github.com/docker/build-push-action#usage
https://docs.github.com/es/actions/deployment/deploying-with-github-actions
https://github.com/marketplace/actions/kubectl-simple