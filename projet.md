# Projet 1 OPSCI

L'objectif de ce projet est de déployer la première partie de l'infrastructure de OPSCI.

Ce projet est composé de 3 parties pour pouvoir fonctionner :

- strapi : un serveur CMS (content manager system)
- la base de données de strapi
- un frontend web utilisant l'API de strapi

## Strapi

La première étape pour déployer l'application est de créer un projet strapi.

```sh
yarn create strapi-app ${project-name}
```

Configurez l'application (la base de données n'a pas trop d'importance pour le moment).
Mais il faudra s'en souvenir pour déployer la bonne base de données plus tard.

Une fois le projet créé (dans le dossier `project-name`) créez un Dockerfile pour créer une image docker du projet.

Strapi nous fourni une documentation et un exemple de dockerfile : https://docs.strapi.io/dev-docs/installation/docker
