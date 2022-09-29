# Pods vs deployments

## Requirements

* a kubernetes cluster
* kubectl CLI

This workshop assumes that `k` alias for `kubectl` is setup. If you do not want to setup the alias, replace `k` in the commands by `kubectl`.

## Create pod & deployment

Create a pod named `nginx-simple-pod` with a single container named `nginx-simple-pod` based of the `nginx` image with the `stable` tag.

```console
$ k apply -f yaml-samples/pod-sample.yaml
pod/nginx-simple-pod created
```

Create a deployment called `nginx-deployment` with a single container named `nginx-pod` based of the `nginx` image with the `stable` tag.

```console
$ k apply -f yaml-samples/deployment-sample.yaml
deployment.apps/nginx-deployment created
```

Now to list the created pods:

```console
$ k get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bb55cd947-kc8b9   1/1     Running   0          4s
nginx-simple-pod                    1/1     Running   0          4m25s
```

* Pods were created based on the yaml manifests
* Pod created based on the deployment has the deployment name as a prefix

## Delete pods

To delete the pods:

```console
$ k delete pod nginx-deployment-6bb55cd947-kc8b9 nginx-simple-pod
pod "nginx-deployment-6bb55cd947-kc8b9" deleted
pod "nginx-simple-pod" deleted
$ k get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bb55cd947-5jrnj   1/1     Running   0          6s
```

Note that the pod from the deployment was automatically created again. This is the difference between a deployment and a pod.

When a deployment is defined with an expected number of pods, kubernetes becomes responsible of maintaining that number of pods at all times. 

## Scale pods

Using deployments allows to handle the scaling of pods.

To **scale out** the `nginx-deployment` to 3 pods:

```console
$ k scale --replicas=3 deployment/nginx-deployment
deployment.apps/nginx-deployment scaled
$ k get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bb55cd947-5jrnj   1/1     Running   0          8m56s
nginx-deployment-6bb55cd947-8hkqk   1/1     Running   0          3s
nginx-deployment-6bb55cd947-bvmqt   1/1     Running   0          3s
```

To **scale in** the `nginx-deployment` to 2 pods:

```console
$ k scale --replicas=2 deployment/nginx-deployment
deployment.apps/nginx-deployment scaled
$ k get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bb55cd947-5jrnj   1/1     Running   0          10m
nginx-deployment-6bb55cd947-bvmqt   1/1     Running   0          105s
```

Delete the pods & kubernetes would create new ones to replace them.

```console
$ k delete pod nginx-deployment-6bb55cd947-5jrnj nginx-deployment-6bb55cd947-bvmqt
pod "nginx-deployment-6bb55cd947-5jrnj" deleted
pod "nginx-deployment-6bb55cd947-bvmqt" deleted
$ k get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bb55cd947-6vqlr   1/1     Running   0          14s
nginx-deployment-6bb55cd947-9r9wd   1/1     Running   0          14s
```

Delete only one pod & kubernetes would create a single one to replace it.

```console
$ k delete pod nginx-deployment-6bb55cd947-6vqlr
pod "nginx-deployment-6bb55cd947-6vqlr" deleted
$ k get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bb55cd947-5h7tv   1/1     Running   0          13s
nginx-deployment-6bb55cd947-9r9wd   1/1     Running   0          92s
```

Note that applying the deployment yaml file will scale back the pods to a single pod as specified by the replicas field. It will delete one of the 2 pods that are currently present.

```console
$ k apply -f yaml-samples/deployment-sample.yaml
deployment.apps/nginx-deployment configured
k get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6bb55cd947-5h7tv   1/1     Running   0          2m27s
```
