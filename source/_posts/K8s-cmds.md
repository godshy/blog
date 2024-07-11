---
title: K8s cmds
date: 2023-06-05 10:57:16
tags: K8s
category: commands
---

# Basic K8s-Commands

## Cluster logs

- Get cluster nodes logs 
``` bash
    kubectl cluster-info dump --output-directory cluster_info_dump_$(uname -n)  -A
```
## View and modify kubectl configs

``` bash
    kubectl config view
    # file is located in ~/.kube/config
```
- change to use other context to access the cluster
``` bash
    kubectl config use-context <name>@<cluster> --kubeconfig /path_to_config_file
```
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
    etcdctl member list  # check how many nodes are managed by etcd server
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

- set ETCD snapshot
``` bash

# create snapshot for ETCD
    ETCDCTL_API=3 etcdctl \
        snapshot save snahpshot.db
    ls

# check status of ETCD
    ETCDCTL_API=3 etcdctl \
        snapshot status snapshot.db


```
- Procedures to restore ETCD from snapshot

``` bash
# stop api-server
    service kube-apiserver stop
# restore ETCD from snapshot, this created a brand new ETCD cluster
    ETCDCTL_API=3 etcdctl \
        snapshot restore <snapshot_name> \ 
        --data-dir /var/lib/etcd-from-backup

# modify etcd.service under defination file to use the assigned data-dir
--data-dir=/var/lib/etcd-from-back-up

    systemctl daemon-reload
    service kube-apiserver start

```
- Restore etcd as external server 


``` bash

# Execute restore on etcd server, same
    
    ETCDCTL_API=3 etcdctl \
        snapshot restore <snapshot_name> \ 
        --data-dir /var/lib/etcd-from-backup
# Modify data-dir, make sure owner is correct

    chown -R etcd:etcd /var/lib/etcd-from-backup

# Modify configurations
    vi etc/systemd/system/etcd.service

# Restart
    systemctl daemon-reload
    systemctl restart etcd
    
```

- Check if ETCD is stacked or external server
``` bash
    kubectl get pods -n kube-system
    # If etcd is shown as pods in kube-system, then it is stacked ETCD.
    # or 
    kubectl describe apiserver -n kube-system
    # URL in it shows hos apiserver communicate with ETCD
```

- Check status on external server
``` bash
    ps -ef | grep -i nels
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

- add username/password or token and securely access to api-server (most insecure)
``` yaml
    # add username/password info in .csv
    # then specify the related file
    --basic-auth-file=user_info.csv
    # kube-apiserver.service
    # or pod defination file under /etc/kubernetes/manifests

    # use curl command to securely access to api server
    curl -v -k <url_to_api_server:6443/api/v1/pods --header "Authorization: Bearer <token>"

    # or 
    curl -v -k <url> -u "user:pass"
```

- RBAC related commands
``` bash
    kubectl get roles
    kubectl get rolebindings
    kubectl describe role developer
    kubectl auth can-i <command > --as <usergroup> # for example create deployments

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
- CSR related command

``` bash
    kubectl get csr
    kubectl certificate approve
    kubectl get csr <name> -o yaml
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
    kubectl run <name> --image=<image_name>
```
- Check pods location
``` bash
    kubectl get pods -o wide
```
- Check pods manifest (able to export as yaml) 
``` bash
    kubectl get pods <pods_name> -o yaml
```

- apply changes to pod 
``` bash
    kubectl apply -f 
```

## Work with replicasets

- Create/update replicaset

``` bash
    kubectl create -f desfilename.yaml
```

- Check status of replicaset

``` bash
    kubectl get replicaset <name_of_replicaset>
    kubectl describe replicaset name_of_replicaset
```

- scale up/down without modifying the yaml file.

``` bash
    kubectl scale --replicas=6 replicaset (type) myapp-replicaset (name in the metadata) # or

    kubectl scale --replicas=6 -f desfilename.yaml
```

- scale up/down by modifying the yaml file first 
``` bash
    kubectl get replicaset <name_of_replicaset> -o yaml > something.yaml
    
    kubectl replace/apply -f something.yaml # able to recreate node
```
## Kubelet
Configuration file of kubelet is located at 
``` /var/lib/kubelet/config.yaml ``` or 
```/etc/kubernetes/kubelet.conf ```

## Deployment

- create deployment by commands

``` bash
    kubectl create deployment --image=<image_name> <deploy_name>
```

- by configuration file
``` bash
    kubectl create -f deploy_des_file.yml
    kubectl get deployments
```

- edit deployment
``` bash
    kubectl edit deployement <name>
```

### Generate deployment yaml file without creating it

- POD manifest
``` bash
    kubectl run <pod_name> --image=<image_name> --dry-run=client -o yaml
```

- Create Deployment YAML file

``` bash
    kubectl create deployment <deployment_name> --image=<image_name> --dry-run=client -o yaml
```

### Check deployment rollout and update pods 

``` bash
    kubectl rollout status deployment/<deployment_name>

    kubectl rollout history deployment/<deployment_name> #Check rollout history


    # update pods
    kubectl apply -f deployment-definition.yml # after edit image spec in definination, apply it

    kubectl set image deployment <deployment_name> \ <image_name:version>

    kubectl rollout undo deployment <deployment_name>
```


## Service creation

- Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
``` bash
    kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml # automatically use pods labels as selectors
```

- Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

``` bash
    kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```

- create service in a imperative way
``` bash
    kubectl expose deployment <deploy_name> --port 80
```

- able to enable service while creating pods
``` bash
    kubectl run <name> --image=<image_name> --port=<port_name> --expose
```

## Namaespaces

- To create namespace

``` bash
    kubectl create namespace dev # or
    kubectl create -f <namespace_definition.yml>
```
- To add pod in the certain namespace:

``` bash
    --namespace=<namespace_name>
```

 Or to add ```namespace:``` under  ```metadata: ``` in the pod definition file.

- To change to other namespace so that no need to add ``` -n <namespace> ``` to see the pods
``` bash
    kubectl config set-context $(kubectl config current-context) --namespace=dev
```
- To see all the pods in all name space

``` bash
    kubectl get pods --all-namespaces
```

## Schedulers

- Filter objects by specifying selectors
``` bash
    kubectl get <sth> --selector <sth.>
```

-Create configmap
``` bash
    kubectl create configmap --name <configmap_name> --from-file=<PATH_TO_FILE> -n <namespace>
```
## Taint - Node 
- add taints to nodes
``` bash
    kubectl taint nodes <node_name> <keys=values>:<taint-effect{NoSchedule|PreferNoSchedule|NoExecute}>
```

- add tolerations on pods

``` yaml
spec:
  tolerations:
  - key: "<key>"
    operator: "Equal"
    value: "<value>"
    effect: "<taint-effect>"
```
## Node affinity

### Node selector:
- one way to assign nodes for specific pods by setting label on nodes 
``` yaml
<pod>
spec:
  nodeSelector:
    size: Large
```
- then label a node by
``` bash
    kubectl label nodes <node-name> <label-key>=<label-value>
```

- Node affinity provide more complicated assignment and placement for nodes 

__Values can be set to blank if operator uses Exists.__
``` yaml 
# pod definition
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExectuion:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In  # placed on any nodes with values under in 
            values:
            - Large
```

### Resource requests

- allocate resources desirable for pods:
``` yaml
spec:
  resources:
    requests:
      memory: "4Gi"
      cpu: 2
    limits:
      memory:
      cpu:
```

## Pod monitoring

- check memory, cpu
``` bash
    kubectl top node/pods
```

- check logs
``` bash
    kubectl logs -f <pod_name>
    kubectl previous -f <pod_name>
```
## Node execution

- enable shell in kubernetes
``` bash
    kubectl exec --stdin --tty <pod_name> <container_name> -n <namespace> -- /bin./bash
```

- copy files from local to pods vice versa.

``` bash
# Copy file from pods to local director
    kubectl cp <namespace>/<pod_name>:/PATH/FILE /LOCAL_PATH/FILE_NAME

# Copy file from local to pods
    kubectl cp /LOCAL_PATH/FILE_NAME <namespace>/<pod_name>:/PATH/FILE 
```

## ENV VAR and ConfigMap

``` bash
    kubectl create configmap \
    <config-name> --from-literal=<key>=<value>

    kubectl create configmap \
    <config-name> --from-file=<configmap_file_name>

    kubectl create -f cfgmap_name

    # display configmaps
    kubectl get/describe <configmap_name>
```

## Secrets

``` bash
    # create secret using command
    kubectl create secret generic <secret-name> \
    --from-literal=<key>=<value>

    kubectl create secret generic \
    <secret-name> --from-file=<secret_file_name>

    # docker registry secret
    kubectl create secret docker-registry <secret-name> \
    --docker-server= \
    --docker-username= \
    --docker-password= \
    --docker-email=
```

## Service account
- A service account is an account for services like jenkins and Prometheus to perform tasks on clusters
``` bash
    # create service account
    kubectl create serviceaccount <account_name>
```
## Procedure for pods when executing node graceful restart

``` bash
# drain pod on the nodes you want to perform reboot procedure
    kubectl drain --ignore-daemonsets --delete-emptydir-data
    kubectl describe nodes <node-name>
    # perform node reboot
    kubectl uncordon <node_name>

```


## Kube storage
- default directory for docker:
```/var/lib/docker ```

- To preserve data for container, always create persistent volume for it
``` bash
    docker volume create <name>
    docker run -v <volume_name>:<dir_in_container> <container_name> #volume mounted on /var/lib/docker/volumes or binding mounting on any directories
    docker run -v <mounted_path>:<dir_in_container> <container_name>
```
## Network
- view kubelet option about CNI
``` bash
    ps -aux | grep kubelet
    ls /opt/cni/bin
    ls /etc/cni/net.d
```

## JsonPath
1. what information you need?
   ``` kubectl get ?```
2. output as json
3. check the info and check JSON PATH
``` bash
kubectl get pods -o=json='{JSON_PATH}'

kubectl get pods -o=jsonpath='{ .items[*].metadata.name}
```