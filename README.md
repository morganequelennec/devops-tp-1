# DevOps: TP1

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

## Partie HTTP

**Commandes**

```cmd
 docker pull httpd:4.2 
```
```cmd
docker run -dit --name my-running-app -p 8081:80 my-apache2 cp  ./public-html/:/usr/local/apache2/conf/httpd.conf
docker exec -it my-running-app "/bin/bash"
```

> Pourquoi avons-nous besoin d'un proxy inverse ?

Un proxy inverse va nous permettre de faire transiter les requêtes d'un réseau privé vers l'Internet. Cela permet de masquer l'identité des utilisateurs du réseau privé, de protéger les systèmes internes contre les attaques directes depuis l'Internet et de contrôler l'accès à Internet pour les utilisateurs du réseau.

**Partie Java**

> Pourquoi avons-nous besoin d'une construction en plusieurs étapes ? Et expliquez chaque étape de ce dockerfile

```
La première déclaration FROM spécifie l'image de base pour l'environnement de construction. 
C'est une image Maven avec la version 3.8.6 et Amazon Corretto 17. L'environnement de construction est nommé "myapp-build".
La déclaration ENV définit la variable d'environnement MYAPP_HOME à "/opt/myapp".
La déclaration WORKDIR définit le répertoire de travail à MYAPP_HOME.
La déclaration COPY copie le fichier "pom.xml" et le répertoire "src" dans le répertoire de travail.
La déclaration RUN exécute la commande "mvn package" avec l'option "-DskipTests" pour construire l'application Java.
```
