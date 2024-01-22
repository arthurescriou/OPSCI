# TME 2 OpsCI

L'objectif de ce TME est d'apprendre à utiliser les fonctionnalité basique de docker et de se familiariser avec l'utilisation de machine distante.

## Partie 1 : Se connecter à une machine distante

### Exercice 1

Pour commencer on veut se connecter avec une machine distante en utilisant du ssh.

```sh
ssh root@${URL_SSH} -v
```

Une fois sur cette machine distante on veut lancer le script créé au TME précédent. (Attention vous allez être plusieurs sur la même machine pensez bien à créer vos propres dossiers).

Plusieurs solutions :

- utiliser le protocole ftp pour transférer le fichier de votre machine vers la machine distante

```sh
scp /path/to/script/report.sh root@${URL_SSH}:/distant/path/to/script/your/name/report.sh
```

- faire un `git pull` de votre dépôt git depuis la machine distante
  (attention il faut avoir créer un dépôt sur un fournisseur git en ligne cf TME1 exercice 6)

- copier coller le contenu du script avec un editeur de fichier en terminal

### Exercice 2

Ajouter sa clé ssh pour s'authentifier sans mot de passe.

## Partie 2 : Docker

### Exercice 3

On veut maintenant lancer un conteneur docker :

```sh
export container_name='ubuntu'
docker run -itd --name ${container_name} ubuntu
```

Maintenant que le conteneur est lancé il est possible de se connecter dessus avec docker.

```sh
docker exec -it ${container_name} bash
```

De la même manière que pour l'Exercice 1 lancez le script `report.sh` dans le conteneur.

### Exercice 4

Il est également possible de se connecter au conteneur docker via SSH mais pour ça on doit configurer notre conteneur et son réseau.

Créez un fichier `Dockerfile`.

```dockerfile
FROM debian:latest

RUN apt-get update && apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:root123' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
```

On veut maintenant créer une image à partir de ce fichier :

```sh
export image_name='ssh_container'
docker build . -t ${image_name}
```

Et on peut la lancer comme dans l'exercice 3

```sh
docker run -itd --name ${container_name} ${image_name}
```

Un nouveau conteneur acceptant le `SSH` à été créé.

Cependant le réseau docker ne permet pas d'accéder au conteneur sur le port ssh : 22.

Pour se connecter en `SSH` il faut changer la configuration de lancement du conteneur. On va donc commencé par l'arréter et le supprimer.

```sh
docker stop ${container_name}
docker rm ${container_name}
```

Pour pouvoir le relancer correctement

```sh
docker run -itd -p 22:22 --name ${container_name} ${image_name}
```

On peut maintenant se connecter dessus en ssh.

```sh
ssh root@localhost
```

Ajouter une clé ssh dans le conteneur pour se connecter dessus sans mot de passe.

### Exercice 5

Maintenant qu'on connait quelques commandes docker il est temps de construire une image par nous même.

Pour cet exercice on veut créer un conteneur hébergeant un site statique.

Vous pouvez récupérer un site html/css sur de dépôt de l'UE sur la `branche` `tme2-site` dans le dossier `site`.

Vous devez maintenant créer un Dockerfile pour créer une image docker permettant de rendre ces fichiers disponible via http.

Quelques exemples pour créer un serveur http simplement :

- en python : `server='python3 -m http.server'` dans le bon dossier
- avec Nginx :

```Dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

- en <a href="https://developer.mozilla.org/en-US/docs/Learn/Server-side/Node_server_without_framework">node.js</a>

Ou tout autres exemples trouvés sur internet.

Une fois le Dockerfile écrit faites un `docker build` comme dans l'exercice 4.

Et lancez votre conteneur en configurant les ports correctement.

## Partie 3 : Le réseau en docker

Dans cette partie on veut faire fonctionner plusieurs conteneur ensemble.

Grace au script `report.sh` on sait récupérer des informations sur le système courant.

Mais pour cette partie on est surtout interessé par la commande `hostname`.
Lorsqu'on est dans un conteneur docker le hostname correspond à l'`ID` du conteneur.

#### Exercice 6

Nous voulons communiquer entre deux conteneur.

Pour commencer nous allons lancer 2 conteneur "ssh" avec des noms différents.

Se connecter aux deux et vérifier que le `hostname` correspond bien.

Il n'est pas possible de d'ouvrir deux conteneur avec le même port sur la machine, car il est impossible pour deux executable d'écouter sur le même port sur une seule machine.

Cependant on peut résoudre ce problème avec les `docker network`.

Créer 2 conteneurs "ssh", n'ouvrir les port que pour un seul.

Ajoutez ces deux conteneurs dans un network.

```sh
docker network create --driver bridge ${network_name}
docker network connect ${network_name} ${container_name_1}
docker network connect ${network_name} ${container_name_2}
```

Maintenant accédez au premier conteneur et à partir du premier accédez au deuxième.

Pour accéder au deuxième conteneur il faut utiliser son nom ou ID comme hostname dans la commande ssh.

```sh
ssh root@${container_name_2}
```

Essayez de se connecter au `container_name_1` à partir du `container_name_2` aussi.

Vérifiez bien le `hostname` à chaque étape.

#### Exercice 7

Une chose importante dans l'utilisation de `docker` et des systèmes en général sont les variables d'environment.

La commande `printenv` affiche tout l'environment courant.

La commande `export COUCOU="COUCOU"` change la variable d'environment `COUCOU` mais seulement pour la session en cours.

Vous pouvez ajouter la commande `printenv` dans le script `report.sh`.

Certains executable utilise la configuration donnée par les variables d'environments comme le `PORT` d'un serveur par exemple.

On peut également récupérer les variables d'environment avec `echo $COUCOU`.

De plus il est possible d'injecter un environment au lancement d'un conteneur docker.

```sh
docker run -itd -e COUCOU=COUCOU --name ${container_name} ${image_name}
```

Ou directement d'un fichier

```sh
echo "COUCOU=COUCOU">env
docker run -itd --env-file ./env --name ${container_name} ${image_name}
```

Vérifiez que dans les deux cas l'environment dans les conteneurs est bien valide.
