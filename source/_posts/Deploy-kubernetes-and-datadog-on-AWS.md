---
title: Deploy kubernetes and datadog on AWS
date: 2026-01-29 14:15:27
tags: AWS, K8s
category: DIY and lab
---

# Deploy Kubernetes cluster on AWS, then perform monitoring and enable observability using datadog

In this blog, I want to test deploying a kubernetes with 2 worker nodes and 1 master node, worker nodes are deployed on different AZs.

I will firstly set up EC-2 with the dumbest way (set up manually with all handcrafts and then set up kubernetes manually) and set datadog to try monitor the kubernetes environment. Then try using terraform, ansible to make the process easier, and temper with some fancy parts(different deploy patterns, ELBs, other services) on AWS.



---
##
sudo hostnamectl set-hostname