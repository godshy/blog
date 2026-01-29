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
- Key pair: it is used to connect to the instance remotelly, I set one for master and one for works
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

Return to console, since by default security group only allow inbound SSH, we need to configure network access with each node. Go to security groups, click create new security group and specify a good name for each instance. Firstly allow ssh from anywhere, then add new inbound rules from other instances. For example SG for mmaster should allow traffic from worker1 and worker2. After configured all of the SGs, go to instance -> Actions -> Security -> Change security groups, 
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