---
title: "Kubernetes commands cheat sheet"
toc: true
toc_label: "Contents"
tags:
  - Kubernetes
  - kubectl

date: February 23, 2021
layout: single
classes: wide
excerpt: "This is a curated list of Kubernetes (kubectl) commands that are most frequently used."
---

This is intended to be a curated list of the most often used Kubernetes (kubectl) commands.
I assume you already have some knowledge of the basic Kubernetes building blocks: Nodes, Pods, Deployments, ReplicaSets, etc.
For some commands I will also include their output in part or in full.

## Getting information about the Kubernetes API (Kubernetes objects that you can configure)
```bash
[liviu@kub]$ kubectl api-resources
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

The first column (NAMES) tells us the object name, that we can use further like: `kubectl get nodes` or `kubectl get services`.  
The second column (SHORTNAMES) tells us the shortname of the object, so we can substitute `services` with `svc` or `configmaps` with `cm`.  
The forth column (KIND) is the object name that we can use in the `kind` attribute when applying YAML configuration declaratively.  

When applying configuration with YAML files, we need to know the possible YAML fields that each object type supports. We can find out this information with the `kubectl explain` command, like this:
```bash
[liviu@kub]$ kubectl explain pods
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
[liviu@kub]$ kubectl explain pods.spec
# even further...
[liviu@kub]$ kubectl explain pods.spec.tolerations
[liviu@kub]$ kubectl explain pods.metadata
[liviu@kub]$ kubectl explain pods.metadata.uid
# You can go like this as deep as you want into the YAML tree

# Most commands have specific options, but all commands have some common options, that can be found here:
[liviu@kub]$ kubectl options
```

## Getting information about your Kubernetes cluster
```bash
[liviu@kub]$ kubectl version  
[liviu@kub]$ kubectl cluster-info
[liviu@kub]$ kubectl config view        # prints the information from ~/.kube/config
[liviu@kub]$ kubectl get nodes
NAME                                 STATUS   ROLES    AGE   VERSION
master-worker-b437e0341e817eb334f2   Ready    master   12d   v1.18.10

[liviu@kub]$ kubectl get namespaces     # or 'ns'
NAME                                STATUS   AGE
default                             Active   13d
kube-node-lease                     Active   13d
kube-public                         Active   13d
kube-system                         Active   13d
mynamespace                         Active   12d
[liviu@kub]$ kubectl get deploy -n mynamespace # get deployments from mynamespace
[liviu@kub]$ kubectl get pods -o wide
[liviu@kub]$ kubectl get pod mypod -o yaml # outputs the YAML config used to create my-pod
[liviu@kub]$ kubectl get pods --show-labels # labels are important for Pods because they connect them to Services.
[liviu@kub]$ kubectl get pods -l 'environment in (prod),tier in (frontend)' # filter Pods by labels.

# Get pods running on a specific Node
[liviu@kub]$ kubectl get pods --field-selector=spec.nodeName=mynode

# Get list of Deployments and Services at the same time
[liviu@kub]$ kubectl get deploy,svc -n ptc-default

# get PersistentVolumes sorted by capacity
[liviu@kub]$ kubectl get pv --sort-by=.spec.capacity.storage # object specific fields can be inspected with `kubectl explain`

# get all running pods in the namespace
[liviu@kub]$ kubectl get pods --field-selector=status.phase=Running -n mynamespace

# Getting all objects from all namespaces
[liviu@kub]$ kubectl get all --all-namespaces
```

## Getting information about a specific object
```bash
[liviu@kub]$ kubectl describe pod <my-pod>
[liviu@kub]$ kubectl describe svc <my-service>
[liviu@kub]$ kubectl describe ingress <my-ingress> --all-namespaces
```

## Editing a Kubernetes object
```bash
# To specify the text editor, use the environment variable KUBE_EDITOR
[liviu@kub]$ kubectl edit deploy mydeploy
[liviu@kub]$ kubectl edit svc myservice
[liviu@kub]$ kubectl scale deploy mydeploy --replicas=3 # creates 3 instances of a given deployment
```

## Creating new objects or altering existing objects
```bash
[liviu@kub]$ kubectl create ns dev          # creates namespace 'dev'
[liviu@kub]$ kubectl create -f obj.yaml     # creates the object defined in obj.yaml

# Compares the current state of the cluster against the state that the cluster would be in if the obj.yaml was applied.
[liviu@kub]$ kubectl diff -f ./obj.yaml

# Create a deployment in imperatively.
[liviu@kub]$ kubectl create deploy kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

# Generate the YAML that would be used to create this deploy declaratively.
[liviu@kub]$ kubectl create deploy --dry-run --image=nginx --output=yaml
```
Difference between `create` and `apply`:  
- `kubectl apply` use the declarative approach (you specify your desired cluster state, and it will try to get it to that state), and won't fail if the resource already exists.  
- `kubectl create` uses the imperative approach, and will fail if the resource already exists. To overcome this, you can do something like this:
```bash
[liviu@kub]$ kubectl create svc --dry-run=true -o yaml | kubectl apply -f -
```

## Restarting a deployment
```bash
[liviu@kub]$ kubectl scale deployment mydeploy --replicas=0
[liviu@kub]$ kubectl scale deployment mydeploy --replicas=1
# or
[liviu@kub]$ kubectl rollout restart deploy mydeploy
```

## Getting the rollout history of your deployments
```bash
[liviu@kub]$ kubectl rollout history deploy -n myns  # get rollout history of all deployments from namespace 'ns'

# This one is useful if you want to see if the Docker image was changed between rollouts.
[liviu@kub]$ kubectl rollout history deploy mydeploy
[liviu@kub]$ kubectl rollout history deploy mydeploy --revision=2

[liviu@kub]$ kubectl rollout undo deploy mydeploy --to-revision=1 # brings back an old revision.
```

## Deleting resources
```bash
[liviu@kub]$ kubectl delete namespace myns
[liviu@kub]$ kubectl delete deploy mydeploy
# Deletes Pods and Services by label
[liviu@kub]$ kubectl delete pods,services -l lkey=lval
[liviu@kub]$ kubectl drain mynode # removes all pods scheduled on mynode
```

## Checking if you have permissions to do something
```bash
# Check if I am allowed to create a new deployment in namespace 'your-namespace'
[liviu@kub]$ kubectl auth can-i create deploy -n your-namespace
[liviu@kub]$ kubectl auth can-i create pods --as liviu --namespace apps
[liviu@kub]$ kubectl auth can-i '*' '*' # check if I can do anything
```

## Getting inside your pods
Sometimes we need to run a command from inside a Pod (e.g. bash). Imagine we have a Pod running Postgres and we need to access 'psql' so we can see the contents of a table.
```bash
[liviu@kub]$ kubectl exec --stdin --tty mypod -n myns -- /bin/sh

# See the contents of '/' inside Pod from Namespace myns
[liviu@kub]$ kubectl exec -it mypod -n myns -- ls /
```

## Getting the logs of a given pod
```bash
[liviu@kub]$ kubectl logs mypod -n mynamespace # get logs of Pod mypod from ns mynamespace
[liviu@kub]$ kubectl logs mypod --all-containers=true # in case mypod has multiple containers

```