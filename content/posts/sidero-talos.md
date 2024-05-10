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
curl --location --output talosctl https://github.com/siderolabs/talos/releases/download/v1.7.1/talosctl-linux-amd64
```

Then get __kubectl__:
```bash
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"   
```

Make sure both tools are executable and can be found by your PATH environment variable, then we can issue the command to create a Talos Linux cluster based on docker. We will create a Kubernetes cluster consisting of three control plane and two worker nodes.

```bash
talosctl cluster create \
      --controlplanes 3 \
      --workers 2 \
      --kubernetes-version "1.29.0" \
      --provisioner "docker" \
      --wait \
      --wait-timeout 30m
```


This will download docker images (of Talos Linux), create a docker network, spin up Kubernetes cluster using docker containers as cluster nodes, and spit out something like this, if all went well.

```bash
validating CIDR and reserving IPs
generating PKI and tokens
downloading ghcr.io/siderolabs/talos:v1.7.1
^Tcreating network talos-default
creating controlplane nodes
creating worker nodes
renamed talosconfig context "talos-default" -> "talos-default-2"
waiting for API
bootstrapping cluster
waiting for etcd to be healthy: OK
waiting for etcd members to be consistent across nodes: OK
waiting for etcd members to be control plane nodes: OK
waiting for apid to be ready: OK
waiting for all nodes memory sizes: OK
waiting for all nodes disk sizes: OK
waiting for kubelet to be healthy: OK
waiting for all nodes to finish boot sequence: OK
waiting for all k8s nodes to report: OK
waiting for all k8s nodes to report ready: OK
waiting for all control plane static pods to be running: OK
waiting for all control plane components to be ready: OK
waiting for kube-proxy to report ready: OK
waiting for coredns to report ready: OK
waiting for all k8s nodes to report schedulable: OK

merging kubeconfig into "/home/flamur/.kube/config"
PROVISIONER           docker
NAME                  talos-default
NETWORK NAME          talos-default
NETWORK CIDR          10.5.0.0/24
NETWORK GATEWAY       10.5.0.1
NETWORK MTU           1500
KUBERNETES ENDPOINT   https://127.0.0.1:44943

NODES:

NAME                            TYPE           IP         CPU    RAM      DISK
/talos-default-controlplane-1   controlplane   10.5.0.2   2.00   2.1 GB   -
/talos-default-controlplane-2   controlplane   10.5.0.3   2.00   2.1 GB   -
/talos-default-controlplane-3   controlplane   10.5.0.4   2.00   2.1 GB   -
/talos-default-worker-1         worker         10.5.0.5   2.00   2.1 GB   -
/talos-default-worker-2         worker         10.5.0.6   2.00   2.1 GB   -

```

Upon sucesfull creation of Kubernets cluster, kubeconfig will be merged to default location and we will have a working connection to Kubernetes API.

```bash
flamur@ub:/mnt/data/labs/talos-docker$ kubectl get nodes -owide
NAME                           STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION       CONTAINER-RUNTIME
talos-default-controlplane-1   Ready    control-plane   6m10s   v1.29.0   10.5.0.2      <none>        Talos (v1.7.1)   5.15.0-106-generic   containerd://1.7.16
talos-default-controlplane-2   Ready    control-plane   6m13s   v1.29.0   10.5.0.3      <none>        Talos (v1.7.1)   5.15.0-106-generic   containerd://1.7.16
talos-default-controlplane-3   Ready    control-plane   6m9s    v1.29.0   10.5.0.4      <none>        Talos (v1.7.1)   5.15.0-106-generic   containerd://1.7.16
talos-default-worker-1         Ready    <none>          6m11s   v1.29.0   10.5.0.5      <none>        Talos (v1.7.1)   5.15.0-106-generic   containerd://1.7.16
talos-default-worker-2         Ready    <none>          6m15s   v1.29.0   10.5.0.6      <none>        Talos (v1.7.1)   5.15.0-106-generic   containerd://1.7.16
```


Kubernetes cluster upgrade is as easy as typing, 
```bash
talosctl upgrade-k8s --to "1.30.0" --nodes 10.5.0.2
```

Upgrade process will start rolling upgrade, control plane nodes first than workers.
```
$ talosctl upgrade-k8s --to "1.30.0" --nodes 10.5.0.2
automatically detected the lowest Kubernetes version 1.29.0
discovered controlplane nodes ["10.5.0.2" "10.5.0.3" "10.5.0.4"]
discovered worker nodes ["10.5.0.5" "10.5.0.6"]
checking for removed Kubernetes component flags
checking for removed Kubernetes API resource versions
 > "10.5.0.2": pre-pulling registry.k8s.io/kube-apiserver:v1.30.0
 > "10.5.0.3": pre-pulling registry.k8s.io/kube-apiserver:v1.30.0
 > "10.5.0.4": pre-pulling registry.k8s.io/kube-apiserver:v1.30.0
 > "10.5.0.2": pre-pulling registry.k8s.io/kube-controller-manager:v1.30.0
 > "10.5.0.3": pre-pulling registry.k8s.io/kube-controller-manager:v1.30.0
 ...
 > "10.5.0.6": pre-pulling ghcr.io/siderolabs/kubelet:v1.30.0
updating "kube-apiserver" to version "1.30.0"
 > "10.5.0.2": starting update
 > update kube-apiserver: v1.29.0 -> 1.30.0
 > "10.5.0.2": machine configuration patched
 > "10.5.0.2": waiting for kube-apiserver pod update
...
```


In the end, to destroy the cluster we issue:
```bash
$ talosctl cluster destroy 
destroying node talos-default-controlplane-1
destroying node talos-default-controlplane-2
destroying node talos-default-controlplane-3
destroying node talos-default-worker-2
destroying node talos-default-worker-1
destroying network talos-default
```
