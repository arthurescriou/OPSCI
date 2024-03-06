# TME 7 : Kafka & Event

## Requirement

- Docker
- Server Kafka / ZooKeeper

Ce TME vous guide à travers les étapes de la configuration et de l'utilisation de Kafka avec Docker et des scripts shell.

- Lancer les services Kafka et ZooKeeper avec Docker.
- Créer des producers et des consumers.
- Créer des topics et s'y abonner.
- Envoyer et recevoir des messages.
- Utiliser des groupes de consumers.
- Filtrer les messages par topic et par groupe de consumers.
- Accuser réception des messages.

## Étapes

1. Lancement des services Kafka et ZooKeeper avec Docker compose

```yaml
version: '3'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - '2181:2181'
  kafka:
    image: wurstmeister/kafka
    ports:
      - '9092:9092'
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

Lors de ce TME on a besoin d'utilser les commandes `kafka-topics`, `kafka-console-producer` et `kafka-console-consumer`.
Cependant ces commandes viennent de l'intérieur du conteneur kafka.

Pour éviter des commandes trop longues on va créer des alias (attention les alias n'existeront que dans le terminal dans lequel on les crée).

```bash
### trouver le script  kafka-topics
docker exec -t kafka-kafka-1 find / -name kafka-topics.sh
alias kafka-topics='docker exec -t kafka-kafka-1 $RESULTAT_DE_LA_LIGNE_PRECEDENTE'

### de même pour les scripts suivants
docker exec -t kafka-kafka-1 find / -name kafka-console-producer.sh
alias kafka-console-producer='docker exec -t kafka-kafka-1 $RESULTAT_DE_LA_LIGNE_PRECEDENTE'

docker exec -t kafka-kafka-1 find / -name kafka-console-consumer.sh
alias kafka-console-consumer='docker exec -t kafka-kafka-1 $RESULTAT_DE_LA_LIGNE_PRECEDENTE'
```

#### Résultat:

Les services ZooKeeper et Kafka sont démarrés et accessibles sur les ports 2181 et 9092 respectivement.

{:start="2"}

2. Vérifier la connexion vers le serveur Kafka

```bash
kafka-topics --list --bootstrap-server localhost:9092
```

#### Résultat:

La liste des topics disponibles sur le serveur Kafka est affichée.

{:start="3"}

3. Création de producer

```bash
kafka-console-producer --topic my-topic --bootstrap-server localhost:9092
```

#### Résultat:

Producer simple: Les messages saisis sont envoyés au topic my-topic.

{:start="4"}

4. Création de consumers:

##### Consumer simple:

```bash
kafka-console-consumer --topic my-topic --from-beginning --bootstrap-server localhost:9092
```

##### Consumer avec filtrage:

```bash
kafka-console-consumer --topic my-topic --from-beginning --bootstrap-server localhost:9092 --filter "substring(key, 0, 1) = 'A'"
```

#### Résultat:

- Consumer simple: Les messages du topic my-topic sont affichés au fur et à mesure qu'ils sont produits.
- Consumer avec filtrage: Seuls les messages dont la clé commence par la lettre A sont affichés.

{:start="5"}

5. Création de topics:

```bash
kafka-topics --create --topic my-topic --partitions 1 --replicas 1 --bootstrap-server localhost:9092
```

#### Résultat:

Le topic my-topic est créé avec 1 partition et 1 réplique.

{:start="6"}

6. Abonnement sur les topics:

##### Consumer simple:

```bash
kafka-console-consumer --topic my-topic --from-beginning --bootstrap-server localhost:9092
```

##### Consumer avec groupe:

```bash
kafka-console-consumer --topic my-topic --group my-group --bootstrap-server localhost:9092
```

#### Résultat:

Consumer simple: Les messages du topic my-topic sont reçus et affichés par le consumer.
Consumer avec groupe: Les messages du topic my-topic sont reçus et affichés par un des consumers du groupe my-group.

{:start="7"}

7. Envoie de messages:

Envoi d'un message:

```bash
echo "Hello World!" | kafka-console-producer --topic my-topic --bootstrap-server localhost:9092
```

Envoi de plusieurs messages:

```bash
cat messages.txt | kafka-console-producer --topic my-topic --bootstrap-server localhost:9092
```

#### Résultat:

Envoi d'un message: Le message est affiché par le consumer.
Envoi de plusieurs messages: Tous les messages du fichier sont envoyés et affichés par le consumer.

{:start="8"}

8. Réception de messages:

##### Consumer simple:

```bash
kafka-console-consumer --topic my-topic --from-beginning --bootstrap-server localhost:9092
```

##### Consumer avec groupe:

```bash
kafka-console-consumer --topic my-topic --group my-group --bootstrap-server localhost:9092
```

#### Résultat:

Consumer simple: Les messages du topic my-topic sont reçus et affichés par le consumer.
Consumer avec groupe: Les messages du topic my-topic sont reçus et affichés par un des consumers du groupe my-group.

{:start="9"}

9. Envoie de message de consumer à producer comme quoi ils ont bien reçu les messages:

Consumer avec accusé de réception:

```bash
kafka-console-consumer --topic my-topic --group my-group --bootstrap-server localhost:9092 --ack all
```

Producer avec affichage des accusés de réception:

```bash
kafka-console-producer --topic my-topic --bootstrap-server localhost:9092 --delivery-report-timeout 10000
```

#### Résultat:

Consumer avec accusé de réception: Le consumer envoie un accusé de réception pour chaque message reçu.
Producer avec affichage des accusés de réception: Le producer affiche les accusés de réception reçus.

{:start="10"}

10. Création de groupe de consumers:

Créer un groupe:

```bash
kafka-consumer-groups --create my-group --bootstrap-server localhost:9092
```

Lister les groupes:

```bash
kafka-consumer-groups --list --bootstrap-server localhost:9092
```

Décrire un groupe:

```bash
kafka-consumer-groups --describe my-group --bootstrap-server localhost:9092
```

#### Résultat:

Créer un groupe: Le groupe my-group est créé.
Lister les groupes: La liste des groupes créés est affichée.
Décrire un groupe: Les informations du groupe my-group sont affichées.

{:start="11"}

11. Faire que les groupes s'abonnent sur différents topics:

Consumer du groupe 1:

```bash
kafka-console-consumer --topic topic-1 --group group-1 --bootstrap-server localhost:9092
```

Consumer du groupe 2:

```bash
kafka-console-consumer --topic topic-2 --group group-2 --bootstrap-server localhost:9092
```

#### Résultat:

- Consumer du groupe 1: Seuls les messages du topic topic-1 sont reçus et affichés.
- Consumer du groupe 2: Seuls les messages du topic topic-2 sont reçus et affichés.

{:start="12"}

12. Producer envoie des messages que sur des topics en particulier:

Producer envoie sur topic-1:

```bash
kafka-console-producer --topic topic-1 --bootstrap-server localhost:9092
```

Producer envoie sur topic-2:

```bash
kafka-console-producer --topic topic-2 --bootstrap-server localhost:9092
```

#### Résultat:

- Producer envoie sur topic-1: Les messages saisis sont envoyés au topic topic-1.
- Producer envoie sur topic-2: Les messages saisis sont envoyés au topic topic-2.

{:start="13"}

13. Réception de message que pour le groupe de consumers qui sont abonné:

Consumer du groupe 1:

```bash
kafka-console-consumer --topic topic-1 --group group-1 --bootstrap-server localhost:9092
```

Consumer du groupe 2:

```bash
kafka-console-consumer --topic topic-2 --group group-2 --bootstrap-server localhost:9092
```

#### Résultat:

Consumer du groupe 1: Seuls les messages du topic topic-1 sont reçus et affichés.
Consumer du groupe 2: Seuls les messages du topic topic-2 sont reçus et affichés.

{:start="14"}

14. Les consumer qui ont reçu le message envoie une réponse au producers:

Consumer avec envoi de réponse:

```bash
# Implémenter la logique d'envoi de réponse au producer dans le code du consumer
```

Producer avec réception des réponses:

```bash
# Implémenter la logique de réception des réponses des consumers dans le code du producer
```

#### Résultat:

Consumer avec envoi de réponse: Le consumer envoie une réponse au producer pour chaque message reçu.
Producer avec réception des réponses: Le producer affiche les réponses reçues des consumers.
Note:

Les configurations et les résultats peuvent varier en fonction de votre environnement et des données que vous utilisez.
N'hésitez pas à adapter les commandes et les configurations à vos besoins spécifiques.

## scheduling

1. Envoi automatique de messages

```bash

#!/bin/bash

# Remplacer par vos valeurs
TOPIC="my-topic"
BROKER_LIST="localhost:9092"

# Envoyer des messages en boucle
while true; do
  MESSAGE=$(cat messages.txt | shuf -n 1)
  echo "Envoi du message : $MESSAGE"
  echo "$MESSAGE" | kafka-console-producer --topic $TOPIC --bootstrap-server $BROKER_LIST
  sleep 1
done
```

{:start="2"}

2. Envoi programmé de messages

```bash


#!/bin/bash

# Remplacer par vos valeurs
TOPIC="my-topic"
BROKER_LIST="localhost:9092"
MESSAGE="Votre message personnalisé"

# Définir l'heure d'envoi
TIME="10:00"

# Envoyer le message à l'heure programmée
while true; do
  if [[ $(date +%H:%M) == $TIME ]]; then
    echo "Envoi du message : $MESSAGE"
    echo "$MESSAGE" | kafka-console-producer --topic $TOPIC --bootstrap-server $BROKER_LIST
    break
  fi
  sleep 1
done
```

{:start="3"}

3. Envoi programmé de messages avec cron

...

Pour aller plus loin

- Gestion des erreurs: Ajoutez des vérifications d'erreurs dans les scripts pour gérer les problèmes de connexion à Kafka ou d'envoi de messages.
- Logs: Implémentez un système de journalisation pour suivre les envois de messages et les erreurs éventuelles.
- Parallélisation: Envoyez plusieurs messages en parallèle pour augmenter le débit.
- Authentification: Configurez l'authentification pour sécuriser l'accès à Kafka.
