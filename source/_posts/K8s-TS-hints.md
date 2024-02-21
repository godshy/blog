---
title: K8s TS hints
date: 2023-06-05 16:00:30
tags: K8s
category: troubleshooting
---

# Provide some basic point of view for troubleshooting K8s

 ### ImagePullBackoff

1. This usually indicates something wrong with the pod description yaml file. Check name under `image: `
``` yaml
spec:
  containers:
  - name: redis
    image: redis123

```

### unable to recognize "something.yaml"

1. if "no matches for / , kind=ReplicaSet" . Possibly the apiVersion problem. Make sure it is ```apps/v1```
``` yaml
  apiVersion: apps/v1
```
### please apply your changes to the latest version and try again error

1. This error after ```kubectl apply``` indicates that you should acquire the newest yaml file after you did some changes to the parts without modyfing the yaml file.

``` bash
    kubectl get replicaset name_of_replicaset -o yaml >> something.yaml
    
    kubectl replace/apply -f something.yaml
```

### Do not add extra name after [pods_name] when creating pods imperatively

``` bash
  kubectl run <pods_name> <args> --image=<image_name>
```

### Things todo if kubectl command ceased responding

- check logs under ```/var/log/container```
- use ``` crictl ``` command 
``` bash
  crictl ps -a
  crictl logs container-id
  # or
  docker ps 
  docker logs
```