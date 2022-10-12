# Namespaces

## Requirements

* a kubernetes cluster
* kubectl CLI

This workshop assumes that `k` alias for `kubectl` is setup. If you do not want to setup the alias, replace `k` in the commands by `kubectl`.

## List namespaces

```console
$ k get ns
NAME              STATUS   AGE
default           Active   6d21h
kube-node-lease   Active   6d21h
kube-public       Active   6d21h
kube-system       Active   6d21h
```

The output shows:
    * default: an automatically created namespace by kubernetes provided as the default namespace.
    * 3 other system namespaces that run kubernetes internal workloads. 
        * Examples of internal workloads include: 
            * dns
            * proxy
            * controller-manager

## Create namespaces

```console
$ k create namespace ns-1
namespace/ns-1 created
$ k create ns ns-2
namespace/ns-2 created
```

## Create a pod in a namespace

Namespace provides context for kubernetes objects such as pods.

```console
$ k run --image=nginx:stable nginx-ns-1 -n ns-1
pod/nginx-ns-1 created
```

## View pods in a single namespace

```console
$ k get pods
No resources found in default namespace.
$ k get pods --namespace=kube-system
NAME                                     READY   STATUS    RESTARTS         AGE
coredns-95db45d46-p5h6c                  1/1     Running   1 (2d4h ago)     6d21h
etcd-docker-desktop                      1/1     Running   1 (2d4h ago)     6d21h
kube-apiserver-docker-desktop            1/1     Running   1 (2d4h ago)     6d21h
kube-controller-manager-docker-desktop   1/1     Running   1 (2d4h ago)     6d21h
kube-proxy-5kpp4                         1/1     Running   1 (2d4h ago)     6d21h
kube-scheduler-docker-desktop            1/1     Running   1 (2d4h ago)     6d21h
# ...
$ k get pod nginx-ns-1 -n ns-1
NAME         READY   STATUS    RESTARTS   AGE
nginx-ns-1   1/1     Running   0          57s
$ k get pod pod/nginx-ns-1 -n default
Error from server (NotFound): pods "nginx-ns-1" not found
```

## View pods in all namespaces

Objects can be listed in all namespaces using `--all-namespaces` or `-A` options.

```console
$ k get pods -A
NAMESPACE     NAME                                     READY   STATUS    RESTARTS        AGE
kube-system   coredns-95db45d46-p5h6c                  1/1     Running   1 (2d4h ago)    6d22h
kube-system   etcd-docker-desktop                      1/1     Running   1 (2d4h ago)    6d22h
kube-system   kube-apiserver-docker-desktop            1/1     Running   1 (2d4h ago)    6d22h
#...
ns-1          nginx-ns-1                               1/1     Running   0               76s
```

## Navigate namespaces

To easily switch between namespaces, use `kubens` command.

```console
$ kubens ns-1
Active namespace is "ns-1".
```

To install `kubens` on MacOs

```console
$ brew install kubectx
```