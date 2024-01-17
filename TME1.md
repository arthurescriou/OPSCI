# TME 1 OpsCI

L'objectif de ce TME est de se familiariser avec l'environment système linux.
Les commandes utilisées lors de ce TME seront utiles pour la suite de l'UE.
Le rendu attendu est un script qui imprime toutes les informations système et réseau d'une machine, puis l'automatisation de ce script.

_NB: Il est important de garder des traces de ce que vous avez appris pour pouvoir le réutiliser dans les séances futures. (Prenez des notes! Papiers ou fichiers)._

## Partie 1 : Informations du système local

### Exercice 1

Pour commencer on veut récupérer plusieurs informations sur le système local :

Ressources materielles :

- taille du disque (`df -h`, qu'on peut comprendre en exécutant `man df`)
- taille de la mémoire (`cat /proc/meminfo`)
- nombre de coeurs du cpu (`cat /proc/cpuinfo`)
- détails du gpu (`lspci |grep VGA` donne l'ID de la carte puis `lspci -v -s ${ID}`)
  exemple de réponse :

```
01:00.0 VGA compatible controller: nVidia Corporation G98 [GeForce 8400 GS] (rev a1)
    Subsystem: nVidia Corporation Device 05cc
    Region 0: Memory at e4000000 (32-bit, non-prefetchable) [size=16M]
    Region 1: Memory at d0000000 (64-bit, prefetchable) [size=256M]
    Region 3: Memory at e2000000 (64-bit, non-prefetchable) [size=32M]
```

### Exercice 2

Ensuite on veut afficher le répertoir racine de notre session (avec les fichiers cachés et les droits d'accès). Exécuter et expliquer la différence entre :

```
cd ~ ; ls -al
```

```
cd ~ & ls -al
```

```
cd ~ && ls -al
```

Vous êtes maintenant à la racine de la session (votre `$HOME`, que vous pouvez voir avec `pwd`. Vérifier avec `echo $HOME`). On veut maintenant avoir des informations sur l'utilisation de l'espace disque de votre home.

Ressources utilisées :

- taille du répertoire courant (`du -hs`, attention ça peut prendre du temps)
- taille des fils du répertoire courant (`du -h --max-detph=1`)
- quota utilisé ( `quota`)
- nombre de fichiers dans la session (bonus: nombre par type d'extensions : utiliser `find ~/workspace -name *\.java| wc -l` par exemple)

### Exercice 3

On veut maintenant étudier les processus s'exécutant sur la machine. Pour ça on va utiliser la commande `ps` (`whoami` renvoie votre nom d'utilisateur `echo $USER` également).
Que renvoie la commande suivante ?

```sh
ps -aux | grep ^`whoami`
```

Pour tester la commande on veut lancer un processus qui s'exécute longtemps :

`sleep 1664 &`

Retrouver le pid du script dans la commande `ps` (première colonne du résultat). Une autre manière d'explorer les processus : `top -u $USER`

_NB: Si on a exécuté par erreur `sleep 1664` (sans le `&`) le terminal attend 1664 secondes avant de vous rendre la main. Une possibilité pour reprendre la main est de faire <kbd>Ctrl</kbd> + <kbd>Z</kbd> suivis de la commande `bg`_

Pour arrêter ce processus avant sa fin `kill -9 $PID`.

Pour continuer avec `top`, afficher les 10 processus utilisant le plus de mémoire et de CPU. (on peut utiliser `sort`)

## Partie 2 : Rapport du système local

Dans cette partie l'objectif et de reprendre les commandes de la partie d'avant pour les automatiser dans un script.

### Exercice 4

Créer un dossier pour la matière `OPSCI`, créer 3 fichiers vides à l'intérieur :

```sh
mkdir system-report
touch system-report/a.info
touch system-report/b.info
touch system-report/c.info
```

Remplir manuellement ces fichiers avec les informations optenues des Exercices 1, 2, 3.
Par exemple :

```sh
df -h | grep sda[0-9] >> a.info 2>> a.error
```

Ainsi de suite

### Exercice 5

Reprendre toutes les commandes de l'Exercice 4 et les sauvegarder dans un fichier `report.sh`.

Pour exécuter ce fichier changer ces droits d'exécutions.

`chmod 755 ./report.sh` ou `chmod a+x ./report.sh`

Vous pouvez maintenant lancer le script directement `./report.sh`.

_NB: Pour éditer votre exécutable nous recommandons un éditeur de texte en terminal (vi, vim, emacs, nano)_

### Exercice 6

Pour sauvegarder votre travail vous devez utiliser un gestionnaire de version : git.

Créer un dépôt dans votre dossier et ajouter votre code au dépôt.

**🔴 ATTENTION 🔴 : aucun fichiers system-report/\*.info de l'Exercice 4 ne doit figurer dans le dépôt (Les informations contenu dedans sont trop sensibles: créez un fichier _.gitignore_ en conséquence).**

_Vous pouvez également créer un projet sur un fournisseur distant (gitlab, github) si vous voulez travailler sur une autre machine._

### Exercice 7 (Devoir à la maison optionnel)

Une des caractéristiques majeure du controle de version est l'exposition et le partage de son code.

Se renseigner sur _Clean Code_ et _Craftmanship_, selectionner le top 10 des recommandations de votre choix.
Pour chacune des recommandations expliquer (entre 1 et 3 lignes max) de quoi il s'agit.

## Partie 3 : Pour aller plus loin

### Exercice 8

#### Exploration du LAN et WAN

Déterminer son adresse IP, les ports ouverts, ...

`ip addr`

`nmap localhost`

_NB: Il existe également les commandes `tshark`, `wireshark` et `ifconfig`._

Enrichir les fichiers du répertoir `system-report` avec ces nouvelles informations.

### Exercice 9

#### Planification récurente

Utiliser la commande `crontab` pour rendre récurente la commande de l'Exercice 5.

_Pensez à vérifier votre expression cron avant de la lancer (sur un simulateur de crontab en ligne par exemple)._
