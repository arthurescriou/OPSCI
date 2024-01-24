# TME 2 OpsCI

L'objectif de ce TME est d'apprendre à utiliser les fonctionnalités basiques de docker et de se familiariser avec l'utilisation de machine distante.

## Partie 1 : Se connecter à une machine distante

### Exercice 1

Pour commencer on veut se connecter avec une machine distante en utilisant `ssh`.

```sh
ssh root@${URL_SSH} -v
```

Une fois sur cette machine distante on veut lancer le script créé au TME précédent. (Attention vous allez être plusieurs sur la même machine pensez bien à créer vos propres dossiers).

Plusieurs solutions :

- utiliser le protocole `ftp` pour transférer le fichier de votre machine vers la machine distante

```sh
scp /path/to/script/report.sh root@${URL_SSH}:/distant/path/to/script/your/name/report.sh
```

- faire un `git pull` de votre dépôt git depuis la machine distante
  (attention il faut avoir créé un dépôt sur un fournisseur git en ligne cf TME1 exercice 6)

- copier coller le contenu du script avec un éditeur de fichier en terminal (vi, vim, emacs, nano)

### Exercice 2

Ajouter sa clé ssh pour s'authentifier sans mot de passe (`ssh-keygen`)

### Exercice 3 (droits d'accès)

La propriété des fichiers et des répertoires est modifiable par la commande `chown`, tandis que
les droits d'accès aux fichiers et répertoires sont modifiables par la commande `chmod`.
Ces droits sont représentés en machine par 9 (neuf) bits:

- trois bits pour l'utilisateur propriétaire du fichier / répertoire (owner).

- trois pour le groupe dont appartient l'utilisateur propriétaire (group).

- les trois derniers bits pour n'importe quel utilisateur (public).

L'usage commun regroupe chaque triplet en chiffre décimal. Voici quelques exemples:

- 051 signifie que l'utilisateur propriétaire n'a aucun droit (ni en lecture, ni en écriture, ni en exécution); que le groupe a le droit de lecture et d'exécution (5 = 2^2+2^0) mais pas celui d'écriture; qu'un utilisateur publique peut uniquement exécuter le fichier / répertoire.

- 421 signifie que l'utilisateur a le droit de lire; que le groupe a le droit d'écrire; qu'un utilisateur publique a le droit d'exécution.

- les droits les plus courants pour un fichier sont: 644, 600, 640.

- les droits les plus courants pour un répertoire sont: 711, 700, 710.

Dans la suite de l'exercice, nous allons tenter de lire le fichier `report.sh` d'un autre étudiant dans les machines de la PPTI. Déterminer l'endroit où le `home` de votre voisin est monté dans le système de fichiers:

```sh
ls -l /users/Etu?/ | grep ${NUM_ETU_DU_VOISIN}
```

Déterminer les droits d'accès à ce `home`, si ceux-ci sont XX7, XX5, XX1, accéder au répertoire et chercher à lire le fichier `OPSCI/systeme-report/report.sh` ainsi que les fichiers `OPSCI/systeme-report/*.info`.

Comment interdire ce type d'intrusion?

## Partie 2 : Docker

### Exercice 4

On veut maintenant lancer un conteneur docker :

```sh
export container_name='ubuntu'
docker run -itd --name ${container_name} ubuntu
```

Maintenant que le conteneur est lancé il est possible de se connecter dessus avec docker.

```sh
docker exec -it ${container_name} bash
```

De la même manière que pour Exercice 1 lancez le script `report.sh` dans le conteneur.

### Exercice 5

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

Et on peut la lancer comme dans Exercice 3

```sh
docker run -itd --name ${container_name} ${image_name}
```

Un nouveau conteneur acceptant le `SSH` a été créé.
Cependant le réseau docker ne permet pas d'accéder au conteneur sur le port ssh : 22.

Pour se connecter en `SSH` il faut changer la configuration de lancement du conteneur. On va donc commencer par l'arrêter et le supprimer.

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

### Exercice 6

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

- ou tout autre exemple similaire.

Une fois le Dockerfile écrit exécuter `docker build` comme dans Exercice 4.
Lancer le conteneur en configurant les ports correctement.

## Partie 3 : Le réseau en docker

Dans cette partie on veut faire fonctionner plusieurs conteneurs ensemble.
Grâce au script `report.sh` on sait récupérer des informations sur le système courant.
Mais pour cette partie on est surtout intéressé par la commande `hostname`.
Lorsqu'on est dans un conteneur docker le hostname correspond à l'`ID` du conteneur.

### Exercice 7

Nous voulons communiquer entre deux conteneurs.
Pour commencer nous allons lancer 2 conteneurs "ssh" avec des noms différents.
Se connecter aux deux et vérifier que le `hostname` correspond bien.

Il n'est pas possible d'ouvrir deux conteneurs avec le même port sur la machine, car il est impossible pour deux exécutables d'écouter sur le même port sur une seule machine.
Cependant on peut résoudre ce problème avec les `docker network`.

Créer 2 conteneurs "ssh", n'ouvrir les ports que pour un seul.
Ajouter ces deux conteneurs dans un réseau.

```sh
docker network create --driver bridge ${network_name}
docker network connect ${network_name} ${container_name_1}
docker network connect ${network_name} ${container_name_2}
```

Accédons maintenant au premier conteneur et à partir du premier accéder au deuxième.
Pour accéder au deuxième conteneur il faut utiliser son nom ou ID comme hostname dans la commande `ssh`.

```sh
ssh root@${container_name_2}
```

Essayer de se connecter au `container_name_1` à partir du `container_name_2` aussi.
Vérifier bien le `hostname` à chaque étape.

### Exercice 8

Une chose importante dans l'utilisation de `docker` et des systèmes en général sont les variables d'environnement:

- la commande `printenv` affiche tout l'environnement courant.

- la commande `export COUCOU="COUCOU"` change la variable d'environnement `COUCOU` mais seulement pour la session en cours.

Ajouter la commande `printenv` dans le script `report.sh`.

Certains exécutables utilisent la configuration donnée par les variables d'environnements comme le `PORT` d'un serveur par exemple.
On peut également récupérer les variables d'environnement avec `echo $COUCOU`.
De plus il est possible d'injecter un environnement au lancement d'un conteneur docker:

```sh
docker run -itd -e COUCOU=COUCOU --name ${container_name} ${image_name}
```

Ou directement d'un fichier:

```sh
echo "COUCOU=COUCOU">env
docker run -itd --env-file ./env --name ${container_name} ${image_name}
```

Vérifier que dans les deux cas l'environnement dans les conteneurs est bien valide.
