# Semainier OPSCI

## Partie 1 Infrastructure

### 1: Linux/systeme (17/01)

Commandes linux basiques :

- cd
- ls
- cp/mv
- chmod

Configuration ssh

Création de script automatisé (début de la CI/CD)

Exécution de script avec un moteur de CRON

Utilisation de git (connexion en clé ssh)

#### Partie 1 machine local

ex 1 :
Commandes linux basique :

- ls
- cd
- whoami
- pwd
- df
- du
- find/grep
- ps

Objectif savoir se situer dans une machine
un fichier/report config

ex 2 :
Script => automatiser ex 1

ex 3:
Git => pousser leur script
Gitlab navigateur

#### Partie 2

ex 4
ifconfig tshark nmap
objectif => la machine distante

ex5
ssh sur la machine
objectif clé ssh et supprimer le password

ex6
gitlab ssh
objectif => lancer leur script de l'ex 2 sur la machine distante

ex7
CRON

### 2: Docker/ Docker compose (24/01)

Lancer un container docker :

- configurer son environment
- docker login sur le repository / gestion des secrets

Création de fichiers docker-compose

### 3: K8S intro (31/01)

- connexion au cluster K8S avec kubectl
- lancement d'un pod / deploiement / service
- deploiement de Strapi
- création de fichiers .yml K8S

### 4: K8S debug (07/02)

Troubleshooting K8S

- récupérations des logs des pods
- installation de monitoring sur l'application déployé

### 5: CI/CD (14/02)

Création d'une pipeline de deploiement automatisée. (gitlabCI)

### 6: Scaling / Loadbalancing / Rolling Update (06/03)

Création d'une infrastructure type "prod" :

- auto-scaling
- Rolling update pour les nouvelles versions

## Evalutation Partie 1

## Partie 2 Data

### 7: Import de données dans Strapi (13/03)

Import en batch de données à partir d'un fichiers CSV via un script bactch (event sourcing/pooling)

### 8: Export automatique (20/03)

Export de données automatisé récurent avec un CRON dans le filesystem.

### 9: Data pipeline (27/03)

Ingestion récurente de données dans un module de "calcul" et sauvegarde des résultats.

## Evalutation Partie 2

## Partie 3 IoT

### 10: IoT sur le terrain (03/04)

Utilisation de capteurs NFC et QR codes pour créer de l'information. (autres capteurs si on trouve)

### 11: Connecter l'IoT (24/04) ??

Connexion des capteurs via MQTT à la pipeline de données (sécurisé par SSL)

## Evalutation Partie 3
