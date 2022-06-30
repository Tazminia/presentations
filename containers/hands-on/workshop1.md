# TP1 : Run a container

**Goal:** Start a container and get used to basic docker commands.

## Commands

The following commands will be used in this workshop:

- run
- image
- ps
- log
- rm

## Run a container

In a terminal, run:

```console
tjegham ~ $ docker run python:3.8.7-alpine3.12 python --version
Python 3.8.7
```

Python 3.8.7 was started & it ran the command `python --version` which returned `Python 3.8.7`.

The first time you run the previous command, expect the following output:

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

## List local images

In a terminal, run:

```console
tjegham ~ $ docker image ls
REPOSITORY      TAG                IMAGE ID       CREATED       SIZE
python          3.8.7-alpine3.12   64df5e2068e3   2 weeks ago   44.5MB
```

The command lists images locally available. Since the previous command used `python:3.8.7-alpine3.12` the image was downloaded and stored locally.

## List containers

In a terminal, run:

```console
tjegham ~ $ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                    NAMES
```

The commands lists containers that are running. 

Although we ran a command to launch a container, it does not appear in the list.
This is because, containers are `volatile`, it means that they dissapear once they have achieved their purpose.

For instance, our container's purpose is to run `python --version`, once the version is printed, the dissapears.

To view all containers present on the machine:

```console
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                      PORTS                    NAMES
bc35d2378254   python:3.8.7-alpine3.12   "python --version"       13 minutes ago   Exited (0) 13 minutes ago                            tender_brahmagupta
```


Here we see that container `bc35d2378254` was ran based on image `python:3.8.7-alpine3.12`.

The container executed command `python --version` which ended with a success code `Exited (0)`.

## View container logs

In a terminal, run:

```console
tjegham ~ $ docker logs bc35d2378254
Python 3.8.7
```

We just viewed the logs produced by container `bc35d2378254`.

## Remove a container

In a terminal, run:

```console
tjegham ~ $ docker rm bc35d2378254
bc35d2378254
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                      PORTS                    NAMES
```

The command validates that the container was removed by sending back the container id `bc35d2378254`.

When expecting the container to be automatically removed once its work is finished, run:

```console
tjegham ~ $ docker run --rm python:3.8.7-alpine3.12 python --version
Python 3.8.7
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                      PORTS                    NAMES
```
