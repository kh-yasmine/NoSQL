# Tutoriel d'installation et d'utilisation de Redis avec Docker

## 1. Introduction

Ce tutoriel présente l'installation de **Redis** à l'aide de **Docker**,
ainsi que quelques commandes de base pour manipuler les données.\
Il est inspiré des vidéos du cours

------------------------------------------------------------------------

## 2. Installation de Docker (si nécessaire)

Pour vérifier si Docker est déjà installé :

``` bash
docker --version
docker compose version
```

Si Docker n'est pas installé, suivez la documentation officielle :\
https://docs.docker.com/engine/install/ubuntu/

------------------------------------------------------------------------

## 3. Installation de Redis avec Docker

Téléchargez et lancez Redis dans un conteneur Docker :

``` bash
docker run -d --name redis -p 6379:6379 redis
```

Vérifiez que le conteneur fonctionne :

``` bash
docker ps
```

------------------------------------------------------------------------

## 4. Accéder à Redis via redis-cli

Pour entrer dans le client Redis intégré au conteneur :

``` bash
docker exec -it redis redis-cli
```

Vous devriez obtenir un prompt :

    127.0.0.1:6379>

------------------------------------------------------------------------

## 5. Commandes Redis de base

### 5.1 SET & GET & DEL

``` bash
SET user:1234 "Yasmine"
GET user:1234
DEL user:1234
```

### 5.2 Incrémentation et décrementation 

``` bash
SET compteur 1
INCR compteur
GET compteur
DECR compteur
```

### 5.3 Listes

``` bash
LPUSH fruits "pomme"
LPUSH fruits "banane"
LRANGE fruits 0 -1
```

### 5.4 Hashes

``` bash
HSET utilisateur name "Yasmine" age 22
HGETALL utilisateur
```

------------------------------------------------------------------------

## 6. Persistance des données (Docker Compose conseillé)

Créez un fichier `docker-compose.yml` :

``` yaml
services:
  redis:
    image: redis:latest
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

Lancez Redis :

``` bash
docker compose up -d
```

------------------------------------------------------------------------

## 7. Gestion du conteneur

Arrêter Redis :

``` bash
docker stop redis
```

Redémarrer :

``` bash
docker start redis
```

Supprimer :

``` bash
docker rm -f redis
```

------------------------------------------------------------------------

## 8. Conclusion

Vous disposez maintenant d'une installation fonctionnelle de Redis via
Docker et d'un ensemble de commandes essentielles pour manipuler des
données.\
Ce tutoriel peut servir de base pour un rapport, un TP ou une initiation
aux bases NoSQL.

------------------------------------------------------------------------
