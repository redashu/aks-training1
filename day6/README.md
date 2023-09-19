### Revision 

## Start AKS cluster and verify 

```
PS C:\Users\humanfirmware> kubectl get nodes
NAME                                STATUS   ROLES   AGE     VERSION
aks-agentpool-18505526-vmss000008   Ready    agent   7m8s    v1.26.6
aks-agentpool-18505526-vmss000009   Ready    agent   6m59s   v1.26.6
PS C:\Users\humanfirmware> kubectl  get  ns
NAME              STATUS   AGE
ashu-project      Active   3d22h
calico-system     Active   7d
default           Active   7d
kube-node-lease   Active   7d
kube-public       Active   7d
kube-system       Active   7d
tigera-operator   Active   7d
xyz-dbspace       Active   3d22h
PS C:\Users\humanfirmware> kubectl config get-contexts
CURRENT   NAME                          CLUSTER         AUTHINFO                                 NAMESPACE
*         aks-ashutoshh                 aks-ashutoshh   clusterUser_aks-training_aks-ashutoshh   ashu-project
```

## Deploying two tier webapp

### creating mysql db deployment manifest

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ashu-db
  name: ashu-db # name of deployment 
spec:
  replicas: 1 # number of db pod 
  selector:
    matchLabels:
      app: ashu-db
  strategy: {}
  template: # pod template 
    metadata:
      creationTimestamp: null
      labels: # label of pods 
        app: ashu-db
    spec:
      containers:
      - image: mysql:8.0 # image from docker hub 
        name: mysql
        ports:
        - containerPort: 3306
        resources: {}
status: {}
```

### updating ENV section 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ashu-db
  name: ashu-db # name of deployment 
spec:
  replicas: 1 # number of db pod 
  selector:
    matchLabels:
      app: ashu-db
  strategy: {}
  template: # pod template 
    metadata:
      creationTimestamp: null
      labels: # label of pods 
        app: ashu-db
    spec:
      containers:
      - image: mysql:8.0 # image from docker hub 
        name: mysql
        ports:
        - containerPort: 3306
        resources: {}
        env: # to call env variable to put some data
        - name: MYSQL_DATABASE
          value: ashu-newdb # this db will be created automatically 
status: {}
```
### to store database root cred we are going to secret 

```
kubectl create  secret generic ashu-root-cred --from-literal  pass1=Hellodb@123    --dry-run=client -o yaml 
apiVersion: v1
data:
  pass1: SGVsbG9kYkAxMjM=
kind: Secret
metadata:
  creationTimestamp: null
  name: ashu-root-cred
```

### store in file and send create request

```
S C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f .\secretroot.yaml 
secret/ashu-root-cred created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get secret 
NAME             TYPE                             DATA   AGE
ashu-reg-cred    kubernetes.io/dockerconfigjson   1      3d21h
ashu-root-cred   Opaque                           1      5s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 


```

### incase of multiple cred details better to use env file

## db-cred.env

```
MYSQL_USER=ashu
MYSQL_PASSWORD=Marlabs@098
```

### creating secret using above env file

```
kubectl create  secret generic ashu-db-cred --from-env-file=db-cred.env --dry-run=client -o yaml
apiVersion: v1
data:
  MYSQL_PASSWORD: TWFybGFic0AwOTg=
  MYSQL_USER: YXNodQ==
kind: Secret
metadata:
  creationTimestamp: null
  name: ashu-db-cred
```

### store this in a file  db-secret.yaml

```
apiVersion: v1
data:
  MYSQL_PASSWORD: TWFybGFic0AwOTg=
  MYSQL_USER: YXNodQ==
kind: Secret
metadata:
  creationTimestamp: null
  name: ashu-db-cred
```

### creating secret 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f  .\db-secret.yaml
secret/ashu-db-cred created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get secrets
NAME             TYPE                             DATA   AGE
ashu-db-cred     Opaque                           2      7s
ashu-reg-cred    kubernetes.io/dockerconfigjson   1      3d21h
ashu-root-cred   Opaque                           1      15m
```

