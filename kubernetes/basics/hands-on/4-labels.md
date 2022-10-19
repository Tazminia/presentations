# Labels

## Requirements

* a kubernetes cluster
* kubectl CLI

This workshop assumes that `k` alias for `kubectl` is setup. If you do not want to setup the alias, replace `k` in the commands by `kubectl`.

## View pod labels

Create a pod

```console
$ k run --image=nginx:stable nginx-ns-1 -n ns-1
pod/nginx-ns-1 created
```

List labels

```console
$ k get pod --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-ns-1   1/1     Running   0          26m   run=nginx-ns-1
$ k get pod nginx-ns-1 -o jsonpath="{.metadata.labels}"
{"run":"nginx-ns-1"}
```

## Add labels

Create pod with one liner and add label `created-by` with value`one-liner`

```console
$ cat <<EOF | k apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-oneliner
  labels:
    created-by: one-liner
spec:
  containers:
  - image: nginx:stable
    name: nginx-oneliner
EOF
pod/nginx-oneliner created
```

List pods and show labels

```console
k get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE   LABELS
nginx-ns-1       1/1     Running   0          37m   run=nginx-ns-1
nginx-oneliner   1/1     Running   0          9s    created-by=one-liner
```

Add label `owner` with value `tazminia` to `nginx-ns-1` pod

```console
$ k label pod/nginx-ns-1 owner=tazminia
pod/nginx-ns-1 labeled
$ k get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
nginx-ns-1       1/1     Running   0          40m     owner=tazminia,run=nginx-ns-1
nginx-oneliner   1/1     Running   0          2m38s   created-by=one-liner
```

## Filters

List pods owned by `tazminia`

```console
$ k get pod --show-labels --selector owner=tazminia
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-ns-1   1/1     Running   0          43m   owner=tazminia,run=nginx-ns-1
```

List pods created by `one-liner`

```console
$ k get pod --show-labels --selector created-by=one-liner
NAME             READY   STATUS    RESTARTS   AGE     LABELS
nginx-oneliner   1/1     Running   0          7m23s   created-by=one-liner
```

List pods that are created by `one-liner` and not owned by `tazminia`

```console
$ k get pod --show-labels --selector created-by=one-liner,owner!=tazminia
NAME             READY   STATUS    RESTARTS   AGE     LABELS
nginx-oneliner   1/1     Running   0          7m23s   created-by=one-liner
```

## Remove label

```console
$ k get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE   LABELS
nginx-ns-1       1/1     Running   0          54m   owner=tazminia,run=nginx-ns-1
nginx-oneliner   1/1     Running   0          17m   created-by=one-liner
$ k label pod/nginx-ns-1 owner-
pod/nginx-ns-1 unlabeled
$ k get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE   LABELS
nginx-ns-1       1/1     Running   0          55m   run=nginx-ns-1
nginx-oneliner   1/1     Running   0          18m   created-by=one-liner
```
