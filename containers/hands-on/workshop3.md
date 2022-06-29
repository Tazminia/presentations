# TP3 : Créer un conteneur

**Objectif:** Créer et configurer des images docker.

## Créer une image

Pour mettre à jour l'image `python:3.8.7-alpine3.12` afin de contenir `vim`, utiliser le `Dockerfile` suivant:

```docker
FROM python:3.8.7-alpine3.12 # Notre image de base
RUN apk add vim # Installer vim
```

On construit ensuite l'image comme suit:

```console
tjegham TP $ docker build -t "python-with-vim:3.8.7-alpine3.12" .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM python:3.8.7-alpine3.12
 ---> 64df5e2068e3
Step 2/2 : RUN apk add vim
 ---> Running in 0cfd46dca811
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/3) Installing xxd (8.2.0735-r0)
(2/3) Installing lua5.3-libs (5.3.5-r6)
(3/3) Installing vim (8.2.0735-r0)
Executing busybox-1.31.1-r19.trigger
OK: 40 MiB in 38 packages
Removing intermediate container 0cfd46dca811
 ---> c26bde9e8a1d
Successfully built c26bde9e8a1d
Successfully tagged python-with-vim:3.8.7-alpine3.12
tjegham TP $ docker images
REPOSITORY        TAG                IMAGE ID       CREATED          SIZE
python-with-vim   3.8.7-alpine3.12   c26bde9e8a1d   10 seconds ago   72.7MB
python            3.8.7-alpine3.12   64df5e2068e3   2 weeks ago      44.5MB
```

Remarquez les messages du type:

```console
 Step 1/2 : FROM python:3.8.7-alpine3.12
 ---> 64df5e2068e3 # executer
 Step 2/2 : RUN apk add vim
 ---> Running in 0cfd46dca811
 Removing intermediate container 0cfd46dca811
 ---> c26bde9e8a1d
 Successfully built c26bde9e8a1d
```

C'est la manifestation de la notions de couches:

1. On crée le tout premier conteneur (vide) `64df5e2068e3`, on éxecute l'instruction `FROM`
2. Le `FROM` réussit ce qui donne vie au conteneur `0cfd46dca811`
3. Dans le conteneur `0cfd46dca811`, on exécute l'instruction `RUN`
4. Le `RUN` réussit et donne vie au conteneur `c26bde9e8a1d`
5. Il n y a plus d'instructions à exécuter, le conteneur `c26bde9e8a1d` est donc notre conteneur final

## Lancer le nouveau conteneur

Vérifions que `vim` est bien présent dans notre nouvelle image. Lancer la commande:

```console
tjegham TP $ docker run --rm  -it python-with-vim:3.8.7-alpine3.12 /bin/sh
/ # vim -version
VIM - Vi IMproved 8.2 (2019 Dec 12, compiled May 15 2020 18:14:07)
Garbage after option argument: "-version"
More info with: "vim -h"
```

On voit bien que vim est présent. Par contre le script python `hello.py` n'existe pas.

```console
/ # python hello.py
python: can't open file 'hello.py': [Errno 2] No such file or directory
```

## Ajouter des fichiers

Dans le même dossier que le `Dockerfile`, créer un fichier `hello.py` avec le contenu suivant:

```python
import os

# Print message based on environment variable content
print("Hello from environment {} !".format(os.getenv('ENV_NAME', default = "DEFAULT_ENV")))
```

Pour ajouter le fichier dans l'image, on utilise l'instruction `COPY`. Dans le même dossier que le `Dockerfile`, créer un fichier `python-app.dockerfile` avec le contenu suivant:

```docker
FROM python-with-vim:3.8.7-alpine3.12
COPY hello.py .
```

Construire l'image comme suit:

```console
tjegham TP $ docker build -t python-app:3.8.7 -t docker-tp:py3.8-app -f python-app.dockerfile .
tjegham TP $ docker images
REPOSITORY        TAG                IMAGE ID       CREATED          SIZE
docker-tp         py3.8-app          bfebcca452fa   2 minutes ago    72.7MB
python-app        3.8.7              bfebcca452fa   2 minutes ago    72.7MB
```

Notez qu'il est possible de donner plusieurs tag à une seule image. Docker construit alors une seule image qu'on peut appeler par deux tags différents.

Lancer le conteneur et vérifier l'output pour les deux images:

```console
tjegham TP $ docker run --rm docker-tp:py3.8-app python hello.py
Hello from environment DEFAULT_ENV !
tjegham TP $ docker run --rm python-app:3.8.7 python hello.py
Hello from environment DEFAULT_ENV !
```

## Configurer le conteneur

On peut paramétrer l'exécution d'un conteneur en lui passant des variables d'environnement. Par exemple:

```console
tjegham TP $ docker run --rm -e ENV_NAME="STAGING" docker-tp:py3.8-app python hello.py
Hello from environment STAGING !
tjegham TP $ docker run --rm -e ENV_NAME="PRODUCTION" docker-tp:py3.8-app python hello.py
Hello from environment PRODUCTION !
```

La valeur par défaut de la variable d'environnement peut être également définie au niveau du dockerfile. Toujours dans le même dossier, créer le fichier `python-app-with-default.dockerfile` avec le contenu suivant:

```docker
FROM python-app:3.8.7
ENV ENV_NAME="DEVELOPMENT"
```

Construire l'image comme suit:

```console
tjegham TP $ docker build -t docker-tp:py3.8-app -f python-app-with-default.dockerfile .
Sending build context to Docker daemon   5.12kB
Step 1/2 : FROM python-app:3.8.7
 ---> bfebcca452fa
Step 2/2 : ENV ENV_NAME="DEVELOPMENT"
 ---> Running in 64060fa5c938
Removing intermediate container 64060fa5c938
 ---> baa97dffcce9
Successfully built baa97dffcce9
Successfully tagged docker-tp:py3.8-app
```

Notez qu'à la base on avait tag la même image par `docker-tp:py3.8-app` et `python-app:3.8.7`. Ce n'est plus le cas désormais, puisque le nouveau tag `docker-tp:py3.8-app` correspond à `python-app:3.8.7` dans lequel on a mis à jour la valeur par défaut de la variable d'environnement `ENV_NAME` à `DEVELOPMENT`.

Vérifier le comportement en lançant les deux images:

```console
tjegham TP $ docker run --rm docker-tp:py3.8-app python hello.py
Hello from environment DEVELOPMENT !
tjegham TP $ docker run --rm python-app:3.8.7 python hello.py
Hello from environment DEFAULT_ENV !
```
