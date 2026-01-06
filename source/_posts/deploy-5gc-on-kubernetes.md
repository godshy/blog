---
title: Deploy-5gc-on-kubernetes
date: 2026-01-05 22:23:51
tags: k8s, helm
category: DIY and lab
---
# Install 5GC on kubernetes cluster

## Background

2 years ago, I tried to deploy 5gc on a minikube vm, and installed free5gc. However, free5gc is no longer supported and there are problems related to calico cannot be resolved. This time I select another repository and installed on a real kubernetes cluster.

## Cluster installation and preparation
For kubernetes cluster installation, check my other post __Deploy kubernetes cluster on virtual machine__. 

### CNI plugin
When installing network plugin, a calico plugin is installed, it enables pod-pod network, node-node connectivety. However, when deploying 5gs, multiple ports on pod are needed. For example, when adding UPF, an interface for user-plane and an interface for control-plane are both required. Here I installed multus-CNI to accomplish such requirement.

``` bash
    kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset.yml
```
Check multus daemonset is installed on each node:
``` bash
    kubectl get daemonset -n kube-system
NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-multus-ds   2         2         2       2            2           <none>                   18d
```
### Helm installation 

Very simple, just execute following:
``` bash
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
    chmod 700 get_helm.sh
    ./get_helm.sh

```


### Metric server
Need metric server to use ``` kubectl top``` command, other wise it will just say: "error: Metrics API not available"

``` bash
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```


However, after applied it, the pod didn't go up, after checking the logs it says:
```
    cannot validate certificate for x.x.x.x because it doesn't contain any IP SANs" node="xxx"
```
This means, kubelet server cert didn't contain Subject Alternative Name, thus metric-server cannot connect to kubelet using TLS.

Run the following to check for the server certificate
``` bash
    openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -noout -text | grep -A2 "Subject Alternative Name"
```
There are two method to solve this:

1. modify api-server side to let it ignore unsecure tls 

``` bash 
# add kubelet-insecure-tls
    spec:
      containers:
      - args:
        - --kubelet-insecure-tls
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```

2. modifiy kubelet settings to renew a certificate and let it contain new SAN.

Add following into /var/lib/kubelet/config.yaml
``` yaml
    serverTLSBootstrap: true
```
### PVC (Under construction)
If installing kubernetes on local machine, storage class and pvc need to be configured, otherwise DB or statefulset cannot be actived.



### 5GC install (Under construction)

``` bash
    helm pull oci://registry-1.docker.io/gradiant/open5gs --version 2.2.0
    helm pull oci://registry-1.docker.io/gradiant/ueransim-gnb --version 0.2.6
    mkdir 5gs
    mv open5gs-2.2.0.tgz 5gs/
    mv ueransim-gnb-0.2.6.tgz 5gs/
    cd 5gs/
    tar -zxf open5gs-2.2.0.tgz
    tar -zxf ueransim-gnb-0.2.6.tgz
    
    # If no need of 4g node, just modify values.yaml before installing. Here is my values without 4g.
    helm install open5gs -n 5gs ./open5gs-2.2.0.tgz --values ./open5gs/values_no4g.yaml
```
 
``` yaml
#  values_no4g.yaml
dbURI: "mongodb://{{ .Release.Name }}-mongodb/open5gs"

populate:
  enabled: true
  image:
    registry: docker.io
    repository: gradiant/open5gs-dbctl
    tag: 0.10.3
    pullPolicy: IfNotPresent
  initCommands: []
  # example of initCommands:
  #  - open5gs-dbctl add 999700000000001 465B5CE8B199B49FAA5F0A2EE238A6BC E8ED289DEBA952E4283B54E88E6183CA
  #  - open5gs-dbctl add_ue_with_apn 999700000000002 465B5CE8B199B49FAA5F0A2EE238A6BC E8ED289DEBA952E4283B54E88E6183CA internet
  #  - open5gs-dbctl add_ue_with_slice 999700000000003 465B5CE8B199B49FAA5F0A2EE238A6BC E8ED289DEBA952E4283B54E88E6183CA internet 1 111111

# Common
mongodb:
  enabled: true
  auth:
    enabled: false
smf:
  enabled: true
upf:
  enabled: true
webui:
  enabled: true

# 4G
hss:
  enabled: false
  mongodb:
    enabled: false
mme:
  enabled: false
pcrf:
  enabled: false
  mongodb:
    enabled: false
sgwc:
  enabled: false
sgwu:
  enabled: false

# 5G
amf:
  enabled: true
ausf:
  enabled: true
bsf:
  enabled: true
nrf:
  enabled: true
nssf:
  enabled: true
pcf:
  enabled: true
  mongodb:
    enabled: false
scp:
  enabled: true
  mongodb:
    enabled: false
udm:
  enabled: true
udr:
  enabled: true
  mongodb:
    enabled: false
```