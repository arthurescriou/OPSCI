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

Ajouter sa clé ssh.

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
