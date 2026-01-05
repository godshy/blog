---
title: Deploy kubernetes cluster on virtual machine
date: 2026-01-05 21:44:58
tags: K8s
---
## Set up VMs

> [!NOTE]
> One Network Adapter for internet, the other is for  connection between Nodes


![VM spec](https://s2.loli.net/2026/01/05/ZuVatjDWwBnSUbT.png)

After VMs are on and running, make sure two VM can ping each other,  edit etc/hosts to let them resolve each other's hostname:


``` bash
# On master
vim /etc/hosts
192.168.168.129 worker

# On worker 
vim /etc/hosts
192.168.168.128 master

# then ping using hostname:
master:$ ping worker # should be ok
worker:$ ping master # should be ok
```

then execute following commands:

On Master and Worker:

``` bash
	sudo apt-get update
	sudo apt-get upgrade -y
	# check your hostname:
	uname -n
	
	# disable swapoff for better performance and best practice:
	sudo swapoff -a
	sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
	
	# Add settings to containerd.conf  
	# overlay (for using overlayfs),  
	# br_netfilter (for ipvlan, macvlan, external SNAT of service IPs)  
	sudo tee /etc/modules-load.d/containerd.conf <<EOF  
	overlay  
	br_netfilter  
	EOF


```


``` bash
    # Add settings to kubernetes.conf  
	# Allow IPv4, IPv6 and IP forwarding  
	sudo tee /etc/sysctl.d/kubernetes.conf <<EOF  
	net.bridge.bridge-nf-call-ip6tables = 1  
	net.bridge.bridge-nf-call-iptables = 1  
	net.ipv4.ip_forward = 1  
	EOF
```

``` bash

	# Reload updated config  
	sudo sysctl --system  
	  
	# Install required tools and CA certificates  
	sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates  
	  
	# Add Docker repository  
	sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg  
	sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

	# Then, install containerd  
	sudo apt update  
	sudo apt install -y containerd.io  
	containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1  
	sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml  
	sudo systemctl restart containerd  
	sudo systemctl enable containerd

	# Add k8s repository
	# Please modify v1.32 to latest from https://kubernetes.io/releases/
	sudo mkdir -p /etc/apt/keyrings
	curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
	echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Then, install Kubernetes components  
	sudo apt update  
	sudo apt install -y kubelet kubeadm kubectl

# Add port granted for etcd:
	sudo iptables -A INPUT -p tcp --dport 6443 -j ACCEPT
# k8s cluster nodes  
	sudo apt-get install docker-ce

	sudo reboot
```

### Set up master and worker and CNI plugin

#### On Master


``` bash
	sudo kubeadm config images pull
	sudo kubeadm init --pod-network-cidr=192.168.0.0/16
	
	
I0602 21:23:23.755053    2030 version.go:261] remote version is much newer: v1.33.1; falling back to: stable-1.32
[config/images] Pulled registry.k8s.io/kube-apiserver:v1.32.5
[config/images] Pulled registry.k8s.io/kube-controller-manager:v1.32.5
[config/images] Pulled registry.k8s.io/kube-scheduler:v1.32.5
[config/images] Pulled registry.k8s.io/kube-proxy:v1.32.5
[config/images] Pulled registry.k8s.io/coredns/coredns:v1.11.3
[config/images] Pulled registry.k8s.io/pause:3.10
[config/images] Pulled registry.k8s.io/etcd:3.5.16-0
shy@shymaster:~$ sudo kubeadm init
I0602 21:24:42.138404    2276 version.go:261] remote version is much newer: v1.33.1; falling back to: stable-1.32
[init] Using Kubernetes version: v1.32.5
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
W0602 21:24:42.808265    2276 checks.go:846] detected that the sandbox image "registry.k8s.io/pause:3.8" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.10" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local shymaster] and IPs [10.96.0.1 11.0.1.130]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost shymaster] and IPs [11.0.1.130 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost shymaster] and IPs [11.0.1.130 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 502.872747ms
[api-check] Waiting for a healthy API server. This can take up to 4m0s
[api-check] The API server is healthy after 7.00102198s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node shymaster as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node shymaster as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: qbqna4.vcib69rginbjmgi3
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join ....

# Follow the instructions:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install cni 
# Find latest version on https://github.com/projectcalico/calico/releases
	kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.1/manifests/calico.yaml
```


#### On worker

``` bash
# just copy and paste kubeadm join .... command from kube init output on master node, it will automatically join the master node
	kubeadm join ....
```

### Troubleshooting:
- Your node will not be ready unless you successfully installed CNI plugin, because your pod cannot communicate outside node. To check issue, run following command:

``` bash
	journalctl -u kubelet
	journalctl -u containerd
```

Example of the finish state:
![Final result](https://s2.loli.net/2026/01/05/2JkmfQElny6X5z1.png)