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

### Kafka

_TODO Lancer kafka_

### Topics

### Consumer et Producer

_TODO_

- product
- event
- stock
- status
