# Workshop 2 : Inside a container

**Goal:** Run commands inside a container.

## Get a shell session

In a terminal, run:

```console
tjegham ~ $ docker run -it python:3.8.7-alpine3.12 /bin/sh
/ # whoami
root
/ # exit
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED             STATUS                     PORTS                    NAMES
bbd40f6809f8   python:3.8.7-alpine3.12   "/bin/sh"                14 seconds ago      Exited (0) 3 seconds ago                            peaceful_wozniak
```

With the previous command we started a shell session **inside** the container. 
Notice that the promet changed from `$` to `#`. 

Make sure you use the `-it` options, otherwise it will not work.

## Install a package

In a terminal, run:

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

With the previous command we acquired a shell session inside the container and installed `vim`.

Now, create and run a simple helloworld python script inside the container:

```console
/ # echo 'print("Hello world !")' > hello.py
/ # python hello.py
/ # exit
```

Let us read the logs of the container:

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
(1/3) Installing xxd (8.2.4836-r0)
(2/3) Installing lua5.3-libs (5.3.5-r6)
(3/3) Installing vim (8.2.4836-r0)
 55% █████████████████████████████████████████████████████████████████████████████
Executing busybox-1.31.1-r19.trigger
OK: 42 MiB in 38 packages
/ # echo 'print("Hello world !")' > hello.py
/ # python hello.py
Hello world !
/ # exit
```

## Immutable images

In a new terminal, run:

```console
tjegham ~ $ docker run --rm  -it python:3.8.7-alpine3.12 /bin/sh
/ # vim
/bin/sh: vim: not found
/ # exit
```

Notice that the new container we created does not have `vim` installed. This is because containers are **isolated**, meaning, an action in a container only affects the container itself. 
