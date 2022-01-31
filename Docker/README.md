# TP01

## Database

### Basics

Cmd pour build dockerfile:  
`docker build -t postgrdb .`

Cmd pour créer network:  
`docker network create app-network`

Cmd pour run le docker PostgreSQL:  
`docker run --rm --network app-network -d -p 5432:5432 --name database postgrdb`

Cmd pour run le docker PostgreSQL (avec variable d'environnement):  
`docker run --rm --network app-network -d -p 5432:5432 -e POSTGRES_USER="usr" POSTGRES_PASSWORD="pwd" --name database postgrdb`

Cmd pour run le docker adminer:  
`docker run --rm --network app-network -d -p 8080:8080 adminer`

Q1:  
The `-e` flag allows us to pass environment variables directly to the docker in the docker build cmd. It's more secure than plain text files containing passwords.


#### dockerfile
```dockerfile
#Based on postgres image
FROM postgres:11.6-alpine

#port
EXPOSE 5432

#Env variables
ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd
```

