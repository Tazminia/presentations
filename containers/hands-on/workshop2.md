# TP2 : Travailler dans un conteneur

**Objectif:** Exécuter des commandes à l'intérieur d'un conteneur.

## Lancer un shell

Il peut être utile de lancer un shell dans un conteneur afin de pouvoir l'explorer et l'exploiter. Pour lancer un shell, utilisez la commande qui suit:

```console
tjegham ~ $ docker run -it python:3.8.7-alpine3.12 /bin/sh
/ # whoami
root
/ # exit
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED             STATUS                     PORTS                    NAMES
bbd40f6809f8   python:3.8.7-alpine3.12   "/bin/sh"                14 seconds ago      Exited (0) 3 seconds ago                            peaceful_wozniak
```

Remarquez la pormpt qui change de `$` à `#`. Egalement, le user est passé de `tjegham` à `root`. 

Notez les option `-it` qui nous permettent de donner des inputs au conteneur (i) et de s'attacher au terminal (t). Sans ces options, on est incapable de nous attacher au conteneur, ni d'y exécuter des commandes.

## Installer un package

Une fois dans le conteneur, on est capable de l'utiliser comme si c'était une VM classique. Pour installer vim dans le conteneur, on peut procéder comme suit:

```console
tjegham ~ $ docker run -it python:3.8.7-alpine3.12 /bin/sh
/ # vim
/bin/sh: vim: not found
/ # apk add vim
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/3) Installing xxd (8.2.0735-r0)
(2/3) Installing lua5.3-libs (5.3.5-r6)
(3/3) Installing vim (8.2.0735-r0)
Executing busybox-1.31.1-r19.trigger
OK: 40 MiB in 38 packages
```

On peut créer un script python et le lancer à l'intérieur du conteneur. Utilisez `vim` pour créer le script `hello.py` suivant:

```python
print("Hello world !")
```

Pour lancer le script, toujours dans le conteneur lancer:

```console
/ # vim hello.py
/ # python hello.py
Hello world !
/ # exit
```

Essayons de récupérer les logs de notre conteneur:

```console
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED             STATUS                     PORTS                    NAMES
ce737aff81a2   python:3.8.7-alpine3.12   "/bin/sh"                5 minutes ago       Exited (0) 3 minutes ago                            peaceful_galileo
tjegham ~ $ docker logs ce737aff81a2
/ # vim
/bin/sh: vim: not found
/ # apk add vim
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/3) Installing xxd (8.2.0735-r0)
(2/3) Installing lua5.3-libs (5.3.5-r6)
(3/3) Installing vim (8.2.0735-r0)
OK: 40 MiB in 38 packages
/ # vim hello.py
/ # python hello.py
Hello world !
/ # exit
```

## Explorer l'image

N'oubliez pas, le caractère `isolé` d'un conteneur indique que tout ce qui passe dans le conteneur n'affecte que le conteneur lui même. En effet, essayons de relancer un conteneur à partir de l'image de base `python:3.8.7-alpine3.12` et voyons si vim existe dedans:

```console
tjegham ~ $ docker run --rm  -it python:3.8.7-alpine3.12 /bin/sh
/ # vim
/bin/sh: vim: not found
/ # exit
```
