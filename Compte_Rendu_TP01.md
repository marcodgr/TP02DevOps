
# TP01

## Database

### Basics
- création d un docker file contenant
`FROM  postgres:11.6-alpine  
ENV  POSTGRES_DB=db \  
POSTGRES_USER=usr \  
POSTGRES_PASSWORD=pwd`


- build de l'image
- `docker build -t marcodgrc/database .`
- run de l'image :
`docker run  -p 5432:5432/ --network app-network --name database marcodgrc/database`

récuperation de l'image adminer
`docker pull adminer`

create network
`docker network create app-network`

run adminer
`docker run --network app-network -p 8080:8080 adminer`

ensuite on se connecte sur postgres avec l'interface adminer

### Init database

on modifie le fichier dockerfile et on ajoute la copie de fichier contenant les scripts SQL
`COPY CreateScheme.sql /docker-entrypoint-initdb.d/
COPY InsertData.sql /docker-entrypoint-initdb.d/`

- on doit ensuite re build et run le docker postgres et on se reconnecte dessus avec adminer


### Persist data 

pour éviter de perdre les données, on ajoute `-v /my/own/datadir:/var/lib/postgresql/data`

`docker run  -p 5432:5432/ --network app-network -v /my/own/datadir:/var/lib/postgresql/data --name database marcodgrc/database`

## Backend API
### Basics

on écrit le fichier java et on le compile avec javac
`javac Main.java`

- on écrit le docker file

FROM openjdk:11

COPY . .

RUN javac Main.java

CMD ["java","Main"] 

- On build le docker 
`docker build -t java .`

On run le docker
`docker run java`
Et nous avons le "hello World" qui s'affiche

### Multistage build
on génère un projet java avec spring initializer

on creer le fichier 	GreetingController et on copie / colle le code du TP à l'intérieur

on créer le dockerfile à la racine du projet java 
- on build le docker 
`docker build -t java .`
- on run 
`docker run -p 8080:8080 --name java-api java`

### Backend API

- on modifie le fichier .yml 
url: jdbc:postgresql://database:5432/db
username: usr
password: pwd

on ajoute le docker file au même endroit que le fichier src

on re build :
`docker build -t backend-api .`

et on run :
`docker run --name backend-api -p 8081:8080 --network app-network backend-api`


## Http Server

### Configuration

`docker cp http-container:/usr/local/apache2/conf/httpd.conf .`


docker build
`docker build . -t http`
run : 
`docker run --name http-container -p 80:80 --network app-network http`

et quand on va sur localhost ca fonctionne

### Docker-compose
on fait un fichier yml contenant :
```
version: '3.3'

services:

backend-api:

build:

./back-end-api/simple-api-main/simple-api/

networks:

- my-network

depends_on:

- database

database:

build:

./database

networks:

- my-network

httpd:

build:

./http

ports:

- 80:80

networks:

- my-network

depends_on:

- backend-api

networks:

my-network:
```
et on run le docker compose
`docker-compose up -d`


