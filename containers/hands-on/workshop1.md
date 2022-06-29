# TP1 : Lancer un conteneur

**Objectif:** Lancer un conteneur et de se familiariser avec les commandes de base de docker.

## Commandes abordées

Les commandes à voir dans ce TP:

- run
- image
- ps
- log
- rm

## Lancer un conteneur

Pour lancer un conteneur, il suffit d'utiliser la commande `run` comme suit:

```console
tjegham ~ $ docker run python:3.8.7-alpine3.12 python --version
```

Cette commande indique qu'on lance la commande `python --version` à l'intérieur d'un conteneur qu'on crée depuis l'image `python` ayant la version (tag) `3.8.7-alpine3.12`.

Si vous n'avez jamais utilisé cette image, votre sortie ressemblera à ce qui suit:

```console
Unable to find image 'python:3.8.7-alpine3.12' locally
3.8.7-alpine3.12: Pulling from library/python
801bfaa63ef2: Already exists
7678dd7631a2: Pull complete
4c6139ab40d8: Pull complete
ff5ef8cd8062: Pull complete
73dee1f700a1: Pull complete
Digest: sha256:60a24db20ad0121b3440681ded50a75b762ceb9cb0855847a4b25d291e9de8c2
Status: Downloaded newer image for python:3.8.7-alpine3.12
Python 3.8.7
```

La dernière ligne de l'output indique que notre `python --version` a bien fonctionné et qu'elle retourne `Python 3.8.7`.

## Lister les images

Si on souhaite lister les images docker disponibles sur notre machine on lance la commande suivante:

```console
tjegham ~ $ docker image ls
REPOSITORY      TAG                IMAGE ID       CREATED       SIZE
python          3.8.7-alpine3.12   64df5e2068e3   2 weeks ago   44.5MB
```

L'output nous indique la présence de l'image qu'on a téléchargé dans la première étape. On voit que l'image vient du repository `python`, que son tag est `3.8.7-alpine3.12` et qu'elle a une taille de `44.5MB`.

D'ailleurs, si on relance la commande, de run du début du TP, on remarquera que l'output est différent:

```console
tjegham ~ $ docker run python:3.8.7-alpine3.12 python --version
Python 3.8.7
```

En effet le log du téléchargement disparaît.

## Lister les conteneur

Pour voir les conteneur en cours d'exécution, on lance la commande `ps` comme suit:

```console
tjegham ~ $ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                    NAMES
```

La commande ne retourne rien et pourtant on a bien lancé notre conteneur python. En effet, notre conteneur étant `volatile` s'est éteint dès qu'il a finit d'exécuter sa tâche `python --version`. Pour voir tous les conteneurs, y compris ceux qui sont éteints, lancer la commande suivantes:

```console
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                      PORTS                    NAMES
bc35d2378254   python:3.8.7-alpine3.12   "python --version"       13 minutes ago   Exited (0) 13 minutes ago                            tender_brahmagupta
```

L'output nous indique que le conteneur `bc35d2378254` a été lancé en se basant sur l'image `python:3.8.7-alpine3.12`.

Le conteneur a exécuté la commande `python --version` et s'est exécuté avec succès puisque le code de sortie était 0 `Exited (0)`.

Comme on a pas donné de nom à notre conteneur, docker lui a généré un nom aléattoire `tender_brahmagupta`.

## Explorer le log

Pour voir les logs d'un conteneur, il suffit de lancer la commande `logs`, comme suit:

```console
tjegham ~ $ docker logs bc35d2378254
Python 3.8.7
```

## Supprimer un conteneur

Une fois qu'on a plus besoin du conteneur, on peut le supprimer en utilisant la commande `rm` comme suit:

```console
tjegham ~ $ docker rm bc35d2378254
bc35d2378254
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                      PORTS                    NAMES
```

Docker nous retourne l'id du conteneur pour valider la suppression. Relancer le listing de conteneur, retourne désormais une liste vide.

Il est possible également, d'indiquer qu'il faut supprimer le conteneur dès qu'il finit de se lancer en utilisant l'option `--rm` lors de la commande run, comme suit:

```console
tjegham ~ $ docker run --rm python:3.8.7-alpine3.12 python --version
Python 3.8.7
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                      PORTS                    NAMES
```
