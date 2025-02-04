1.1 cela evite de devoir mettre les variable dans le dockerfile ce qui permet de faire des modifications plus facilement

1.2 Un volume est nécessaire pour garantir la persistance des données de la base de données, cela évite que les données soit supprimées 
quand le conteneur est supprimé. Grace au volume les données sont stockées en dehors du conteneur sur le disque de l'hote.

1.3 Voici le dockerfile pour postgres

FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/

FROM postgres:14.1-alpine : On utilise l'image officielle de PostgreSQL 14.1 basée sur Alpine Linux, qui est une image plus légère et donc plus efficace.

Variables d'environnement (POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD) : Ces variables permettent de définir la base de données par défaut, l'utilisateur
et le mot de passe lors du démarrage de PostgreSQL. Cela permet d'automatiser la configuration de la base de données au lancement du conteneur.
COPY des scripts SQL dans le répertoire /docker-entrypoint-initdb.d/ : Les fichiers SQL sont copiés dans ce répertoire, où ils seront automatiquement exécutés par 
PostgreSQL au démarrage du conteneur (uniquement lors de la première initialisation de la base de données). Les scripts vont créer les tables et insérer les données nécessaires.

1.4 Le Dockerfile utilise une construction multistage pour séparer la compilation et l'exécution de l'application. Dans la première étape, 
nous utilisons Maven pour compiler le code source, tandis que dans la deuxième étape, nous utilisons une image plus légère (sans Maven) pour exécuter l'application Java. Cela permet de réduire la taille de l'image finale en ne gardant que ce qui est nécessaire pour l'exécution (le fichier .jar), et non les outils de développement comme Maven.
Cette approche permet de créer des images Docker plus efficaces et plus petites, ce qui est essentiel dans les environnements de production.

1.5 Le reverse proxy permet de gérer le trafic entre les utilisateurs et les serveurs backend. Ici il  redirige les requetes des utilisateurs vers l'application backend.

1.6 Docker compose permet de gérer facilement plusieurs conteneurs Docker. Ainsi cela evite de devoir faire des docker run et permet de définir toute l'architecture de l'application
avec l'orchestration des conteneurs, la gestion des réseaux et la configuration des volumes.

1.7 
Démarrer l'application (les services) :
docker-compose up

Démarrer l'application en mode détaché (en arrière-plan) :
docker-compose up -d

Arrêter l'application :
docker-compose down

Voir le statut des services :
docker-compose ps

Construire ou reconstruire les services :
docker-compose build

Voir les logs de tous les services :
docker-compose logs

1.8 mon docker compose:

services:
  backend:
    build: "Backend API"  # Répertoire contenant le Dockerfile pour le backend
    networks:
      - my-network  # Connecte le service au réseau my-network
    ports:
      - "8080:8080"  # Mappe le port 8080 du conteneur au port 8080 de la machine hôte
    depends_on:
      - database  # Le service backend dépend de la base de données, donc il sera démarré après le service database

  database:
    build: "database"  # Répertoire contenant le Dockerfile pour la base de données
    environment:
      POSTGRES_DB: db  # Nom de la base de données
      POSTGRES_USER: usr  # Utilisateur de la base de données
      POSTGRES_PASSWORD: pwd  # Mot de passe pour l'utilisateur de la base de données
    networks:
      - my-network  # Connecte le service au même réseau my-network que les autres services

  httpd:
    build: "Server HTTP"  # Répertoire contenant le Dockerfile pour le serveur HTTP (probablement Apache ou autre)
    ports:
      - "80:80"  # Mappe le port 80 du conteneur au port 80 de la machine hôte
    networks:
      - my-network  # Connecte le service au réseau my-network
    depends_on:
      - backend  # Le service httpd dépend du backend, donc il sera démarré après le service backend

networks:
  my-network:  # Définit le réseau nommé 'my-network' pour que les services puissent communiquer entre eux


1.10 Mettre nos images dans un dépôt en ligne comme Docker Hub permet de les partager et de les rendre accessibles à d'autres personnes ou systèmes. Cela facilite l'accès aux images depuis
différentes machines, et permet de les utiliser dans différents environnements (par exemple, en production ou en développement). De plus, cela permet de versionner les images et de suivre les différentes versions utilisées.
