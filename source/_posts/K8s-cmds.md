---
title: K8s cmds
date: 2023-06-05 10:57:16
tags: K8s
category: Basic_Commands
---

# Basic K8s-Commands

## ETCD related commands

- ETCDCTL version 2
``` bash
    etcdctl backup
    etcdctl cluster-health
    etcdctl mk
    etcdctl mkdir
    etcdctl set
```

- ETCDCTL version 3
``` bash
    etcdctl snapshot save 
    etcdctl endpoint health
    etcdctl get
    etcdctl put
```

- Set version of version
``` bash
    export ETCDCTL_API = 3
```
__When API version is not set, it is assumed to be set to version 2.__

- For ETCDCTL to authenticate to the ETCD API server, the certificate files are available in the etcd-master.

``` bash
    --cacert /etc/kubernetes/pki/etcd/ca.crt     
    --cert /etc/kubernetes/pki/etcd/server.crt     
    --key /etc/kubernetes/pki/etcd/server.key
```

In total:

``` bash
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"
```

## Kube-API commands

- View api-server thru kubeadm, kubeadm deploys kube-api as a pod in the kube-system namespace on the master node.

``` bash
    kubectl get pods -n kube-system
```
- view api-server options with kubeadmin

``` bash
    cat /etc/kubernets/manifests/kube-apiserver.yaml
```

- view api-server options without kubeadmin

``` bash
    cat /etc/systemd/system/kube-apiserver.service
```
checking active process of apiserver by

``` bash
    ps -aux | grep kube-apiserver
```

## Kube controller manager

- check status with the deployment by kubeadmin

``` bash
    kube get pods -n kube-system
```
- view options with kubeadmin

``` bash
    cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```
- view options without kubeadmin
 
``` bash
   cat /etc/systemd/system/kube-controller-manaer.service
```
- check process of kube controller manager

``` bash
    ps -aux | grep kube-controller-manager
```

## Kube-scheduler 

- View kube scheduler thru kubeadm

``` bash
    cat /etc/kubernetes/manifests/kube-scheduler.yaml
```
- Check thru process 

``` bash
    ps -aux | grep kube-scheduler
```


## Run pods 
``` bash
    kubectl run //name --image=//image_name
```
- Check pods location
``` bash
    kubectl get pods -o wide
```
- Check pods manifest (able to export as yaml) 
``` bash
    kubectl get pods pods_name -o yaml
```

- apply changes to pod (not verified)
``` bash
    kubectl apply -f 
```

## Work with replicasets

- Create replicaset

``` bash
    kubectl create -f desfilename.yaml
```

- Check status of replicaset

``` bash
    kubectl get replicaset name_of_replicaset
    kubectl describe replicaset name_of_replicaset
```

- scale up/down without modifying the yaml file.

``` bash
    kubectl scale --replicas=6 replicaset (type) myapp-replicaset (name in the metadata) # or

    kubectl scale --replicas=6 -f desfilename.yaml
```

- scale up/down by modifying the yaml file first 
``` bash
    kubectl get replicaset name_of_replicaset -o yaml >> something.yaml
    
    kubectl replace/apply -f something.yaml
```

