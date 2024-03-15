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

alias kafka-console-producer='docker exec -i kafka-kafka-1 $RESULTAT_DE_LA_LIGNE_PRECEDENTE'

docker exec -t kafka-kafka-1 find / -name kafka-console-consumer.sh
alias kafka-console-consumer='docker exec -it kafka-kafka-1 $RESULTAT_DE_LA_LIGNE_PRECEDENTE'

docker exec -it kafka-kafka-1 find / -name kafka-consumer-groups.sh
alias kafka-consumer-groups='docker exec -it kafka-kafka-1 $RESULTAT_DE_LA_LIGNE_PRECEDENTE'
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
echo "test" | kafka-console-producer --topic [TOPIC] --bootstrap-server localhost:9092
```

#### Résultat:

Les messages saisis sont envoyés au topic `[TOPIC]`.

{:start="4"}

4. Création de consumers:

##### Consumer simple:

```bash
kafka-console-consumer --topic [TOPIC] --from-beginning --bootstrap-server localhost:9092
```

#### Résultat:
Les messages du topic `[TOPIC]` sont affichés au fur et à mesure qu'ils sont produits.

{:start="5"}

5. Création de topics:

```bash
kafka-topics --create --topic [TOPIC] --partitions 1 --replication-factor 1 --bootstrap-server localhost:9092
```

#### Résultat:

Le topic `[TOPIC]` est créé avec 1 partition et 1 réplique.

{:start="6"}

6. Abonnement sur les topics:

#### Consumer simple:

```bash
kafka-console-consumer --topic [TOPIC] --from-beginning --bootstrap-server localhost:9092
```
#### Résultat:
* Les messages du topic `[TOPIC]` sont reçus et affichés par le consumer.

#### Consumer avec groupe:

```bash
kafka-console-consumer --topic [TOPIC] --group [GROUP] --bootstrap-server localhost:9092
```

#### Résultat:
* Les messages du topic `[TOPIC]` sont reçus et affichés par un des consumers du groupe `[GROUP]`.

#### Vérfier la création du group

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group [GROUP]
```

#### Résultat:
```
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
[GROUP]        test-topic      0          4               4               0               -               -               -
```

{:start="7"}

7. Envoie de messages:

#### Envoi d'un message à un consummer:

```bash
echo "Hello World!" | kafka-console-producer --topic [TOPIC] --bootstrap-server localhost:9092
```

#### Envoi de plusieurs messages:

```bash
cat messages.txt | kafka-console-producer --topic [TOPIC] --bootstrap-server localhost:9092
```

#### Résultat:

Envoi d'un message: Le message est affiché par le consumer.
Envoi de plusieurs messages: Tous les messages du fichier sont envoyés et affichés par le consumer.

{:start="8"}

8. Réception de messages:

##### Consumer simple:

```bash
kafka-console-consumer --topic [TOPIC] --from-beginning --bootstrap-server localhost:9092
```

#### Résultat:

Consumer simple: Les messages du topic `[TOPIC]` sont reçus et affichés par le consumer.


#### Consumer avec groupe:

* Vérifier la description du group :
```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --describe  --group [GROUP]
```

* Consommer les messages dans un group :
```bash
kafka-console-consumer --topic [TOPIC] --group [GROUP]  --bootstrap-server localhost:9092
```
#### Résultat:
Les messages du topic [TOPIC] sont reçus et affichés par un des consumers du groupe `[GROUP]`.


Note : Si cela ne fonctionne pas, essayer de réinitialiser les offsets du groupe `[GROUP]` pour le sujet `[TOPIC]`:

```bash
kafka-consumer-groups --bootstrap-server localhost:9092   --group [GROUP]  --reset-offsets   --to-earliest   --execute  --topic [TOPIC]
```

##### Resultat :
```
GROUP                          TOPIC                          PARTITION  NEW-OFFSET
[GROUP]                       [TOPIC]                     0          0
```

Cette commande permet de réinitialiser des offsets du groupe "[GROUP]" pour le topic "[TOPIC]" en positionnant les offsets de consommation au tout début du sujet, c'est-à-dire à partir du premier message disponible.

```bash
kafka-console-consumer --topic test-topic --group [GROUP]  --bootstrap-server localhost:9092
```

#### Résultat:
Les messages du topic [TOPIC] sont reçus et affichés par un des consumers du groupe `[GROUP]`.

{:start="9"}

9. Envoie de message de consumer à producer comme quoi ils ont bien reçu les messages:

#### Consumer avec accusé de réception:

* Producer : Envoyer des messages avec des clés :

Lorsque le producteur envoie des messages, il doit ajouter une clé unique à chaque message. Cette clé sera utilisée par le consumer pour envoyer un accusé de réception pour chaque message spécifique.

```bash
kafka-console-producer --topic test-topic --broker-list localhost:9092  --property "parse.key=true" --property "key.separator=:"
#> Resultat :
>1:msg1
>2:msg2
```

* Consumer : Consommer les messages et envoyer des accusés de réception :

Lorsque vous configurez le consumer Kafka pour envoyer automatiquement des accusés de réception (acks) en utilisant `enable.auto.commit=true, les accusés de réception sont envoyés au groupe de consumers pour le topic spécifique à partir duquel vous lisez les messages.

```bash
kafka-console-consumer.sh \
  --topic test-topic1 \
  --group group1 \
  --bootstrap-server localhost:9092 \
  --property "print.key=true" \
  --property "key.separator=:" \
  --property "enable.auto.commit=true" \
  --property "auto.offset.reset=earliest" \
  --property "key.deserializer=org.apache.kafka.common.serialization.StringDeserializer" \
  --property "value.deserializer=org.apache.kafka.common.serialization.StringDeserializer"
```
Les accusés de réception seront envoyés automatiquement au groupe de consumers group1 pour le topic test-topic1.

#### Resultat :
```
1:msg1
2:msg2
```

* Vérifier les accusés de réception :
#### 1. Vérification de l'Offset Commit
```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --group group1 --describe
```
  
Cette commande affichera les informations sur les offsets consommés pour le groupe group1 et le topic test-topic1.

#### Resultat :
Consumer group 'group1' has no active members.
```
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
group1          test-topic1     0          3               3               0               -               -               -
```

#### 2. Vérification des Accusés de Réception
Vous pouvez également vérifier les accusés de réception directement dans le topic spécial `__consumer_offsets`. Ce topic est utilisé par Kafka pour stocker les informations sur les offsets commis par les groupes de consumers.

Pour afficher les messages dans le `topic __consumer_offsets`, vous pouvez utiliser la commande suivante :

```bash
kafka-console-consumer.sh \
  --topic __consumer_offsets \
  --bootstrap-server localhost:9092 \
  --from-beginning
```

Cette commande affichera les messages dans le topic `__consumer_offsets`, qui inclura les enregistrements pour les accusés de réception envoyés par votre groupe de consumers group1 pour le topic test-topic1.

En vérifiant les accusés de réception dans ce topic, vous pouvez confirmer que les offsets sont bien commis par votre consumer Kafka. 

{:start="10"}

10. Création de groupe de consumers:

Créer un groupe:

```bash
kafka-consumer-groups --create [GROUP] --bootstrap-server localhost:9092
```

Vérifier l'existance du group : 

```bash
kafka-consumer-groups   --bootstrap-server localhost:9092 --describe --group [GROUP]
```

##### Exemple de resultat 
```bash

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
[GROUP]        test-topic      0          0               6               6               -               -               -
[GROUP]        [TOPIC]        0          3               3               0               -               -               -
```

Lister les groupes:

```bash
kafka-consumer-groups --list --bootstrap-server localhost:9092
```

Décrire un groupe:

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group [GROUP]
```


#### Résultat:

Créer un groupe: Le groupe `[GROUP]` est créé.
Lister les groupes: La liste des groupes créés est affichée.
Décrire un groupe: Les informations du groupe `[GROUP]` sont affichées.

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
TOPIC="[TOPIC]"
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
TOPIC="[TOPIC]"
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
