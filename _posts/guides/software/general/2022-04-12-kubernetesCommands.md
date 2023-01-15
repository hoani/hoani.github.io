---
title: "K8s Kubectl"
excerpt: "Kubernetes commands"
permalink: "/guides/software/kubernetes"
toc: true
categories:
  - guide
  - software
  - software-guide
---

This is a reminder of kubectl commands for those like me who won't use `kubectl` often.

## Getting Stuff

Basics:

```sh
kubectl get all # Get deployments, pods, services etc.
kubectl get pod 
kubectl get configmap
kubectl get secret
kubectl get node # What the cluster is running on.
```

More info:

```sh
kubectl get all -o wide
kubectl get node -o wide # Gives us IP address to the cluster
```

Get full details of something:

```sh
kubectl describe pod pod-name
kubectl describe svc service-name
```

## Checking logs

```sh
kubectl logs podname
kubectl logs podname -f # Stream logs
```

## Deploying stuff

```sh
kubectl apply -f ./config.yaml
```
