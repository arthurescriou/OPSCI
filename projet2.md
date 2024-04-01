# Projet OPSCI: Architecture évènementielle

Ce projet à pour objectif d'ajouter des éléments évènementiel à aux éléments créés dans le premier projet.

## Prérequis

Pour facilier le lancement du premier projet vous pouvez utiliser ce docker-compose, ou directement votre travail.

Si vous utilisez votre travail il faudra modifier le type de `product` :
Ajouter un champs `status` de type `enumeration` : `safe|danger|empty`

Et créer un type event avec un champ `value` de type `string` et un champ `metadata` de type `JSON`.

Le projet initialise les formats des objets nécessaire au projet 2. Il peut être nécessaire de regénérer un API TOKEN.

```yaml
version: '3'
services:
  strapi:
    container_name: strapi
    image: arthurescriou/strapi:1.0.0
    restart: unless-stopped
    env_file: .env
    environment:
      DATABASE_CLIENT: ${DATABASE_CLIENT}
      DATABASE_HOST: strapiDB
      DATABASE_PORT: ${DATABASE_PORT}
      DATABASE_NAME: ${DATABASE_NAME}
      DATABASE_USERNAME: ${DATABASE_USERNAME}
      DATABASE_PASSWORD: ${DATABASE_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      ADMIN_JWT_SECRET: ${ADMIN_JWT_SECRET}
      APP_KEYS: ${APP_KEYS}
      NODE_ENV: ${NODE_ENV}
      PORT: 8080
    ports:
      - '8080:8080'
    networks:
      - strapi
    depends_on:
      - strapiDB

  strapiDB:
    container_name: strapiDB
    restart: unless-stopped
    env_file: .env
    image: arthurescriou/strapi-pg:1.0.0
    environment:
      POSTGRES_USER: ${DATABASE_USERNAME}
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
      POSTGRES_DB: ${DATABASE_NAME}

    ports:
      - '5432:5432'
    networks:
      - strapi

networks:
  strapi:
    name: Strapi
    driver: bridge
```

## Architecture

On souhaite créer un système permettant d'intégrer de grande quantité de données venant de différents flux avec beaucoup de résilience à l'erreur.

Pour ça on utilise un message broker: Kafka.

<img src="./img/projet2.png"/>

### Kafka

On veut lancer un kafka avec docker (ou kubernetes).

On utilise la même solution qu'au TME7.

```yml
version: '3'
services:
  zookeeper:
    image: wurstmeister/zookeeper:latest
    container_name: zookeeper
    ports:
      - "2181:2181"
    expose:
      - "2181"

  kafka:
    image: wurstmeister/kafka:2.11-1.1.1
    container_name: kafka
    ports:
      - "9092:9092"
      - "9093:9093"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://localhost:9093,OUTSIDE://kafka:9092,
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_DELETE_TOPIC_ENABLE: "true"
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ZOOKEEPER_CONNECT: "zookeeper:2181"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKAJS_NO_PARTITIONER_WARNING: "1" 
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_NO_LISTENER_AUTHENTICATION_PLAINTEXT: "true"
      KAFKA_NO_LISTENER_AUTHENTICATION_SSL: "true"
      KAFKA_BROKER_ID: 1
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_RETENTION_BYTES: 1073741824
      KAFKA_LOG_DIRS: /kafka/logs
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - zookeeper

```

### Topics

Notre kafka va nécéssiter plusieurs topics :

- product : un topic dédié à la création de nouveau produit en masse, venant de différentes sources
- event : un topic dédié à la création de nouveau produit en masse, venant de différentes sources
- stock : un topic pour enregistrer et appliquer tous les mouvements de stocks de nos produits
- error : pour récupérer les évènement ayant abouti à une erreur diverse

_Attention tous ces topics devront être précisés dans les configurations des consumers et producers. Si le noms n'est pas le même des deux cotées la communication ne marchera pas._

### Consumer et Producer

Pour utiliser ces topics plusieurs élèments sont à votre disposition sous la forme d'image docker ou de dépôt github (<a href="https://github.com/orgs/opsci-su/repositories">https://github.com/orgs/opsci-su/repositories</a>).

Il s'agit des consumers et producers pour les différents éléments.

Pensez bien à lire les readme des différents élèments pour bien configurer leurs variables d'environments.

#### product-producer

<a href="https://hub.docker.com/repository/docker/arthurescriou/product-producer/" > https://hub.docker.com/repository/docker/arthurescriou/product-producer/ </a>

Pour lancer la création de `product` insérez le <a href="./assets/products.csv" >fichier</a> dans votre container et spécifiez sa position dans la configuration.

#### product-consumer

<a href="https://hub.docker.com/repository/docker/arthurescriou/product-consumer/" > https://hub.docker.com/repository/docker/arthurescriou/product-consumer/ </a>

#### event-producer

<a href="https://hub.docker.com/repository/docker/arthurescriou/event-producer/" > https://hub.docker.com/repository/docker/arthurescriou/event-producer/ </a>

#### event-consumer

<a href="https://hub.docker.com/repository/docker/arthurescriou/event-consumer/" > https://hub.docker.com/repository/docker/arthurescriou/event-consumer/ </a>

#### stock-producer

<a href="https://hub.docker.com/repository/docker/arthurescriou/stock-producer/" > https://hub.docker.com/repository/docker/arthurescriou/stock-producer/ </a>

#### stock-consumer

<a href="https://hub.docker.com/repository/docker/arthurescriou/stock-consumer/" > https://hub.docker.com/repository/docker/arthurescriou/stock-consumer/ </a>

#### artificial-intelligence

_TBA_
