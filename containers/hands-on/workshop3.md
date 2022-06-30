# Workshop 3 : Build a container

**Goal:** Build docker images and customize containers.

## Build an image

Crate a file named `Dockerfile` with the content:

```docker
FROM python:3.8.7-alpine3.12 # Base image to use
RUN apk add vim # Install vim
```

In a terminal, run:

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

Notice messages like:

```console
 Step 1/2 : FROM python:3.8.7-alpine3.12
 ---> 64df5e2068e3
 Step 2/2 : RUN apk add vim
 ---> Running in 0cfd46dca811
 Removing intermediate container 0cfd46dca811
 ---> c26bde9e8a1d
 Successfully built c26bde9e8a1d
```

This is because docker uses layers to build the image in an incremental manner:

1. Create the base empty image `64df5e2068e3` and run `FROM` instruction
2. `FROM` instruction succeeds and we now get a new image with id `0cfd46dca811`
3. On image `0cfd46dca811` run instruction `RUN`
4. `RUN` instruction succeeds and we now get a new image with id `c26bde9e8a1d`
5. No more instructions left to run, so, `c26bde9e8a1d` is our final image

## Run built container

In a terminal, run:

```console
tjegham TP $ docker run --rm  -it python-with-vim:3.8.7-alpine3.12 /bin/sh
/ # vim -version
VIM - Vi IMproved 8.2 (2019 Dec 12, compiled May 15 2020 18:14:07)
Garbage after option argument: "-version"
More info with: "vim -h"
/ # python hello.py
python: can't open file 'hello.py': [Errno 2] No such file or directory
```

We can see that `vim` exists in the container we launched based on our newly created image.

## Add files

In the same folder as `Dockerfile`, create `hello.py` file with the content:

```python
import os

# Print message based on environment variable content
print("Hello from environment {} !".format(os.getenv('ENV_NAME', default = "DEFAULT_ENV")))
```

In the same folder as `Dockerfile`, create `python-app.dockerfile` file with the content:

```docker
FROM python-with-vim:3.8.7-alpine3.12 # use base image having vim
COPY hello.py . # Copy local hello.py file to the container
```

In a terminal, run:

```console
tjegham TP $ docker build -t python-app:3.8.7 -t docker-tp:py3.8-app -f python-app.dockerfile .
tjegham TP $ docker images
REPOSITORY        TAG                IMAGE ID       CREATED          SIZE
docker-tp         py3.8-app          bfebcca452fa   2 minutes ago    72.7MB
python-app        3.8.7              bfebcca452fa   2 minutes ago    72.7MB
```

Notice that we provided multiple tags `-t`. Tags are identifiers for docker images and a single docker image can have multiple tags (just like a person can have a middle name).

In a terminal, run:

```console
tjegham TP $ docker run --rm docker-tp:py3.8-app python hello.py
Hello from environment DEFAULT_ENV !
tjegham TP $ docker run --rm python-app:3.8.7 python hello.py
Hello from environment DEFAULT_ENV !
```

## Customize container

In a terminal, run:

```console
tjegham TP $ docker run --rm -e ENV_NAME="STAGING" docker-tp:py3.8-app python hello.py
Hello from environment STAGING !
tjegham TP $ docker run --rm -e ENV_NAME="PRODUCTION" docker-tp:py3.8-app python hello.py
Hello from environment PRODUCTION !
```

As per the output, even though both containers use the same image, each command was able to override the `ENV_NAME` environment variable with a different value.

Create a file named `python-app-with-default.dockerfile` with the following contents:

```docker
FROM python-app:3.8.7
ENV ENV_NAME="DEVELOPMENT"
```

In a terminal, run:

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

Notice that we reused the tag `docker-tp:py3.8-app` that used to point to `python-app:3.8.7` without the `python-app:3.8.7` tag.
This means that `docker-tp:py3.8-app` and `python-app:3.8.7` no longer point to the same image. 

Actually, `docker-tp:py3.8-app` is now `python-app:3.8.7` in which we updated the default value of environment variable `ENV_NAME` to `DEVELOPMENT`.

To check that both images are not the same. In a terminal, run:

```console
tjegham TP $ docker run --rm docker-tp:py3.8-app python hello.py
Hello from environment DEVELOPMENT !
tjegham TP $ docker run --rm python-app:3.8.7 python hello.py
Hello from environment DEFAULT_ENV !
```
