---
marp: true
theme: default
style: |
    img[alt~="center"] {
    display: block;
    margin: 0 auto;
    }
    section {
    background-color: #FBFEF9;
    color: #000004;
    }
    section.lead h1 {
    text-align: center;
    }
    b, strong {
        color: #D62828;
    }
paginate: true
---

<!-- _class: lead -->

# Kubernetes basics

---

# Discussed topics

* Containers orchestration 
* Kubernetes
* Kubernetes architecture components
* Pods
* Interacting with the cluster
* Deployments, Daemonsets, Jobs & CronJobs
* Labels & filtering
* Namespaces
* Services

---

<!-- _class: lead -->
# Containers orchestration 

---

## Containers

In order to follow this presentation, basic understanding of containers and docker usage is required. 

If this kind of knowledge is missing, the reader is invited to first, check the [containers](https://github.com/Tazminia/presentations/blob/main/containers/containers.pdf) presentation before proceeding any further.

---

## Containers limits

When working with containers multiple limits can be observed:

* What happens if the host machine is unstable ?
* What happens if the containers use all of the available cpu or memory ?
* Which machine is best suited to run the containers ?
* What if one container is no longer enough to handle the load ?
* What is responsible for restarting an unhealthy container ?
* What is an unhealthy container ?
* How to access a given service in a container ?
* How to define communication rules between containers ?

---

## Containers orchestration

Containers orchestration is the automated management of the operational loads related to running containers.

Examples of what this operational load includes:

* Scheduling
* Scaling
* Health monitoring
* Network access

---

<!-- _class: lead -->
# Kubernetes 

---

## Kubernetes

**Kubernetes** is the most popular container orchestration system.

**Kubernetes** is open-source & each public cloud providers offers his own managed service:

* GCP: Google Kubernetes Engine (GKE & GKE autopilot)
* AWS: Elastic Kubernetes Service (EKS)
* Microsoft Azure: Azure Kubernetes Service (AKS)

---

## Kubernetes features

Some of kubernetes basic features are:

* Running containers on a cluster (multiple machines connected by network)
* Containers health monitoring & auto recovery
* Node health monitoring & container re-scheduling
* Service discovery

---

<!-- _class: lead -->
# Kubernetes architecture

---

## Kubernetes components (1/3)

![center](img/kubernetes-components.png)

---

## Kubernetes components (2/3)

![center](img/control-plane.png)

---

## Kubernetes components (3/3)

![center](img/nodes-components.png)

---

<!-- _class: lead -->
# Pods & Deployments

---

## Pods

Pods are units containing one or multiple containers.

Pods are kubernetes objects that are created by calling the kubernetes API.

Pods follow the structure below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  # ... ignored for now
  name: nginx # Pod name
spec:
  containers:
  - image: nginx:stable # Container image
    name: nginx # Container name
  # ... ignored for now
```

---

## Running a simple Pod

In order to run a simple pod we can do the following:

```console
$ cat hands-on/yaml-samples/pod-sample.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:stable
    name: nginx
$ kubectl apply -f hands-on/yaml-samples/pod-sample.yaml
pod/my-nginx created
```

In docker, this would have been equivalent to:

```console
$ docker run -d -it nginx:stable
```

---

<!-- _class: lead -->
# Kubernetes basic commands

---

Please proceed to doing lab-1 in the hands-on folder.

---
