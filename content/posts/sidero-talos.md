+++
title = 'Talos Linux'
date = 2024-01-09T12:34:33+01:00
draft = false
+++

# Talos Linux - OS for Kubernetes

Talos Linux is immutable, secure, minimal OS created by [Sidero Labs](https://www.talos.dev/). It can be deployed on all cloud platforms, bare metal and virtualisation platforms.

For testing purposes it can be deployed on Docker as well. 

We will need the following :
- docker host,
- __talosctl__ - this is the client for all talos related functions
- __kubectl__ - Kubernetes client


First, get __talosctl__ on your docker host:
```bash
curl https://github.com/siderolabs/talos/releases/download/v1.6.7/talosctl-linux-amd64 -O talos-linux-amd64
```

Then get __kubectl__:
```bash
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"   
```

Once we have both tools we can issue the command to create a Talos Linux cluster based on docker:

```bash
talosctl cluster create
```


This will download docker images (of Talos Linux), create a docker network, spin up Kubernetes cluster using docker containers as cluster nodes.

```bash
flamur@ub:~$ talosctl cluster show
PROVISIONER       docker
NAME              talos-default
NETWORK NAME      talos-default
NETWORK CIDR      10.5.0.0/24
NETWORK GATEWAY   
NETWORK MTU       1500

NODES:

NAME                           TYPE           IP         CPU   RAM   DISK
talos-default-controlplane-1   controlplane   10.5.0.2   -     -     -
talos-default-worker-1         worker         10.5.0.3   -     -     -
```

Upon sucesfull creation of Kubernets cluster, kubeconfig will be merged to default location and we will have a working connection to Kubernetes API.

```bash
kubectl get nodes -o wide
NAME                           STATUS   ROLES    AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION   CONTAINER-RUNTIME
talos-default-controlplane-1   Ready    master   115s   v1.29.3   10.5.0.2      <none>        Talos (v1.6.7)   <host kernel>    containerd://1.5.5
talos-default-worker-1         Ready    <none>   115s   v1.29.3   10.5.0.3      <none>        Talos (v1.6.7)   <host kernel>    containerd://1.5.5
```



In the end, to destroy the cluster we issue:
```bash
talosctl cluster destroy
```
