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
    command:
    - cmd
    - "args"
    # or 
    command: ["somecommand"]
    #
    args: ["some_args"]
    ports:
    - containerPort: 8080
  initContainers:
  - name: sth
    image: sth

  - name: sidecar  
    image: sidecar_image
```

ENV VAR for pods

``` yaml
spec:
  containers:
  - env:
    - name: ENV_VAR_NAME
      value: ENV_VAR_VALUE

```


### Template for configmap 

``` yaml
apiVersion: v1
kind: ConfigMap
metadata: 
  name: <ConfigMap_Name>
data:
  <key_name>: <value_name>

```

### Template for secret

``` yaml
# need to convert k-v data into coded form
# echo -n '<plain_text>' | base64
# echo -n '<secret_text>' | base64 --decode
apiVersion: v1
kind: Secret
metadata: 
  name: <secret_Name>
data:
  <key_name>: <value_name>
  <key_name_1>: <Pwd_1>

```
### Inject configmap and secrets to pods
``` yaml

spec:
  containers:
  - name:
    image:
    ports:
      - 
    envFrom:
      - configMapRef:
          name: <config_map_name>
        secretRef:
          name: <secret_name>
  
      

```
or
``` yaml
- env: 
  - name:
  valueFrom:
    configMapKeyRef:
      name: <config_map_name>
      key: <some_keys_of_cmap>
    secretKeyRef:
      name: <secret_name>
      key: <some_keys>
      
```




### YAML description file for replication-sets

``` yaml
apiVersion: apps/v1  # error: unable to recognize "filename.yml": no matches for /, kind=ReplicaSet appears if apiVersion was set to v1
kind: ReplicaSet
metadata:
  name: myrs
  namespace: [ns]

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
  namespace: [ns]

spec:
  template:
    metadata:[]
      name: myapp-pod
	    labels: 
	      app: myapp
		    type: front-end
	  spec:
      securityContext: # also can be put under "containers" to choose appling context in pod or container scope
        runAsUser: 1000 # define what user to run in the pod
        capabilities:
          add: ["MAC_ADMIN"]
	    containers:
	    - name: dockerhub_url/nginx-containers
		    image: nginx
      imagePullSecrets: # image security, docker hub username/pwd secret object
      - name: <docker-registry-secret-name>
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      type: front-end
```

### YAML description file for DaemonSet

``` yaml
apiVsesion: apps/v1
kind: DaemonSet
metadata:
  name: myrs
  namespace: [ns]

spec:
  template:
    metadata:
      name: myapp-pod
	    labels: 
	      app: myapp
		    type: front-end
	  spec:
	    containers:
	    - name: monitoring-agent
		    image: monitoring-agent
  selector: 
    matchLabels:
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

### Limit range for resource management

__only affect on creation__
``` yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: [name]
spec:
  limits:
  - default:
      cpu:
    defaultRequest:
      cpu: 
    max:
      cpu:
    min:
      cpu:
    type: Container  
```


### Allocate pods to deploy on modified scheduler
``` yaml
# on pod definition file
spec:
  schedulerName: <scheduler-name>

```

### Allocate the priority to be scheduled in the queue
``` yaml
kind: pod
spec:
  priorityClassName: high-priority
```

### description file for CSR
``` yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: somename
spec:
  expirationSeconds: 600
  usages:
  - digital signature
  - key encipherment
  - server auth
  request: <BASE64_encoded CSR_info>
```

### Definition file for kubeconfig
``` yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /path_to_ca_crt
    # or
    certificate-authority-data: <base64>
    server: https://controlplane:6443
  name: kubernetes

contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}

users:
- name: kubernetes-admin
  user:
    client-certificate-data: <base64>
    client-key-data: <base64>
```

### role definition file
- namespace-based resource role definition
``` yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: developer
    namespace: <namespace_name>
  rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list", "get", "create", "update"]
```
- cluster role definition file
``` yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: role_definition_name
  rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["list", "get", "create", "update"]  
```
### link user to role, role binding definition
- namespace-based resource role binding
``` yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: devuser-developer-binding
  subjects:
  - kind: User
    name: dev-user
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: developer
    apiGroup: rbac.authorization.k8s.io
```
- cluster role binding

``` yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: cluter-binding
  subjects:
  - kind: User
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: ClusterRole
    name: cluster-administrator
    apiGroup: rbac.authorization.k8s.io

```

### network policy definition
``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:  # or
        matchLabels:
          role: frontend
    ports:  # and 
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978

```


### Storage definition files
``` yaml
  spec:
    containers:
      volumeMounts:
      - mountPath: <dir_in_container> # directory in container which you need to store file
        name: <volume_name>
    volumes:
    - name: <volume_name>
      hostPath:
        path: <dir_mounted_path> # path outside container which you need to store data on 
        type: Directory
      # or
      persistentVolumeClaim:
        claimName: <pvclaim_name>
# this will cause directory created on every node in multi-node env
```

- persistent volume
``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle/Retain/Delete
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2

    # or
spec:
  hostPath:
    path: <dir_local_storage_path>
```

- persistent volume claim: bind to persistent volume
``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem 
  volumeName: <name_of_volume> # match what pv you want to bind
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}

```
__If a pvc is deleted before the deletion of pod, it will stuck in terminating state.__
- attach pvc to pod
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd # name should match with name under volumes
  volumes:
    - name: mypd  # should be same with name under volumeMounts
      persistentVolumeClaim:
        claimName: myclaim
```

- storage class: allowed for automatically provision volume
``` yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/example
parameters:
  type: pd-standard
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: topology.kubernetes.io/zone
    values:
    - us-central-1a
    - us-central-1b
```
then update pvc-definition file with:
``` yaml
spec:
  storageClassName: <SC_NAME>
```