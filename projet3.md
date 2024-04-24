# Projet OPSCI: Internet des objets

Ce projet à pour objectif d'ajouter la gestion d'objets connectés dans l'architecture créé dans les deux premiers projets.

## Prérequis

Pour ce projet on a besoin de l'architecture évènementiel car on va créer des producteur "IoT" pour le kafka.

Il faut donc un kafka fonctionnel avec un strapi pour consommer les données.

## Architecture

On souhaite créer un système récupérant des données provenant du "terrain". Cependant on ne peut pas exposer notre kafka à l'extérieur de notre data center pour des raisons de sécurité.

On va donc passer par un autre message broker pour communiquer avec nos client "IoT".

<img src="./img/mqtt-OPSCI.jpg"/>

### MQTT

On veut lancer un broker MQTT.

Pour ça on va utiliser <a href="https://mosquitto.org/">Mosquitto</a>, une implémentation open source de MQTT.

On peut le lancer facilement avec <a href="https://hub.docker.com/_/eclipse-mosquitto" >l'image officielle </a>.

Il suffit de créer un fichier `mosquitto.conf` et de le lier avec un volume.

```
#mosquitto.conf
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
allow_anonymous true
listener 1883
```

Une fois le conteneur lancé on peut se connecter dessus avec client `mqtt` sur le port 1883.

Vous pouvez utiliser ces <a href="https://github.com/arthurescriou/mqtt-js-test">scripts</a> pour tester votre broker mosquitto.
