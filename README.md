# Devops: TP1

## Partie Postgres

> Pourquoi avons-nous besoin d'un volume à attacher à notre conteneur postgres ?

Les volumes de Docker permettent de persister les données d'un conteneur même après son suppression ou sa mise à jour. En attachant un volume à un conteneur PostgreSQL, vous pouvez stocker les données de la base de données de manière permanente sur le système de fichiers hôte, plutôt que de les stocker uniquement dans le conteneur, qui peut être supprimé ou modifié. De cette manière, vous pouvez facilement sauvegarder et restaurer les données de la base de données.

**Commandes** 

```cmd
* docker pull postgres:14.1-alpine
* docker run --name devops-postgres -e POSTGRES_PASSWORD=pwd -e  POSTGRES_USER=usr -e POSTGRES_DB=db -d postgres:14.1-alpine
* docker network create app-network
* docker run --name postgres -d --net=app-network mquelennec/postgres
* docker run -p 8080:8080 --net=app-network -d --name=adminer adminer
* docker ps
* docker -v /my/own/datadir:/var/lib/postgresql/data
```
**Dockerfile**  
```Dockerfile
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd
```

Voir les screens dans le dossier `./postgres/screen`
