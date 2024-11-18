# Projet Fullstack avec Docker : Backend Node.js et Frontend React

## Table des matières

- [Description du projet](#description-du-projet)
- [Architecture](#architecture)
- [Prérequis](#prérequis)
- [Installation et démarrage](#installation-et-démarrage)
- [Dockerfiles](#dockerfiles)
  - [Backend](#backend)
  - [Frontend](#frontend)
- [Gestion de la persistance des données](#gestion-de-la-persistance-des-données)
- [Mise en réseau des conteneurs](#mise-en-réseau-des-conteneurs)
- [Test et validation](#test-et-validation)
- [Bonnes pratiques](#bonnes-pratiques)
- [Conclusion](#conclusion)

---

## Description du projet

Ce projet consiste en une application fullstack composée :
- D'un **backend** en Node.js pour gérer l'API et la communication avec une base de données PostgreSQL.
- D'un **frontend** en React qui consomme l'API du backend et sert l'interface utilisateur.
- La persistance des données est assurée via un volume Docker pour PostgreSQL.

L'ensemble de l'application est dockerisé pour assurer une portabilité maximale et faciliter la mise en production.

---

## Architecture

L'application se compose de trois conteneurs principaux :
1. **Backend** (Node.js) : expose une API REST pour interagir avec la base de données.
2. **Frontend** (React) : une application web statique servie par Nginx.
3. **Base de données** (PostgreSQL) : pour la persistance des données.

Les conteneurs sont mis en réseau via Docker et communiquent entre eux pour fournir une solution complète.

---

## Prérequis

Avant de commencer, assurez-vous d'avoir installé les outils suivants :
- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)

---

## Installation et démarrage

### 1. Clonez le dépôt

```bash
git clone https://github.com/IannisVasselet/concepts-base.git
cd concepts-base
```

### 2. Construisez et lancez les conteneurs

#### Option 1 : Utiliser Docker directement

Commencez par créer le réseau et le volume Docker :

```bash
docker network create app-network
docker volume create db-data
```

Ensuite, démarrez les conteneurs individuellement :

```bash
# Démarrer le backend
docker build -t backend-image ./backend
docker run -d --name backend --network app-network backend-image

# Démarrer le frontend
docker build -t frontend-image ./frontend
docker run -d --name frontend --network app-network -p 80:80 frontend-image

# Démarrer la base de données
docker run -d --name db --network app-network -v db-data:/var/lib/postgresql/data -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password -e POSTGRES_DB=mydb postgres:14
```

#### Option 2 : Utiliser Docker Compose

```yaml
version: '3'
services:
  backend:
    build: ./backend
    networks:
      - app-network
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:password@db:5432/mydb

  frontend:
    build: ./frontend
    networks:
      - app-network
    ports:
      - "80:80"
    environment:
      - REACT_APP_API_URL=http://localhost:3000

  db:
    image: postgres:14
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    networks:
      - app-network

networks:
  app-network:

volumes:
  db-data:
```

Ensuite, lancez tous les services avec :

```bash
docker-compose up --build
```

---

## Dockerfiles

### Backend

Le Dockerfile pour le backend utilise un **multi-stage build** pour optimiser la taille de l'image finale en ne gardant que les fichiers nécessaires à l'exécution.

```dockerfile
# Étape 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install --only=production
COPY . .
RUN npm run build

# Étape 2: Run
FROM node:18-slim AS run
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app /app
CMD ["npm", "start"]
```

### Frontend

Le Dockerfile pour le frontend suit le même principe, utilisant une image Nginx pour servir les fichiers statiques compilés par React.

```dockerfile
# Étape 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Étape 2: Run
FROM nginx:alpine AS run
COPY --from=build /app/build /usr/share/nginx/html

# Configuration Nginx
COPY nginx.conf /etc/nginx/nginx.conf
```

---

## Gestion de la persistance des données

Nous utilisons un **volume Docker** pour assurer que les données de la base de données ne soient pas perdues lors du redémarrage du conteneur PostgreSQL.

### Créer le volume Docker

```bash
docker volume create db-data
```

Ce volume est monté dans le conteneur PostgreSQL, comme indiqué dans les Dockerfiles et `docker-compose.yml`.

---

## Mise en réseau des conteneurs

Les conteneurs sont connectés via un réseau Docker personnalisé. Cela permet au backend et à la base de données de communiquer via des noms de services.
### Créer le réseau Docker

```bash
docker network create app-network
```

---

## Test et validation

### 1. Tester l'interface utilisateur

- Accédez à localhost pour ouvrir l'application frontend.
- Le frontend doit communiquer avec le backend et afficher les données récupérées via l'API. (API comming soon)

### 2. Tester la persistance des données

1. Ajoutez des données via le frontend ou en utilisant l'API du backend.
2. Redémarrez les conteneurs avec la commande suivante :

```bash
docker stop <container_name>
docker start <container_name>
```

3. Vérifiez que les données sont toujours présentes après le redémarrage.

---

Ce projet présente une architecture fullstack dockerisée avec un backend Node.js, un frontend React, et une base de données PostgreSQL. L'usage des multi-stage builds permet d'optimiser la taille des images Docker, et la persistance des données est gérée via des volumes Docker. Le réseau Docker assure une communication fluide entre les conteneurs.

## Modes de lancement et commandes

### Lancer les services avec Docker Compose

Pour lancer tous les services, utilisez la commande suivante :

```sh
docker-compose up --build -d
```

### Arrêter les services

Pour arrêter les services, utilisez la commande suivante :

```sh
docker-compose down
```

### Valider la configuration
```sh
docker-compose config
``` 