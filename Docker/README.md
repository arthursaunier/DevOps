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
`docker run --rm --network app-network -d -v databaseData:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD="pwd" --name database postgrdb`  
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
COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d
COPY 02-InsertData.sql /docker-entrypoint-initdb.d

#Env variables
ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd
```

