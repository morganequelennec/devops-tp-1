# DEVOPS - TP1

## Les bases de Docker

Build de l'image

```docker
docker build -t postgres .
```

Création d'un network pour nos ressources

```docker
docker network create app-network
```

Run + exécution d'un conteneur

```docker
docker run --rm \
    --name postgres \
    --env-file .env \
     -p 5432:5432 \
    --network=app-network \ 
    postgres
```

## Partie Postgres

> Pourquoi avons-nous besoin d'un volume à attacher à notre conteneur postgres ?

```docker -v /my/own/datadir:/var/lib/postgresql/data```

Les volumes de Docker permettent de faire persister les données d'un conteneur même après son suppression ou sa mise à jour. En attachant un volume à un conteneur PostgreSQL, on peut stocker les données de la base de données de manière permanente sur le système de fichiers hôte, plutôt que de les stocker uniquement dans le conteneur, qui pourrait être supprimé ou modifié. Ainsi, les données sont sauvegardées.

### Commandes

**Build de l'image**

```docker
docker build -t backend .
```

**Run du conteneur**
```
docker run --network=app-network -p 8080:8080 --name backend backend
```

**Attacher un volume au conteneur**

```docker
docker -v /my/own/datadir:/var/lib/postgresql/data
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

### Commandes

**Build de l'image**

```docker
docker build -t app-apache .
```

**Run du conteneur**
```
docker run -p 80:80 --name app-apache --net=app-network app-apache
```

**Dockerfile**  
```Dockerfile
FROM httpd:2.4 AS web-server
COPY ./public-html/ /usr/local/apache2/htdocs/
COPY httpd.conf /usr/local/apache2/conf/httpd.conf
```

> Pourquoi avons-nous besoin d'un Reverse-Proxy ?

Un proxy inverse va nous permettre de faire transiter les requêtes d'un réseau privé vers Internet. Cela permet de masquer l'identité des utilisateurs du réseau privé, de protéger les systèmes internes contre les attaques directes depuis Internet et de contrôler l'accès à Internet pour les utilisateurs du réseau.
Pour le Reverse-proxy, nous exécutons les mêmes commandes que les étapes précédents: Build et run de l'image du proxy httpd
On récupère ensuite la conf: ```docker exec -it your_running_container cat /usr/local/apache2/conf/httpd.conf > httpd.conf```

**Partie Java**

> Pourquoi avons-nous besoin d'une construction en plusieurs étapes ? Et expliquez chaque étape de ce dockerfile

L'utilisation de la construction en plusieurs étapes dans un fichier Dockerfile est un moyen efficace d'optimiser la taille de l'image finale et d'améliorer les performances du processus de construction.

Dans notre cas, la première étape consiste à construire le package Java en utilisant Maven. Le résultat de cette étape est un fichier jar. Ce fichier jar sera copié dans une image de base plus légère, comme Amazon Corretto, pour exécuter l'application. Cela signifie que la première étape de la construction peut inclure toutes les dépendances nécessaires pour la construction, telles que Maven, tandis que la seconde étape n'a besoin que des fichiers nécessaires à l'exécution de l'application.

En utilisant ce processus de construction en plusieurs étapes, vous pouvez obtenir une image plus petite, qui se télécharge et s'exécute plus rapidement, sans compromettre les ressources nécessaires à la construction.

```docker
# Construction de l'image
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
# Définition de la variable d'environnement MYAPP_HOME
ENV MYAPP_HOME /opt/myapp
# Définition du répertoire de travail
WORKDIR $MYAPP_HOME
# Copie du fichier pom.xml
COPY pom.xml .
# Copie du répertoire src
COPY src ./src
# Exécution de la commande Maven pour construire le package, en sautant les tests
RUN mvn package -DskipTests

# Exécution
FROM amazoncorretto:17
# Définition de la variable d'environnement MYAPP_HOME
ENV MYAPP_HOME /opt/myapp
# Définition du répertoire de travail
WORKDIR $MYAPP_HOME
# Copie du fichier jar construit dans la première étape dans le répertoire MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

# Point d'entrée pour exécuter le jar
ENTRYPOINT java -jar myapp.jar
```

**Build de l'image**

```docker
docker build -t app-java-with-api . 
```

**Run du conteneur**
```
docker run -p 8080:8080 --name app-java-with-api --net=app-network app-java-with-api
```
  
**docker compose**

```docker
# Définition de la version de Docker Compose utilisée
version: '3.3'

# Définition des services Docker Compose
services:
  # Service backend
  backend:
    # Nom du container pour ce service
    container_name: backend
    # Construit une image à partir du répertoire API2
    build: ./API2
    # Utilise le réseau app-network
    networks:
      - app-network
    # Dépend du service database
    depends_on:
      - database

  # Service database
  database:
    # Nom du container pour ce service
    container_name: database
    # Redémarre automatiquement en cas de panne
    restart: always
    # Construit une image à partir du répertoire postgres
    build: ./postgres
    # Utilise le réseau app-network
    networks:
      - app-network
    # Charge les variables d'environnement à partir du fichier .env dans le répertoire database
    env_file:
      - database/.env

  # Service httpd (reverse proxy)
  httpd:
    # Nom du container pour ce service
    container_name: reverse_proxy
    # Construit une image à partir du répertoire http
    build: ./http
    # Expose le port 80 du host à travers le port 80 du container
    ports:
      - "80:80"
    # Utilise le réseau app-network
    networks:
      - app-network

# Définition des volumes Docker Compose
volumes:
  # Volume my_db_volume
  my_db_volume:
    # Pilote de stockage local
    driver: local

# Définition des réseaux Docker Compose
networks:
  # Réseau app-network
  app-network:
    # Pilote bridge pour permettre la communication entre les services et containers
    driver: bridge
```

**Commandes pour docker-compose**
```docker
docker-compose up # Lance tous les services définis par le fichier docker-compose.yml
docker-compose down # Arrête et supprime tous les coneteneurs
docker-compose ps # Affiche l'état des services
docker-compose logs # Affiche les logs des services
docker-compose build # Construit les images des services définis dans le fichier docker-compose.yml
docker-compose run # Exécute une commande dans un nouveau conteneur
docker-compose config
```

Lancement de tous les services définis par le fichier `docker-compose.yml`:  

```docker
docker-compose up
```
Vérification que les conteneurs sont bien lancés  `docker ps`:  

![alt text](./docker-ps.png)

## Push des images sur DockerHub

**Tague des images:**
```docker
docker tag my-database mquelennec/my-database:1.0
```

**Push des images:**
```docker
docker push mquelennec/my-database
```

> Documentez vos commandes de publication et vos images publiées dans dockerhub