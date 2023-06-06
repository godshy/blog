---
title: k8s templates
tags: K8s
date: 2023-06-05 18:07:29
---


# Config templates for K8s

### YAML description file for pods

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels: 
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

### YAML description file for replication-controllers

``` yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myrs
  labels: 
    app: myapp
	  type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
	    labels: 
	      app: myapp
		    type: front-end
	  spec:
	    containers:
	    - name: nginx-containers
		    image: nginx
  replicas: 3
```


### YAML description file for replication-sets

``` yaml
apiVersion: apps/v1  # error: unable to recognize "filename.yml": no matches for /, kind=ReplicaSet appears if apiVersion was set to v1
kind: ReplicaSet
metadata:
  name: myrs
  labels: 
    app: myapp
	  type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
	    labels: 
	      app: myapp
		    type: front-end
	  spec:
	    containers:
	    - name: nginx-containers
		    image: nginx
  replicas: 3
  selector: 
    matchLabels:
      type: front-end
```