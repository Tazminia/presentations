# TP4 : Manipuler les ports et les fichiers

**Objectif:** Comprendre comment exposer les ports et modifier les fichiers de façon persistente.

## Lancer une image nginx

Nginx est un serveur web et proxy largement utilisé. Il est disponible sous forme d'une image docker et peut être lancé comme suit:

```console
tjegham ~ $ docker run --rm -d -p 30000:80 nginx:1.19.6-alpine
tjegham ~ $ curl --write-out 'HTTP CODE: %{http_code}\n' --silent --show-error --output /dev/null localhost:30000
HTTP CODE: 200
```

La commande curl retourne un status 200 pour dire que le site est bien joignable. Pour le valider, on peut également ouvrir un navigateur à l'adresse http://localhost:30000 qui affiche le message d'acceuil nginx.

Pour comprendre l'utilité du paramètre `-p`, lancer les commandes suivantes:

```console
tjegham ~ $ docker run --rm -d nginx:1.19.6-alpine
tjegham ~ $ curl --write-out 'HTTP CODE: %{http_code}\n' --silent --show-error --output /dev/null localhost:30000
HTTP CODE: 000
curl: (7) Failed to connect to localhost port 30000: Connection refused
```

Le conteneur nginx est en cours d'exécution, mais la commande curl ne fonctionne plus. En effet, le conteneur étant `isolé`, le port n'est exposé que si on l'indique explicitement. On peut d'ailleurs vérifier que le nginx, à l'intérieur du conteneur, fonctionne toujours correctement sur le port 80:

```console
tjegham ~ $ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                    NAMES
bbcdac864ba7   nginx:1.19.6-alpine   "/docker-entrypoint.…"   2 minutes ago    Up 2 minutes    80/tcp                   vibrant_wright
tjegham ~ $ docker exec -it bbcdac864ba7 /bin/sh
/ # apk add curl
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
OK: 25 MiB in 42 packages
/ # curl --write-out 'HTTP CODE: %{http_code}\n' --silent --show-error --output /dev/null localhost:80
HTTP CODE: 200
/ # exit
```

Remarquez que la ligne retournée par `docker ps` montre le port `80` sous la colonne `PORTS`. Ceci veut dire que le conteneur expose le port `80`.

Pour vérifier l'utilité du paramètre `-p`, on peut lancer un nginx qui expose son port `80` sur le port `30000` de la machine hôte et on observe le nouveau retour de la commande `docker ps`  comme suit:

```console
tjegham ~ $ docker run -d --rm -p 30000:80 nginx:1.19.6-alpine
5ae7a3bdcaabed511da4243f505bbbe5f0464978748a480e70dba243fcc60766
tjegham ~ $ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                    NAMES
5ae7a3bdcaab   nginx:1.19.6-alpine   "/docker-entrypoint.…"   3 seconds ago    Up 2 seconds    0.0.0.0:30000->80/tcp    hopeful_wu
```

Remarquez que la colonne `PORTS` affiche désormais que le port `30000` est redirigé vers le port `80` du conteneur.

## Modifier un fichier depuis la machine hôte

Créer un conteneur `ubuntu-21.04` comme suit:

```console
tjegham ~ $ docker run --rm -it ubuntu:21.04 /bin/bash
root@714e1692cff4:/#
```

Dans le conteneur, créer le fichier `greeting.txt` avec le contenu `Hello from docker !` et l'afficher:

```console
root@714e1692cff4:/# echo "Hello from docker !" > greeting.txt
root@714e1692cff4:/# cat greeting.txt
Hello from docker !
```

Sortir du contenaire, lancer un nouveau conteneur `ubuntu` et afficher le fichier `greeting.txt`:

```console
root@714e1692cff4:/# exit
exit
tjegham ~ $ docker run --rm -it ubuntu:21.04 /bin/bash
root@2d03709a4121:/# cat greeting.txt
cat: greeting.txt: No such file or directory
```

Etant donné la caractéristique `non persistent` de notre conteneur, le fichier précedemment créé n'existe plus. Pour contourner cet aspect, on va partager un fichier `greetings.txt` entre la machine hôte et notre conteneur:

```console
tjegham ~ $ echo "Hello from host machine !" > greetings.txt
tjegham ~ $ cat greetings.txt
Hello from host machine !
tjegham ~ $ docker run -v $(pwd):/data --rm -it ubuntu:21.04 /bin/bash 
root@99b1f73e131a:/# cat /data/greetings.txt
Hello from host machine !
```

Remarquez que le fichier à le même contenu dans le conteneur.

Modifier le fichier dans le conteneur pour rajouter la ligne `Welcome from ubuntu-21.04 !`:

```console
root@47ff2eff5260:/# echo "Welcome from ubuntu-21.04 !" >> /data/greetings.txt
root@47ff2eff5260:/# cat /data/greetings.txt
Hello from host machine !
Welcome from ubuntu-21.04 !
```

Valider que le contenu du fichier persiste après être sorti du conteneur et que le conteneur soit supprimé:

```console
root@47ff2eff5260:/# exit
exit
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                    NAMES
tjegham ~ $ cat greetings.txt
Hello from host machine !
Welcome from ubuntu-21.04 !
```
