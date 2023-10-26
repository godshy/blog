---
title: Deploy free5GC with k8s and helm
date: 2023-10-23 15:32:30
tags: k8s, helm
category: DIY and lab
---
# Install free5gc on personal pcs using k8s and helm

## Background

Very trivial and simple try to install and run free5gc on my personal PC by using Free5GC with Kubernetes, helm, calico and multus.

For the first step, I was planning to install free5gc
on one cluster and test it using UERANSIM.

As for next step I plan to deploy free5gc on multiple nodes environments in which master and worker node are separated. Also planning to deploy on openstack environment or AWS.

## Prepare

I was using vmware workstation and deployed ubuntu 20.6 image on it. Memory was set to around 10G to make sure calico works ok.

![VM info](https://s2.loli.net/2023/10/26/VRX3ncqWjINhxzY.png)

After ubuntu is up, do the install and update packages first.

``` bash
    :~$ sudo apt update -y
    :~$ sudo apt upgrade -y
    :~$ sudo apt install -y curl wget apt-transport-https
```

Then install and start up minikube, minikube enables user to set up kubernetes cluster on the system.

``` bash
    :~$ wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    :~$ sudo cp minikube-linux-amd64 /usr/local/bin/minikube
    :~$ sudo chmod +x /usr/local/bin/minikube
    # use calico as cni(container network interface) plugin 
    :~$ minikube start --driver=docker --cpus=4 --memory=8g --disk-size=20g --cni=calico

    :~$ minikube status

        minikube
        type: Control Plane
        host: Running
        kubelet: Running
        apiserver: Running
        kubeconfig: Configured
```

![minikube status](https://s2.loli.net/2023/10/26/gJeHqMjoGhIONCR.png)


After minikube was installed, install kubectl for better experience. 

``` bash
    :~$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
    # If exeucte the command directly, it may have warning regard to minikube and kubectl version mismatch. To avoid this, you can designate version of kubectl to match with minikube
    :~$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.26.3/bin/linux/amd64/kubectl
    :~$ chmod +x kubectl
    :~$ sudo mv kubectl /usr/local/bin/
    :~$ kubectl version -o yaml

```

![Installed kubectl](https://s2.loli.net/2023/10/26/HkLGdtBNwoVfFCc.png)

Then for the last step of preparation, we need to install helm and multus-CNI plugin to enable multiple interface support for pods.


``` bash
    :~$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
    :~$ chmod 700 get_helm.sh
    :~$ ./get_helm.sh
    :~$ helm list -A
    :~$ git clone https://github.com/k8snetworkplumbingwg/multus-cni.git
    :~$ cd multus-cni
    :~$ cat ./deployments/multus-daemonset-thick.yml | kubectl apply -f -
    :~$ kubectl get pods -n kube-system
```

For me I had multus pods oom killed, so I modified the request and limit resource in the multus-daemonset-thick.yml then pod went up fine.

```yaml
      containers:
        - name: kube-multus
          image: ghcr.io/k8snetworkplumbingwg/multus-cni:snapshot-thick
          command: [ "/usr/src/multus-cni/bin/multus-daemon" ]
          resources:
            requests:
              cpu: "100m"
              memory: "50Mi" --> "250Mi"
            limits:
              cpu: "100m"
              memory: "50Mi" --> "350Mi"
          securityContext:
            privileged: true
          volumeMounts:
            - name: cni
              mountPath: /host/etc/cni/net.d
```

## Free5gc install

Main step of installing 5gc is to use helm to pull repository from free5gc git, then install the helm chart.

```bash
    # create namespace 
    :~$ kubectl create namespace free5gc
    # add helm repository
    :~$ helm repo add towards5gs 'https://raw.githubusercontent.com/Orange-OpenSource/towards5gs-helm/main/repo/'
    :~$ helm repo update
    :~$ helm repo list
    :~$ helm search repo
```


![Helm install](https://s2.loli.net/2023/10/26/oF6m4k1uTtsqc3d.png)


After checking the helm repository, it is ok to procced with installing free5gc.

```bash

    :~$ helm -n free5gc install free5gc towards5gs/free5gc
    :~$ kubectl get pods -n free5gc
    
```


![pod install complete](https://s2.loli.net/2023/10/26/qcLoV1daCsj8TSR.png)


**Upf pod cannot set up and will stuck in creating state because multus cannot add eth1 to this pod.**


## References

- [towards5gs-helm-github](https://github.com/Orange-OpenSource/towards5gs-helm)
- [Deploying 5G core network with Free5GC, Kubernetes and Helm](https://medium.com/rahasak/deploying-5g-core-network-with-free5gc-kubernets-and-helm-charts-29741cea3922)
- [Kubernetes環境でfree 5GCを動かすまで](https://medium.com/rahasak/deploying-5g-core-network-with-free5gc-kubernets-and-helm-charts-29741cea3922) 
- [kubernetes cluster上にfree5gcを構築してみる](https://qiita.com/wzm/items/0dbe928ed891d82380d6)
