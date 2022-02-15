# TP02

## Setup Github Actions

### Build and test your application

Cmd pour runner des test java/maven (a lancer depuis le pom.xml directory)
>mvn clean verify

**2-1**
> les test containers sont des lib java qui permettent de runner des conteneur docker pour tester différent modules d'application.

###.main.yml
```yml
name: CI devops 2022 CPE
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: 
      - main
      - master
      - develop
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-18.04
    steps:
      #checkout your github code using actions/checkout@v2.3.3
      - uses: actions/checkout@v2.3.3

      #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with: 
          java-version: '11'
          distribution: 'adopt'
          
      
      #finally build your app with the latest command
      - name: Build and test with Maven
        run: mvn clean verify --file ./Docker/API_backend/simple-api-main/pom.xml
        
```

Attention au typo (qui bloque tout) et aux choix de branche (master!=main)

Le build avec maven:
```yml
- name: Build and test with Maven
        run: mvn clean verify --file ./Docker/API_backend/simple-api-main/pom.xml
```
bien spécifier le fichier target de maven avec --file

### First steps into the CD world

**secured variables, why ?**
> Pour eviter les fuites de données sensibles.

**Why did we put needs: build-and-test-backend on this job? Maybe try without this and you will see !**
> Pour ne pas build des images docker si il ya des erreurs.

**For what purpose do we need to push docker images?**
> Celanous permet d'avoir une version qui focntionne accessible en permanence. 

#### build et push docker
```yml
name: Build and push docker images
on:
  workflow_call:
    secrets:
      DOCKERHUB_USERNAME:
        required: true
      DOCKERHUB_TOKEN:
        required: true
jobs:        
  # define job to build and publish docker image
  build-and-push-backend:
    runs-on: ubuntu-latest
    # steps to perform in job
    env:
      working-directory: ./Docker/API_backend/simple-api-main
    steps:
      # login dockerhub
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Docker/API_backend/simple-api-main
          # Note: tags has to be all lower-case
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/devops-tp:simple-api
          # build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/master' }}

  build-and-push-database:
    runs-on: ubuntu-latest
    # steps to perform in job
    env:
      working-directory: ./Docker/DB
    steps:
      # login dockerhub
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Build image and push database
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Docker/DB
          # Note: tags has to be all lower-case
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/devops-tp:database
          # build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/master' }}

  build-and-push-httpd:
    runs-on: ubuntu-latest
    # steps to perform in job
    env:
      working-directory: ./Docker/HTTP_Server
    steps:
      # login dockerhub
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Checkout code
        uses: actions/checkout@v2
      - name: Build image and push httpd
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Docker/HTTP_Server
          # Note: tags has to be all lower-case
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/devops-tp:httpd
          # build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/master' }}

  build-and-push-front:
    runs-on: ubuntu-latest
    # steps to perform in job
    env:
      working-directory: ./Docker/devops-front-main
    steps:
      # login dockerhub
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Build image and push front
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Docker/devops-front-main
          # Note: tags has to be all lower-case
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/devops-tp:front
          # build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/master' }}
```

## Setup Quality Gate:

Backend build avec sonar pour vérification et split pipeline (avec le .yml précédent)

```yml
name: CI devops 2022 CPE
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: 
      - main
      - master
      - develop
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-18.04
    env:
      working-directory: ./Docker/API_backend/simple-api-main
    steps:
      #checkout your github code using actions/checkout@v2.3.3
      - uses: actions/checkout@v2.3.3

      #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with: 
          java-version: '11'
          distribution: 'adopt'
      
      #finally build your app with the latest command
      - name: Build and test with Maven
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=arthursaunier_DevOps --file ./Docker/API_backend/simple-api-main/pom.xml

  call-workflow:
    needs: test-backend
    uses: arthursaunier/DevOps/.github/workflows/main.yml@master 
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```