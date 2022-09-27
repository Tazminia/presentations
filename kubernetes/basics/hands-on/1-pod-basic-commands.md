# Workshop 1: Kubernetes Pods basics

## Requirements

* a kubernetes cluster
* kubectl CLI

This workshop assumes that `k` alias for `kubectl` is setup. If you do not want to setup the alias, replace `k` in the commands by `kubectl`.

## Create pods

There are several ways to create a pod. 

For now, **please**, follow along the commands, in the next section we will check if pods were created correctly.

### Run command

Create a pod named `nginx-run` with a single container named `nginx-run` based of the `nginx` image with the `stable` tag.

```console
$ k run --image=nginx:stable nginx-run
pod/nginx-run created # This is an output line
# Equivalent command in docker would have been
# docker run -d --name=nginx-run nginx:stable
```

### Yaml file

Create a pod named `nginx-from-yaml` with a single container named `nginx-from-yaml` based of the `nginx` image with the `stable` tag.

Make sure to be in the [hands-on](../hands-on/) folder.

```console
$ k apply -f yaml-samples/pod-sample.yaml
pod/nginx-from-yaml created # This is an output line
```

### One liner

Create a pod named `nginx-oneliner` with a single container named `nginx-oneliner` based of the `nginx` image with the `stable` tag.

```console
$ cat <<EOF | k apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-oneliner
spec:
  containers:
  - image: nginx:stable
    name: nginx-oneliner
EOF
pod/nginx-oneliner created # This is an output line
```

## List pods

To list pods:

```console
$ k get pods
k get pods
NAME              READY   STATUS    RESTARTS   AGE
nginx-from-yaml   1/1     Running   0          5m1s
nginx-oneliner    1/1     Running   0          2m16s
nginx-run         1/1     Running   0          12m
```

Each output line shows:

* NAME (column 1): the name of the pod
* READY (column 2): format is `r/t`
    * r: container that are ready
    * t: total number of containers
* STATUS (column 3): status of the pod (ContainerCreating, Running, CrashLoopBackOff, etc)
* RESTARTS (column 4): number of restarts since creation
* AGE (column 5): time since creation

## Get information on a specific pod

### Describe a pod

To get information on a pod:

```console
$ k describe pod/nginx-run
Name:         nginx-run
# ... ignore for now
Status:       Running # status of the pod
# ... ignore for now
Containers:
  nginx-run:
    # ... ignore for now
    Image:          nginx:stable
    # ... ignore for now
    State:          Running # status of the container
      Started:      Tue, 27 Sep 2022 16:21:54 +0200
    Ready:          True
    Restart Count:  0
    # ... ignore for now
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
# ... ignore for now
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  27m   default-scheduler  Successfully assigned default/nginx-run to docker-desktop
  Normal  Pulled     27m   kubelet            Container image "nginx:stable" already present on machine
  Normal  Created    27m   kubelet            Created container nginx-run
  Normal  Started    27m   kubelet            Started container nginx-run
```

The event section states:

* `default-scheduler` which is the kubernetes cluster scheduler assigned the pod to `docker-desktop` node which is the only node of the cluster.
* Once assigned to the node, it is `kubelet` that is now responsible for the pod:
    * docker image `nginx:stable` was not pulled because it was already on the machine.
    * container was created on the machine
    * container health check passed and was marked ready for use

### Get pod yaml manifest

To get yaml manifest for a pod:

```console
$ k get pod/nginx-run -o=yaml
apiVersion: v1
kind: Pod
metadata:
  # ... ignore for now
  labels:
    run: nginx-run
  name: nginx-run
    # ... ignore for now
spec:
  containers:
  - image: nginx:stable
    imagePullPolicy: IfNotPresent
    name: nginx-run
    # ... ignore for now
```

It is possible to store this manifest in a file and use it later to create the same pod:

```console
$ k get pod/nginx-run -o=yaml > /tmp/pod-nginx-run.yaml # to store the pod
$ k apply -f /tmp/pod-nginx-run.yaml # To create the pod
```

## Get pod logs

To view logs of the pod:

```console
$ k logs nginx-run
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/09/27 14:21:54 [notice] 1#1: using the "epoll" event method
2022/09/27 14:21:54 [notice] 1#1: nginx/1.22.0
2022/09/27 14:21:54 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/09/27 14:21:54 [notice] 1#1: OS: Linux 5.10.124-linuxkit
2022/09/27 14:21:54 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/09/27 14:21:54 [notice] 1#1: start worker processes
```

## Destroy a pod

To destroy a pod:

```console
$ k delete pod/nginx-run
pod "nginx-run" deleted
$ k get pod/nginx-run
Error from server (NotFound): pods "nginx-run" not found
```

## List all pods

To list all pods:

```console
$ k get pods
NAME              READY   STATUS    RESTARTS   AGE
nginx-from-yaml   1/1     Running   0          78m
nginx-oneliner    1/1     Running   0          75m
$ k get pod
NAME              READY   STATUS    RESTARTS   AGE
nginx-from-yaml   1/1     Running   0          78m
nginx-oneliner    1/1     Running   0          75m
$ k get po
NAME              READY   STATUS    RESTARTS   AGE
nginx-from-yaml   1/1     Running   0          78m
nginx-oneliner    1/1     Running   0          75m
```

All of the above commands are equivalent



