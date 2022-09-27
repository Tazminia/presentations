# Kubernetes local environment

## Overview

This page is not a real lab but it explain possible solutions to have a local working kubernetes cluster to test commands with.

A local working k8s cluster is mandatory to run the following labs unless you have acces to a real cluster.

## Docker desktop

When already using docker desktop, you can enable a local kubernetes cluster.

This will create a single node cluster on your personal laptop.

Follow the official docker [documentation](https://docs.docker.com/desktop/kubernetes/#enable-kubernetes) to do the setup.

## Minikube

Minikube is a popular solution for managing a local kubernetes cluster. It can use docker as the container runtime.

Follow the official [minikube](https://minikube.sigs.k8s.io/docs/start/) documentation to get started.

## k3s

[k3s](https://k3s.io/) is a lightweight version of kubernetes.

It does not include all kubernetes feature but is a good choice when working with machines that have limited resources (CPU & RAM)

## Other

There are many other choices to run a local kubernetes cluster, including [colima](https://github.com/abiosoft/colima).

Feel free to choose one or setup a real cluster if possible.
