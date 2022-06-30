# Workshop 4 : Ports and files

**Goal:** Exposing ports & manage persistent files.

## Run an nginx container

Nginx is a popular web-server/proxy. It can be ran as a docker container:

```console
tjegham ~ $ docker run --rm -d -p 30000:80 nginx:1.19.6-alpine
tjegham ~ $ curl --write-out 'HTTP CODE: %{http_code}\n' --silent --show-error --output /dev/null localhost:30000
HTTP CODE: 200
```

`curl` command returns status code 200. You can also check by going to http://localhost:30000 in your browser.

Stop the nginx container and run:

```console
tjegham ~ $ docker run --rm -d nginx:1.19.6-alpine
tjegham ~ $ curl --write-out 'HTTP CODE: %{http_code}\n' --silent --show-error --output /dev/null localhost:30000
HTTP CODE: 000
curl: (7) Failed to connect to localhost port 30000: Connection refused
```

The server is no longer available, this is due to the **isolated** characteristic of our container. Since we did not specify the `-p` option, the port is not exposed.

You can check that the nginx inside the container is still running. In a terminal run:

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

Notice that port 80 is listed under `PORTS` in the `docker ps` output. This means, the container exposes the port 80.

Let us add the `-p` option and check the ouput again. In a terminal, run:

```console
tjegham ~ $ docker run -d --rm -p 30000:80 nginx:1.19.6-alpine
5ae7a3bdcaabed511da4243f505bbbe5f0464978748a480e70dba243fcc60766
tjegham ~ $ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                    NAMES
5ae7a3bdcaab   nginx:1.19.6-alpine   "/docker-entrypoint.…"   3 seconds ago    Up 2 seconds    0.0.0.0:30000->80/tcp    hopeful_wu
```

Notice that `PORTS` now shows that port `30000` from the host is redirected to  port `80` from the container.

## Edit a file from the host machine

In a terminal, run:

```console
tjegham ~ $ docker run --rm -it ubuntu:21.04 /bin/bash
root@714e1692cff4:/#
```

In the container, run:

```console
root@714e1692cff4:/# echo "Hello from docker !" > greeting.txt
root@714e1692cff4:/# cat greeting.txt
Hello from docker !
```

Now, exit the container and run a new one as follows:

```console
root@714e1692cff4:/# exit
exit
tjegham ~ $ docker run --rm -it ubuntu:21.04 /bin/bash
root@2d03709a4121:/# cat greeting.txt
cat: greeting.txt: No such file or directory
```

Containers are **stateless** and **isolated**, so the file we created in the other container does not exist in the new one.

To avoid this, we can share files between the host and the container. In a terminal, run:

```console
tjegham ~ $ echo "Hello from host machine !" > greetings.txt
tjegham ~ $ cat greetings.txt
Hello from host machine !
tjegham ~ $ docker run -v $(pwd):/data --rm -it ubuntu:21.04 /bin/bash 
root@99b1f73e131a:/# cat /data/greetings.txt
Hello from host machine !
```

Notice that the file content is the same in the host machine and inside the container.

Edit the file inside the container to add `Welcome from ubuntu-21.04 !` as follows:

```console
root@47ff2eff5260:/# echo "Welcome from ubuntu-21.04 !" >> /data/greetings.txt
root@47ff2eff5260:/# cat /data/greetings.txt
Hello from host machine !
Welcome from ubuntu-21.04 !
```

Check that the file still exists on the host machine.
Validate that changes to the content of the file done in the container appear on the host machine.
The changes persist even after the container was removed.

```console
root@47ff2eff5260:/# exit
exit
tjegham ~ $ docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                    NAMES
tjegham ~ $ cat greetings.txt
Hello from host machine !
Welcome from ubuntu-21.04 !
```
