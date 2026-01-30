---
title: Deploy kubernetes and datadog on AWS
date: 2026-01-29 14:15:27
tags: AWS, K8s
category: DIY and lab
---

# Deploy Kubernetes cluster on AWS, then perform monitoring and enable observability using datadog

In this blog, I want to test deploying a kubernetes with 2 worker nodes and 1 master node, worker nodes are deployed on different AZs.

I will firstly set up EC-2 with the dumbest way (set up manually with all handcrafts and then set up kubernetes manually) and set datadog to try monitor the kubernetes environment. Then try using terraform, ansible to make the process easier, and temper with some fancy parts(different deploy patterns, ELBs, other services) on AWS.




## Set up EC2 instance
Launch EC2 instance first, open console, search for EC2 and click Launch instances

Configure following:
- Name and tages: Master x1 and Worker x2
- Application and OS image: I chose Ubuntu, but any other distribution is OK
- Key pair: it is used to connect to the instance remotely, I set one for master and one for works
- Network settings: For one worker, change Subnet to let it placed in a different AZ from other instances. Other parameters just leave them default here. (For security group, it will create security group and allow inbound SSH only in default we can change it later.) 

Others leave the same. And click launch

![ec2 launch.png](https://s2.loli.net/2026/01/29/mtqsnkbhXKY4AVR.png)

![ec2 launch 2.png](https://s2.loli.net/2026/01/29/d2pwOvitNB3berf.png)

After launch successful, we should see all three instance are up and running. By default, these 3 instances are attached with public ipv4 ip address and attached default internet gateway, so we can ssh to them using key pair created previously. Notice that one worker's AZ is different from the others.

![1.1 3 ec2 launched on aws.png](https://s2.loli.net/2026/01/29/senbyGNAwap6VCu.png)


Then we connect to them using terminal and do some initial configurations:

``` bash
    sudo apt-get update
    sudo apt-get upgrade
    sudo hostnamectl set-hostname Master1/Worker1/Worker2
    sudo reboot

```

Return to console, since by default security group only allow inbound SSH, we need to configure network access with each node. Go to security groups, click create new security group and specify a good name for each instance. Firstly allow ssh from anywhere, then add new inbound rules from other instances. For example SG for master should allow traffic from worker1 and worker2. After configured all of the SGs, go to instance -> Actions -> Security -> Change security groups, 
Unbind the default SG and attach created new SG to it.

![SG.png](https://s2.loli.net/2026/01/29/TVmNnW3XL7BFMdh.png)

![sg configure.png](https://s2.loli.net/2026/01/29/v7GNjkq9sPD1WyR.png)

![SG detach.png](https://s2.loli.net/2026/01/29/6HoTKMNCDcxfbwp.png)

Check ping for each instance, no ping should get through.

Modify ```/etc/hosts```. add hostname for each other. So that each instance can ping with others with their hostname.

***Important things to be noticed:***
- Since there is no Elastic IP attached, after instance is relaunched thru console, Public IP will change
- Micro EC2 (mem under 2048) may have some problem when installing kubernetes control plane components. We may need to change instance type for larger memory.

Refer to [Install k8s on vm](https://godshy.github.io/blog/2026/01/05/Deploy-k8s-cluster/) for the cluster installation.

After all installation process finished, check using ``` kubectl get nodes -o wide  ``` to make sure every node is on and ready.


## Create and pull some container to be used as workload on demo cluster
We can use existing container image from docker hub or even helm to deploy workload on the cluster, but since we are doing things in the dumbest way, I will create a sample web server from scratch also.

### Create a web server using go

- Create a folder, add source file main.go with codes below:

``` go
package main
import (
"fmt"
"log"
"net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
fmt.Fprintln(w, "Hello world!")
}
func main() {
http.HandleFunc("/", handler)
log.Fatal(http.ListenAndServe(":8080", nil))
}
```

- Add a Docker file with contents below:

``` bash
FROM golang:1.11-alpine AS build
WORKDIR /src/
COPY main.go go.* /src/
RUN CGO_ENABLED=0 go build -o /bin/demo
FROM scratch
COPY --from=build /bin/demo /bin/demo
ENTRYPOINT ["/bin/demo"]
```

- Build the docker image:

``` bash
(base) esnhhoe@E-5CG3504J46:~/demo_webserver$ sudo docker build -t shy_webserver .
[sudo] password for esnhhoe:
[+] Building 33.3s (10/10) FINISHED                                                              docker:default
 => [internal] load build definition from Dockerfile                                                       0.1s
 => => transferring dockerfile: 226B                                                                       0.0s
 => [internal] load metadata for docker.io/library/golang:1.11-alpine                                     11.5s
 => [internal] load .dockerignore                                                                          0.0s
 => => transferring context: 2B                                                                            0.0s
 => [build 1/4] FROM docker.io/library/golang:1.11-alpine@sha256:09e47edb668c2cac8c0bbce113f2f72c97b1555  18.3s
 => => resolve docker.io/library/golang:1.11-alpine@sha256:09e47edb668c2cac8c0bbce113f2f72c97b1555d70546d  0.0s
 => => sha256:65ba82af53f74920766ed58eab751a13eec4f7297e0bca6854e13fecc7523cca 126B / 126B                 0.4s
 => => sha256:ec448270508e1a0c0710953a409b9bc9969c1766eefc52e91e3c5ae9ce393a41 110.29MB / 110.29MB        16.1s
 => => sha256:9fe9984849c103d07696bf941dcae6df23b652efdaedf3d804baeede686e8faf 154B / 154B                 0.8s
 => => sha256:7f94eaf8af200ac18deb367dab5fb993b8ee609611a0493aa4adc287f8c682f7 301.73kB / 301.73kB         0.9s
 => => sha256:9d48c3bd43c520dc2784e868a780e976b207cbf493eaff8c6596eb871cbd9609 2.79MB / 2.79MB             1.1s
 => => extracting sha256:9d48c3bd43c520dc2784e868a780e976b207cbf493eaff8c6596eb871cbd9609                  0.1s
 => => extracting sha256:7f94eaf8af200ac18deb367dab5fb993b8ee609611a0493aa4adc287f8c682f7                  0.1s
 => => extracting sha256:9fe9984849c103d07696bf941dcae6df23b652efdaedf3d804baeede686e8faf                  0.0s
 => => extracting sha256:ec448270508e1a0c0710953a409b9bc9969c1766eefc52e91e3c5ae9ce393a41                  2.0s
 => => extracting sha256:65ba82af53f74920766ed58eab751a13eec4f7297e0bca6854e13fecc7523cca                  0.0s
 => [internal] load build context                                                                          0.1s
 => => transferring context: 263B                                                                          0.0s
 => [build 2/4] WORKDIR /src/                                                                              0.6s
 => [build 3/4] COPY main.go go.* /src/                                                                    0.0s
 => [build 4/4] RUN CGO_ENABLED=0 go build -o /bin/demo                                                    2.2s
 => [stage-1 1/1] COPY --from=build /bin/demo /bin/demo                                                    0.0s
 => exporting to image                                                                                     0.5s
 => => exporting layers                                                                                    0.3s
 => => exporting manifest sha256:72ea11f50031639bdffc645e1988b315be984e8940eaf114da2c7dd425956518          0.0s
 => => exporting config sha256:13e4bade4069480fa42105bafb3e602f25c9bb0cb3e09ff87a2a23d2c33dd506            0.0s
 => => exporting attestation manifest sha256:fde63a4a2f529414d5f8c4f1d1591d7da07bca39718ca88dcf15ae2a3ace  0.0s
 => => exporting manifest list sha256:585f1425d316732b8b19a5b9a17d9d41319a7256e4224cb6728239557201b268     0.0s
 => => naming to docker.io/library/shy_webserver:latest                                                    0.0s
 => => unpacking to docker.io/library/shy_webserver:latest                                                 0.1s

(base) esnhhoe@E-5CG3504J46:~/demo_webserver$ sudo docker image ls
                                                                                            i Info →   U  In Use
IMAGE                  ID             DISK USAGE   CONTENT SIZE   EXTRA
ilo_tool:latest        84b1c86c6314        106MB         26.3MB    U
rhel8mim:latest        463c4b430227        813MB          227MB    U
shy_webserver:latest   585f1425d316        9.8MB         3.29MB

```


- Retag, login to my own docker hub, and push it to my docker repository:

``` bash
(base) esnhhoe@E-5CG3504J46:~/demo_webserver$ sudo docker login
(base) esnhhoe@E-5CG3504J46:~/demo_webserver$ sudo docker image tag shy_webserver:latest hyshen/shy_webserver

(base) esnhhoe@E-5CG3504J46:~/demo_webserver$  sudo docker image push hyshen/shy_webserver:latest
The push refers to repository [docker.io/hyshen/shy_webserver]
b50b3df2e3db: Pushed
b1f4ec0bc773: Pushed
latest: digest: sha256:585f1425d316732b8b19a5b9a17d9d41319a7256e4224cb6728239557201b268 size: 855
```

### Deploy workload on cluster
Go back to aws console, login to master node and do a pull from docker hub
``` bash
ubuntu@ec2-master:~$ sudo docker image pull hyshen/shy_webserver
Using default tag: latest
latest: Pulling from hyshen/shy_webserver
b50b3df2e3db: Pull complete
b1f4ec0bc773: Download complete
Digest: sha256:585f1425d316732b8b19a5b9a17d9d41319a7256e4224cb6728239557201b268
Status: Downloaded newer image for hyshen/shy_webserver:latest
docker.io/hyshen/shy_webserver:latest
ubuntu@ec2-master:~$

ubuntu@ec2-master:~$ sudo docker image ls
                                                                                                               i Info →   U  In Use
IMAGE                         ID             DISK USAGE   CONTENT SIZE   EXTRA
hyshen/shy_webserver:latest   585f1425d316        9.8MB         3.29MB
ubuntu@ec2-master:~$

```

Label each nodes in advance to make sure workload are scheduled on worker node instead of master. And also, create namespaces:

``` bash
ubuntu@ec2-master:~$ kubectl label node ec2-master nodetype=controlplane
node/ec2-master labeled
ubuntu@ec2-master:~$ kubectl label node worker1 nodetype=workernode
node/worker1 labeled
ubuntu@ec2-master:~$ kubectl label node worker2 nodetype=workernode

ubuntu@ec2-master:~$ kubectl create ns webserver
namespace/webserver created
```

Create a yaml file as deployment like below:
PS. I used nodeAffinity to make sure pod are scheduled on worker node, but 
``` yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: shy-webserver
  namespace: webserver
  labels:
    app: serverWorkload
spec:
  replicas: 2
  selector:
    matchLabels:
      app: go
  template:
    metadata:
      labels:
        app: go
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: nodetype
                operator: In
                values:
                - workernode
      containers:
      - name: helloworld
        image: hyshen/shy_webserver:latest
        ports:
        - containerPort: 8080

```

Do the following command to deploy and check pod status:

``` bash
ubuntu@ec2-master:~/definitions$ kubectl apply -f webserver.yaml
ubuntu@ec2-master:~/definitions$ kubectl get pod -n webserver -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP                NODE      NOMINATED NODE   READINESS GATES
shy-webserver-5dc84f46b4-hnln5   1/1     Running   0          20s   192.168.189.67    worker2   <none>           <none>
shy-webserver-5dc84f46b4-k5zvz   1/1     Running   0          20s   192.168.235.131   worker1   <none>           <none>

```

To make container be able to access from outside the node, we can expose by creating a nodeport service. For more advanced one like loadbalancer and how to combine it with AWS built-in load balancer, we will discuss it later.

``` bash
80
service/shy-webserver exposed
ubuntu@ec2-master:~/definitions$ kubectl get service -n webserver
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
shy-webserver   NodePort   10.102.72.252   <none>        8080:30314/TCP   8s

ubuntu@ec2-master:~/definitions$ curl 10.102.72.252:8080
Hello world!

```

Now the basic workload is created, let us deploy monitoring (datadog, grafana) for it.

## Deploy datadog agent, prometheus grafana 

Deploy a metric server to able able to observe metrics on pods and nodes:
``` bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Sign up for datadog trail and install datadog agent
Datadog is a charged monitoring and observation platform to monitor kubernetes platforms, hosts and cloud native workloads. I here created a 14 day trail to test deployment of datadog agent.

- Firstly sign up for trail account using company mail:
![datadog sign up.png](https://s2.loli.net/2026/01/30/7Un6xD8TGSRoHgk.png)

- Login using the provided mail account, you will be prompted to install agent by your workload type, we want to try both kubernetes based and host based, so we install both.

![datadog start.png](https://s2.loli.net/2026/01/30/CLbKt2Agh6D7sVF.png)


#### Datadog agent installation on host

- Select host based linux
![agent on linux.png](https://s2.loli.net/2026/01/30/AhK9WGTOLaZtv3s.png)

- Choose env, this will change ```DD_ENV``` parameter. I chose dev

- Run install command on my master host and check if it is running:
``` bash
ubuntu@ec2-master:~$ DD_API_KEY=<API_KEY> \
DD_SITE="ap1.datadoghq.com" \
bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"

## SKIP install output..
/usr/bin/systemctl
* Starting the Datadog Agent...

  Your Datadog Agent is running and functioning properly.
  It will continue to run in the background and submit metrics to Datadog.
  If you ever want to stop the Datadog Agent, run:

      sudo systemctl stop datadog-agent

  And to run it again run:

      sudo systemctl start datadog-agent

  Consider adding dd-agent to the docker group to enable the docker support, run:

      sudo usermod -a -G docker dd-agent


ubuntu@ec2-master:~$ sudo systemctl status datadog-agent
● datadog-agent.service - Datadog Agent
     Loaded: loaded (/usr/lib/systemd/system/datadog-agent.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-01-30 08:26:53 UTC; 1min 17s ago
   Main PID: 97871 (agent)
      Tasks: 9 (limit: 2213)
     Memory: 134.1M (peak: 134.9M)
        CPU: 2.470s
     CGroup: /system.slice/datadog-agent.service
             └─97871 /opt/datadog-agent/bin/agent/agent run -p /opt/datadog-agent/run/agent.pid


```

I checked log of datadog-agent using journalctl command, there were a lot of following errors:

``` bash
journal -f -u datadog-agent
Jan 30 08:37:11 ec2-master agent[104343]: 2026-01-30 08:37:11 UTC | CORE | WARN | (pkg/collector/corechecks/system/disk/diskv2/disk.go:567 in getPartitionUsage) | Unable to get disk metrics for /run/containerd/io.containerd.grpc.v1.cri/sandboxes/80024b9ad338cedc7522392eb82d97e3106c01014a5efe273a230690e24085f0/shm: permission denied. You can exclude this mountpoint in the settings if it is invalid.
```

Do the following to fix:

``` bash
cd /etc/datadog-agent/conf.d/disk.d
cp /etc/datadog-agent/conf.d/disk.d/conf.yaml.default /etc/datadog-agent/conf.d/disk.d/conf.yaml
vim conf.yaml
```
Find ```file_system_global_exclude``` commented out section and un comment, add following **tmpfs** and **overlay**, then perform a restart:
``` yaml
root@ec2-master:/etc/datadog-agent/conf.d/disk.d# cat conf.yaml
## All options defined here are available to all instances.
#
init_config:

    ## @param file_system_global_exclude - list of strings - optional
    ## Instruct the check to always add these patterns to `file_system_exclude`.
    ##
    ## WARNING: Overriding these defaults could negatively impact your system or
    ## the performance of the check.
    #
     file_system_global_exclude:
       - tmpfs
       - overlay


```

Do a restart and check logs again, the errors are gone.
``` bash
systemctl restart datadog-agent
journactl -f -u datadog-agent
```


Another problem I met is 
![stuck.png](https://s2.loli.net/2026/01/30/6e4njV5OwYlMZtT.png)
Agent cannot connect to datadog site..
I am using ap1.datadoghq.com but looks like ap1 is not reachable...
So I changed to other datadog site
``` bash

root@ec2-master:/etc/datadog-agent# cat datadog.yaml | grep -i ap1
## Set to 'ap1.datadoghq.com' to send data to the AP1 site.
site: ap1.datadoghq.com
## The site parameter must be set to enable your agent with Remote Configuration.
## Set to 'datadoghq.eu' to send data to the EU site.
## Set to 'us3.datadoghq.com' to send data to the US3 site.
## Set to 'us5.datadoghq.com' to send data to the US5 site.
## Set to 'ap1.datadoghq.com' to send data to the AP1 site.
## Set to 'ddog-gov.com' to send data to the US1-FED site.

PING ap1.datadoghq.com (43.206.164.131) 56(84) bytes of data.
^C
--- ap1.datadoghq.com ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

root@ec2-master:/etc/datadog-agent# ping us5.datadoghq.com
PING us5.datadoghq.com (34.149.66.147) 56(84) bytes of data.
64 bytes from 147.66.149.34.bc.googleusercontent.com (34.149.66.147): icmp_seq=1 ttl=114 time=1.03 ms
64 bytes from 147.66.149.34.bc.googleusercontent.com (34.149.66.147): icmp_seq=2 ttl=114 time=1.02 ms
^C

```