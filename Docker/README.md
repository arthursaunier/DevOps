# TP01

## Database

### build
>Cmd pour build dockerfile:  
`docker build -t postgrdb .`

### network
>Cmd pour créer network:  
`docker network create app-network`

>Cmd pour inspecter network (et voir les ip des conteneur connectés):  
`docker network inspect app-network`

### run
>Cmd pour run le docker PostgreSQL:  
`docker run --rm --network app-network -d -p 5432:5432 --name database postgrdb`

>Cmd pour run le docker PostgreSQL (avec variable d'environnement):  
`docker run --rm --network app-network -d -p 5432:5432 -e POSTGRES_PASSWORD="pwd" --name database postgrdb`

>Cmd pour run le docker adminer:  
`docker run --rm --network app-network -d -p 8080:8080 adminer`

>**Q1**:  
The `-e` flag allows us to pass environment variables directly to the docker in the docker build cmd (here the password for example). It's more secure than plain text files containing passwords.

### volume
>Cmd pour créer un volume:  
` docker volume create databaseData`

>### Version Finale  
>  
>Cmd pour run le docker PostgreSQL:  
`docker run --rm --network app-network -d -v $pwd/data:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD="pwd" --name database postgrdb`
>
>Cette partie de la commande permet d'associer le volume local databaseData avec les volume /var/lib/postgresql/data du docker:  
 `-v databaseData:/var/lib/postgresql/data`  

>**Q2**:  
L'utilisation d'un volume permet de stocker les donneés de notre DB même si on arrête le conteneur et qu'on le supprime. Le volume restera intègre et pourra être réutilisé si on recrée un autre conteneur.


#### dockerfile:
```dockerfile
#Based on postgres image
FROM postgres:11.6-alpine

#port
EXPOSE 5432

#copy sql scripts to init db
COPY ./script/ /docker-entrypoint-initdb.d

#Env variables
ENV POSTGRES_DB=db \
POSTGRES_USER=usr 

```

## API

### basics

- Install openjdk11  
- compile java: `javac Main.java`

>build command:
`docker build -t backendapi .`

>run command: 
`docker run --rm --network app-network --name api backendapi`

>Output: 
```
Hello World! 
```

#### dockerfile:
```dockerfile
#use openjdk11
FROM openjdk:11

#create a folder to put .class in
RUN mkdir /usr/src/myapp
COPY Main.class /usr/src/myapp

#specify work directory
WORKDIR /usr/src/myapp

#cmd to run when launching docker
CMD ["java", "Main"]

```

### Multistage build

>build command:
`docker build -t backendapi .`

>run command: 
`docker run --rm --network app-network -p 8080:8080 --name api backendapi`

#### dockerfile:
```dockerfile
# Build
FROM maven:3.6.3-jdk-11 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM openjdk:11-jre
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
ENTRYPOINT java -jar myapp.jar
```

**1-2**
>La première partie (**Build**) permet de build l'executable .jar.
On part de l'image maven, on spécifie un directory pour placer le .jar avec la var MYAPP_HOME et on y copie les données. On run mvn pour récupérer les packages.

>La deuxième partie (**Run**) permet de lancer le .jar. On se base sur openjdk11, on récupère le .jar qu'on a build avec toute les dependencies ds la première partie, et on la monte. On spécifie ensuite l'entrypoint de notre API avec la dernière ligne, qui sera notre myapp.jar.

### Backend API

On récupère les zip de simple-api-main en ligne.

On réutilise le même dockerfile que pour la partie précédente, en le placant dans le nouveau dossier.

Le .yml est mis a jour comme suivi pour que l'API se connecte à la DB.

#### application.yml:
```yml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          lob:
            non_contextual_creation: true
    generate-ddl: false
    open-in-view: true
  datasource:
    url: jdbc:postgresql://database:5432/db
    username: usr
    password: pwd
    driver-class-name: org.postgresql.Driver
management:
 server:
   add-application-context-header: false
 endpoints:
   web:
     exposure:
       include: health,info,env,metrics,beans,configprops
```

Les même commande pour build ou pour run le conteneur.

## HTTP Server

### Basics

pull image de apache httpd: 
>docker pull httpd:2.4.52-alpine

Build command:
>docker build -t apache-server .

Run command: 
>docker run -dit --network app-network --name http-server --rm -p 5000:80 apache-server

#### dockerfile:
```dockerfile
# our base image
FROM httpd:2.4.52-alpine

COPY ./templates/index.html /usr/local/apache2/htdocs/

EXPOSE 5000
```

### Configuration

Cmd pour dump la conf:
>docker cp CONTAINER:/usr/local/apache2/conf/httpd.conf .

On modifie le dockerfile

#### dockerfile:
```dockerfile
# our base image
FROM httpd:2.4.52-alpine

COPY ./templates/index.html /usr/local/apache2/htdocs/
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf

EXPOSE 5000
```

### Reverse Proxy

Dans le fichier httpd.conf, on ajoute:
```conf
ServerName localhost
<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://localhost:8080/
ProxyPassReverse / http://localhost:8080/
</VirtualHost>
```

et on charge bien ces 2 modules: 
```conf
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

Ensuite on build le conteneur puis on le lance de nouveau.

>**Q3**:  
L'utilisation du reverse proxy permet d'utiliser le serveur comme une gateway. Ici httpd va simplement générer la reponse http et delguera tout le traitement au backend distant. Le reverse proxy est utilisé pour sécuriser les applications, en séparant le backend et donc les données de ce qui est accessible par l'utilisateur. Cela peut aussi être utilisé pour du load balancing, de la haute disponibilité voir pour centraliser l'authentification sur un système.


## Link Application

### docker compose

#### docker-copose.yml
```yml
version: "3.7"

services:

  api:
    build: ./API_backend/simple-api-main/ #chemin pour le dockerfile
    restart: always
    networks:
      - app-network
    depends_on:
      - database
  
  database:
    build: ./DB
    restart: always
    networks:
      - app-network
    volumes:
      - /DB/data:/var/lib/postgresql/data/
  
  httpd:
    build: ./HTTP_Server/
    restart: always
    ports: #permet la visibilité du port 80 depuis l'exterieur du conteneur
      - "80:80"
    networks:
      - app-network
    depends_on: #permet demarrer ce service uniquement après le démarrage de database et api
      - database
      - api

networks:
  app-network:
```

>**Q**  
Un docker compose permet de lancer de multiple conteneur en une seule commande, en gérant leur dependance et leur ouverture de port vers l'exterieur.  
Il permet de lancer l'intégralité des composants d'une application en une seule fois.

**1.3**
 - build: permet de spécifier le chemin vers le dockerfile correspondant pour build le conteneur
 - network: permet de spécifier le network dans lequel le conteneur va être placé
 - ports: les ports ouvert en dehors du network
 - depends on: permet d'attendre le demarrage d'autre service avant de démarrer.


 ### Publish

 >**Q**
 Publish nos images permet de partager les configurations et l'intégralité du système que nous avons mis en place, et peut permettre de deployer l'application sur une autre machine distante.