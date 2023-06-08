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
    ports:
    - containerPort: 8080
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

### YAML description file for deployments

``` yaml
apiVersion: v1
kind: Deployment
metadata:
  name: my_deploy

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
      app: myapp
      type: front-end
```

### YAML description file for Services

- Nodeport
``` yaml
apiVsersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: NodePort # type of service
  ports:
  - targetPort: 80 # port want to access on the pod
    port: 80 # mapping port on service
    nodePort: 30008 # port expose to outside
  selector: # links the service to the pod by pod labels
    app: myapp
	  type: front-end
```

- Clusterip

``` yaml
apiVsersion: v1
kind: Service
metadata:
  name: back-end

spec:
  type: ClusterIP # type of service
  ports:
  - targetPort: 80 # port exposed from backends
    port: 80 # mapping port from service

  selector: # links the service to the pod by pod labels
    app: myapp
	  type: front-end
```


### YAML description file for NameSpace

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

### YAML description file for NameSpace quota

``` yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: [ns_name]

spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

