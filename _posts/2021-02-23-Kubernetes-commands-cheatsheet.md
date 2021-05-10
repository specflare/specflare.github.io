---
title: "Kubernetes (kubectl) commands cheat sheet"
toc: true
toc_label: "Contents"
tags:
  - Kubernetes
  - kubectl

date: February 23, 2021
layout: single
classes: wide
excerpt: "A curated list of Kubernetes (kubectl) commands that are most frequently used."
---

This is intended to be a curated list of the most often used Kubernetes (kubectl) commands.
I assume you already have some knowledge of the basic Kubernetes building blocks: Nodes, Pods, Deployments, ReplicaSets, etc.
For some commands I will also include their output in part or in full.

## Getting information about the Kubernetes API (Kubernetes objects that you can configure)
```bash
$ kubectl api-resources
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
...
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
replicationcontrollers            rc                                          true         ReplicationController
services                          svc                                         true         Service
daemonsets                        ds           apps                           true         DaemonSet
deployments                       deploy       apps                           true         Deployment
replicasets                       rs           apps                           true         ReplicaSet
statefulsets                      sts          apps                           true         StatefulSet
ingresses                         ing          networking.k8s.io              true         Ingress
... (some list elements were omitted)
```

The first column `NAMES` tells us the object name, that we can use further like: `kubectl get nodes` or `kubectl get services`.  
The second column `SHORTNAMES` tells us the shortname of the object, so we can substitute `services` with `svc` or `configmaps` with `cm`.  
The forth column `KIND` is the object name that we can use in the `kind` attribute when applying YAML configuration declaratively.  

When applying configuration with YAML files, we need to know the possible YAML fields that each object type supports. We can find out this information with the `kubectl explain` command, like this:
```bash
$ kubectl explain pods
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion   <string>
   kind         <string>
   metadata     <Object>
   spec         <Object>
   status       <Object>

# Then, we go even further...
$ kubectl explain pods.spec
# even further...
$ kubectl explain pods.spec.tolerations
$ kubectl explain pods.metadata
$ kubectl explain pods.metadata.uid
# You can go like this as deep as you want into the YAML tree

# Most commands have specific options, but all commands have some common options, that can be found here:
$ kubectl options
```

## Getting information about your Kubernetes cluster
```bash
$ kubectl version  
$ kubectl cluster-info
$ kubectl config view        # prints the information from ~/.kube/config
$ kubectl get nodes
NAME                                 STATUS   ROLES    AGE   VERSION
master-worker-b437e0341e817eb334f2   Ready    master   12d   v1.18.10

$ kubectl get namespaces     # or 'ns'
NAME                                STATUS   AGE
default                             Active   13d
kube-node-lease                     Active   13d
kube-public                         Active   13d
kube-system                         Active   13d
mynamespace                         Active   12d
$ kubectl get deploy -n mynamespace # get deployments from mynamespace
$ kubectl get pods -o wide
$ kubectl get pod mypod -o yaml # outputs the YAML config used to create my-pod
$ kubectl get pods --show-labels # labels are important for Pods because they connect them to Services.
$ kubectl get pods -l 'environment in (prod),tier in (frontend)' # filter Pods by labels.

# Get pods running on a specific Node
$ kubectl get pods --field-selector=spec.nodeName=mynode

# Get list of Deployments and Services at the same time
$ kubectl get deploy,svc -n ptc-default

# get PersistentVolumes sorted by capacity
$ kubectl get pv --sort-by=.spec.capacity.storage 
# object specific fields can be inspected with `kubectl explain pv.spec.capacity.storage`

# get all running pods in the namespace
$ kubectl get pods --field-selector=status.phase=Running -n mynamespace

# Getting all objects from all namespaces
$ kubectl get all --all-namespaces
```

## Getting information about a specific object
```bash
$ kubectl describe pod <my-pod>
$ kubectl describe svc <my-service>
$ kubectl describe ingress <my-ingress> --all-namespaces
```

## Editing a Kubernetes object
```bash
$ kubectl edit deploy mydeploy

# To specify the text editor, use the environment variable KUBE_EDITOR
$ KUBE_EDITOR="nano" kubectl edit svc myservice
$ kubectl scale deploy mydeploy --replicas=3 # scales mydeploy to 3 instances
```

## Creating new objects or altering existing objects
```bash
$ kubectl create ns dev          # creates namespace 'dev'
$ kubectl create -f obj.yaml     # creates the object defined in obj.yaml

# Compares the current state of the cluster against the state that the cluster would be
# in if the obj.yaml was applied.
$ kubectl diff -f ./obj.yaml

# Create a deployment in imperatively.
$ kubectl create deploy kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

# Generate the YAML that would be used to create this deploy declaratively.
$ kubectl create deploy --dry-run --image=nginx --output=yaml
```
Difference between `create` and `apply`:  
- `kubectl apply` use the declarative approach (you specify your desired cluster state, and it will try to get it to that state), and won't fail if the resource already exists.  
- `kubectl create` uses the imperative approach, and will fail if the resource already exists. To overcome this, you can do something like this:
```bash
$ kubectl create svc --dry-run=true -o yaml | kubectl apply -f -
```

## Restarting a deployment
```bash
$ kubectl scale deployment mydeploy --replicas=0
$ kubectl scale deployment mydeploy --replicas=1
# or
$ kubectl rollout restart deploy mydeploy
```

## Getting the rollout history of your deployments
```bash
$ kubectl rollout history deploy -n myns  # get rollout history of all deployments from namespace 'ns'

# This one is useful if you want to see if the Docker image was changed between rollouts.
$ kubectl rollout history deploy mydeploy
$ kubectl rollout history deploy mydeploy --revision=2

$ kubectl rollout undo deploy mydeploy --to-revision=1 # brings back an old revision.
```

## Deleting resources
```bash
$ kubectl delete namespace myns
$ kubectl delete deploy mydeploy
# Deletes Pods and Services by label
$ kubectl delete pods,services -l lkey=lval
$ kubectl drain mynode # removes all pods scheduled on mynode
```

## Checking if you have permissions to do something
```bash
# Check if I am allowed to create a new deployment in namespace 'your-namespace'
$ kubectl auth can-i create deploy -n your-namespace
$ kubectl auth can-i create pods --as liviu --namespace apps
$ kubectl auth can-i '*' '*' # check if I can do anything
```

## Getting inside your pods
Sometimes we need to run a command from inside a Pod (e.g. bash). Imagine we have a Pod running Postgres and we need to access 'psql' so we can see the contents of a table.
```bash
$ kubectl exec --stdin --tty mypod -n myns -- /bin/sh

# See the contents of '/' inside Pod from Namespace myns
$ kubectl exec -it mypod -n myns -- ls /
```

## Getting the logs of a given pod
```bash
$ kubectl logs mypod -n mynamespace # get logs of Pod mypod from ns mynamespace
$ kubectl logs mypod --all-containers=true # in case mypod has multiple containers
$ kubectl logs --since=1h mypod
```