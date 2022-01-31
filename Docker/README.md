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
`docker run --rm --network app-network -d -v $pwd/data:/var/lib/postgre
sql/data -p 5432:5432 -e POSTGRES_PASSWORD="pwd" --name database postgrdb`
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

