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