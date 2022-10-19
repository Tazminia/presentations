# Labels

## Requirements

* a kubernetes cluster
* kubectl CLI

This workshop assumes that `k` alias for `kubectl` is setup. If you do not want to setup the alias, replace `k` in the commands by `kubectl`.

## Pods & IPs

Create a new namespace `lab-5` and activate it

```console
$ k create ns lab-5
namespace/lab-5 created
$ kubens lab-5
Context "docker-desktop" modified.
Active namespace is "lab-5".
```

Create `nginx-servers` deployment with a single pod

```console
$ k apply -f yaml-samples/nginx-servers.yaml
deployment.apps/nginx-servers created
$ k get po
NAME                             READY   STATUS    RESTARTS   AGE
nginx-servers-84788994c9-mwslh   1/1     Running   0          64s
```

Create `curl-client` pod and attach to it

```console
$ k run -it --rm --image curlimages/curl:7.85.0 curl-client -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ $
```

**Please note**, at this point splitting the terminal or opening a new one is highly recommended.

In the new terminal list pods running & note the `nginx-server` IP (`10.1.0.33`)

```console
$ k get pods -o=wide
NAME                             READY   STATUS    RESTARTS   AGE   IP          NODE             NOMINATED NODE   READINESS GATES
curl-client                      1/1     Running   0          20m   10.1.0.31   docker-desktop   <none>           <none>
nginx-servers-84788994c9-mwslh   1/1     Running   0          92s   10.1.0.33   docker-desktop   <none>           <none>
```

From within `curl-client` call the nginx server:

```console
/ $ curl --connect-timeout 5 -I 10.1.0.33
HTTP/1.1 200 OK
Server: nginx/1.22.0
Date: Wed, 19 Oct 2022 09:29:54 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Mon, 23 May 2022 23:59:19 GMT
Connection: keep-alive
ETag: "628c1fd7-267"
Accept-Ranges: bytes
```

From `terminal`, scale `nginx-servers` deployment to `0` pods & from within the `curl-client` call the nginx server again:

```console
$ k scale deployment/nginx-servers --replicas=0
deployment.apps/nginx-servers scaled
/ $ curl --connect-timeout 5 -I 10.1.0.33
curl: (7) Failed to connect to 10.1.0.33 port 80 after 3079 ms: Host is unreachable
```

From `terminal`, scale `nginx-servers` deployment to `1` pod, find its IP & from within the `curl-client` call the nginx server again:

```console
$ k scale deployment/nginx-servers --replicas=1
deployment.apps/nginx-servers scaled
$ k get po -o=wide
NAME                             READY   STATUS    RESTARTS   AGE   IP          NODE             NOMINATED NODE   READINESS GATES
curl-client                      1/1     Running   0          23m   10.1.0.31   docker-desktop   <none>           <none>
nginx-servers-84788994c9-v6656   1/1     Running   0          7s    10.1.0.34   docker-desktop   <none>           <none>
/ $ curl --connect-timeout 5 -I 10.1.0.34
HTTP/1.1 200 OK
Server: nginx/1.22.0
Date: Wed, 19 Oct 2022 09:47:16 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Mon, 23 May 2022 23:59:19 GMT
Connection: keep-alive
ETag: "628c1fd7-267"
Accept-Ranges: bytes
```

## Kubernetes services

Services can be created:

* Using `kubectl expose`
* Using a yaml file & `kubectl apply -f`

### Expose command

Create service using `expose` and note that `SELECTOR` is the same as the `nginx-servers` pod labels:

```console
$ kubectl expose deployment nginx-servers --port=80 --target-port=80
service/nginx-servers exposed
$ k get svc -o wide
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx-servers   ClusterIP   10.97.20.87   <none>        80/TCP    8s    app=nginx,lab=services
$ k get pods --show-labels
NAME                             READY   STATUS    RESTARTS   AGE     LABELS
curl-client                      1/1     Running   0          28m     run=curl-client
nginx-servers-84788994c9-v6656   1/1     Running   0          5m26s   app=nginx,lab=services,pod-template-hash=84788994c9
```

From within `curl-client` call the `nginx-servers` service:

```console
/ $ curl --connect-timeout 5 -I nginx-servers
HTTP/1.1 200 OK
Server: nginx/1.22.0
Date: Wed, 19 Oct 2022 09:54:54 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Mon, 23 May 2022 23:59:19 GMT
Connection: keep-alive
ETag: "628c1fd7-267"
Accept-Ranges: bytes
```

### Service from YAML

Delete `nginx-servers` service & create it using `Yaml` file:

```console
$ k delete svc/nginx-servers
service "nginx-servers" deleted
/ $ curl --connect-timeout 5 -I nginx-servers
curl: (6) Could not resolve host: nginx-servers
$ k apply -f yaml-samples/nginx-servers-service.yaml
service/nginx-servers created
/ $ curl --connect-timeout 5 -I nginx-servers
HTTP/1.1 200 OK
Server: nginx/1.22.0
Date: Wed, 19 Oct 2022 12:14:34 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Mon, 23 May 2022 23:59:19 GMT
Connection: keep-alive
ETag: "628c1fd7-267"
Accept-Ranges: bytes
```

### Endpoints

When a service is created, `endpoints` are also created to point to the different pods that provide the service:

```console
$ k get pods -o=wide
NAME                             READY   STATUS    RESTARTS   AGE    IP          NODE             NOMINATED NODE   READINESS GATES
curl-client                      1/1     Running   0          173m   10.1.0.31   docker-desktop   <none>           <none>
nginx-servers-84788994c9-v6656   1/1     Running   0          150m   10.1.0.34   docker-desktop   <none>           <none>
$ k get ep
NAME            ENDPOINTS      AGE
nginx-servers   10.1.0.34:80   3m32s
$ k scale deployment/nginx-servers --replicas=3
deployment.apps/nginx-servers scaled
$ k get pods -o=wide --show-labels
NAME                             READY   STATUS    RESTARTS   AGE    IP          NODE             NOMINATED NODE   READINESS GATES   LABELS
curl-client                      1/1     Running   0          176m   10.1.0.31   docker-desktop   <none>           <none>            run=curl-client
nginx-servers-84788994c9-hjhsr   1/1     Running   0          82s    10.1.0.36   docker-desktop   <none>           <none>            app=nginx,lab=services,pod-template-hash=84788994c9
nginx-servers-84788994c9-r7lpv   1/1     Running   0          82s    10.1.0.37   docker-desktop   <none>           <none>            app=nginx,lab=services,pod-template-hash=84788994c9
nginx-servers-84788994c9-v6656   1/1     Running   0          153m   10.1.0.34   docker-desktop   <none>           <none>            app=nginx,lab=services,pod-template-hash=84788994c9
$ k get ep
NAME            ENDPOINTS                                AGE
nginx-servers   10.1.0.34:80,10.1.0.36:80,10.1.0.37:80   5m46s
```

So, based on **selectors**, the `nginx-servers` service is able to identify the `nginx-servers` pods & load balance the traffic to them.

## Namespaces

Create a separate namespace called `lab-5-client` & activate it:

```console
$ k create ns lab-5-client
namespace/lab-5-client created
$ kubens lab-5-client
Context "docker-desktop" modified.
Active namespace is "lab-5-client".
```

Run a `curl-client` pod in the `lab-5-client` & try to access the `nginx-servers` pods by IP, then by service:

```console
k get pods -n lab-5 -o=wide
NAME                             READY   STATUS    RESTARTS   AGE     IP          NODE             NOMINATED NODE   READINESS GATES
nginx-servers-84788994c9-hjhsr   1/1     Running   0          8m54s   10.1.0.36   docker-desktop   <none>           <none>
nginx-servers-84788994c9-r7lpv   1/1     Running   0          8m54s   10.1.0.37   docker-desktop   <none>           <none>
nginx-servers-84788994c9-v6656   1/1     Running   0          160m    10.1.0.34   docker-desktop   <none>           <none>
$ k run -it --rm --image curlimages/curl:7.85.0 curl-client -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ $ curl --connect-timeout 5 -I 10.1.0.36
HTTP/1.1 200 OK
Server: nginx/1.22.0
Date: Wed, 19 Oct 2022 12:27:55 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Mon, 23 May 2022 23:59:19 GMT
Connection: keep-alive
ETag: "628c1fd7-267"
Accept-Ranges: bytes
/ $ curl --connect-timeout 5 -I nginx-servers
curl: (6) Could not resolve host: nginx-servers
```

IP resolution is OK, however the service is KO. This is expected, because kubernetes services are scoped by namespace:

```console
/ $ curl --connect-timeout 5 -I nginx-servers.lab-5
HTTP/1.1 200 OK
Server: nginx/1.22.0
Date: Wed, 19 Oct 2022 12:31:47 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Mon, 23 May 2022 23:59:19 GMT
Connection: keep-alive
ETag: "628c1fd7-267"
Accept-Ranges: bytes
```