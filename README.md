# GitOps Project

## Sobre:

Um projeto pessoal para estudar as boas praticas do GitOps, CI/CD e toda a sua cultura, nesse projeto estarei realizando o build de uma imagem docker de um jogo chamado 2048, aonde o objetivo e realizar o build da imagem e realizar o deploy dela em um cluester local usando uma ferramenta chamada argoCD 


Uma versão em docker do projeto: `gabrielecirulli/2048(https://github.com/gabrielecirulli/2048)`


## como funciona a parte do CI no github Actions:

Basicamente eu tenho o game 2048 em uma pasta chamada 2048 e faço o build da imagem usando o ubuntu como base de SO faço o clone da msm pasta no diretorio /var/www/html e faço a instalação e o start do nginx e quem faz todo o trampo e o git

`Imagem docker:`

```dockerfile
# Base image
FROM ubuntu:latest

# Commands to install nginx
RUN apt-get update && \
    apt-get install -y -q nginx

# Copy the diretory 2048 to /var/www/html
COPY 2048 /var/www/html

# expose port 80
EXPOSE 80

# some commands to active the nginx
CMD ["nginx", "-g", "daemon off;"]

```

`Workflow do action:`

``` yaml
name: CI

on:
  release:
    types: [created]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v4
      -
        name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      -
        name: get release tag
        run: echo "RELEASE_VERSION=${GITHUB_REF#refs/*/}" >> $GITHUB_ENV
      -
        name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/marcus-2048:${{ env.RELEASE_VERSION }}


```
## Agora vamos para o CD da historia

### Configurações de ambientes

> [!IMPORTANT]  
> nesse ambiente fiz tudo em uma vm ubuntu 22.01 LTS, utilizando o minikube como cluster local de node unico.


``` conf
# Config da vm:

 - 4GB Ram
 - 20GB Disk
 - 6 CPU

```



## Instalando o ArgoCd

### Instalando o ArgoCD como um operador no Kubernetes
Para instalar o ArgoCD como um operador no Kubernetes, antes precisamos criar uma namespace chamada argocd, e para isso basta executar o seguinte comando:

``` 
kubectl create namespace argocd
```

A saída desse comando será algo parecido com isso:

```
namespace/argocd created
```

Agora vamos instalar o ArgoCD como um operador no Kubernetes:

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```