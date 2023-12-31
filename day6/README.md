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
### final deployment manifest which is calling all the possible secrets

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
        envFrom: # for calling env and their values as well
        - secretRef:
            name: ashu-db-cred
        env: # to call env variable to put some data
        - name: MYSQL_DATABASE
          value: ashu-newdb # this db will be created automatically 
        - name: MYSQL_ROOT_PASSWORD # to set mysql admin password
          valueFrom: # reading password from some where
            secretKeyRef:
              name:  ashu-root-cred # name of secret 
              key:  pass1 # key of secret 
status: {}
```

### lets deploy it 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f .\db_deployment.yaml
deployment.apps/ashu-db created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  deploy 
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
ashu-db   1/1     1            1           47s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  pods
NAME                      READY   STATUS    RESTARTS   AGE
ashu-db-5bcdd76f4-zphmg   1/1     Running   0          51s
```
### creating clusterIP type service to only get Internal LB not external 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  deploy                                                             NAME      READY   UP-TO-DATE   AVAILABLE   AGE                                                                                                       ashu-db   1/1     1            1           27m                                                                                                       PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp>                                                                                 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl expose deployment ashu-db --type ClusterIP --port 3306 --name dblb --dry-run=client -o yaml  >dbsvc.yaml 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f .\dbsvc.yaml  
service/dblb created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  svc
NAME   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
dblb   ClusterIP   10.0.126.7   <none>        3306/TCP   3s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get ep
NAME   ENDPOINTS         AGE
dblb   10.244.0.3:3306   22s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get po -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP           NODE                                NOMINATED NODE   READINESS GATES
ashu-db-5bcdd76f4-zphmg   1/1     Running   0          29m   10.244.0.3   aks-agentpool-18505526-vmss000008   <none>           <none>
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 
```

## secret vs configmap in k8s

<img src="cm.png">

### creating configmap with key value options to store db name 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create configmap ashu-db-details --from-literal dbname=ashu-newdb --dry-run=client -o yaml
apiVersion: v1
data:
  dbname: ashu-newdb
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: ashu-db-details
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create configmap ashu-db-details --from-literal dbname=ashu-newdb --dry-run=client -o yaml   >dbnamecm.yaml 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f .\dbnamecm.yaml 
configmap/ashu-db-details created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get configmap 
NAME               DATA   AGE
ashu-db-details    1      8s
kube-root-ca.crt   1      4d
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get cm        
NAME               DATA   AGE
ashu-db-details    1      12s
kube-root-ca.crt   1      4d
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 

```

### updating deployment manifest with configmap calling 

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
        envFrom: # for calling env and their values as well
        - secretRef:
            name: ashu-db-cred
        env: # to call env variable to put some data
        - name: MYSQL_DATABASE
          valueFrom: # reading data from CM 
            configMapKeyRef:
              name: ashu-db-details
              key: dbname  
        - name: MYSQL_ROOT_PASSWORD # to set mysql admin password
          valueFrom: # reading password from some where
            secretKeyRef:
              name:  ashu-root-cred # name of secret 
              key:  pass1 # key of secret 
status: {}
```

### redeploy it 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl replace -f .\db_deployment.yaml --force
deployment.apps "ashu-db" deleted
deployment.apps/ashu-db replaced
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get deploy    
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
ashu-db   1/1     1            1           6s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get po    
NAME                      READY   STATUS    RESTARTS   AGE
ashu-db-f8645c785-bkgsp   1/1     Running   0          9s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get svc
NAME   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
dblb   ClusterIP   10.0.126.7   <none>        3306/TCP   26m
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get ep 
NAME   ENDPOINTS         AGE
dblb   10.244.0.4:3306   26m
```
### creating webapp deployment 

```
kubectl create deployment  ashu-webapp --image=wordpress:6.2.1-apache --port 80    --dry-run=client -o yaml  >web_deploy.yaml 
```

### update webapp deployment yaml to connect Database pod 

### creating envfile to store info 

### dbconn.env 

```
WORDPRESS_DB_HOST=dblb
WORDPRESS_DB_NAME=ashu-newdb
```

### creating configmap 

```
 C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create  configmap db-conn-cm --from-env-file=dbconn.env --dry-run=client  -o yaml  >dbconn.yaml                                                                                                                              
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f .\dbconn.yaml                                                 
configmap/db-conn-cm created                                                                                                                         
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get cm 
NAME               DATA   AGE
ashu-db-details    1      43m
db-conn-cm         2      4s
kube-root-ca.crt   1      4d1h
```

### store connect secret env -- dbconn-cred.env

```
WORDPRESS_DB_USER=ashu
WORDPRESS_DB_PASSWORD=Marlabs@098
```

### creating secret using above info 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create secret generic ashu-db-conn --from-env-file=dbconn-cred.env --dry-run=client -o yaml  >dbconn_secret.yaml                                                                                                             PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f .\dbconn_secret.yaml                                          secret/ashu-db-conn created                                                                                                                          
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl.exe  get  secret
NAME             TYPE                             DATA   AGE
ashu-db-conn     Opaque                           2      5s
ashu-db-cred     Opaque                           2      109m
ashu-reg-cred    kubernetes.io/dockerconfigjson   1      3d23h
ashu-root-cred   Opaque                           1      124m
```

### finally updating CM & secret to the deployment manifest file

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ashu-webapp
  name: ashu-webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ashu-webapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: ashu-webapp
    spec:
      containers:
      - image: wordpress:6.2.1-apache
        name: wordpress
        ports:
        - containerPort: 80
        resources: {}
        envFrom:
        - configMapRef:
            name: db-conn-cm 
        - secretRef:
            name: ashu-db-conn 
status: {}

```

### lets deploy it 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f .\web_deploy.yaml
deployment.apps/ashu-webapp created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
ashu-db       1/1     1            1           48m
ashu-webapp   0/1     1            0           6s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  pods 
NAME                           READY   STATUS              RESTARTS   AGE
ashu-db-f8645c785-bkgsp        1/1     Running             0          48m
ashu-webapp-69f7775f6f-rmlxg   0/1     ContainerCreating   0          9s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 
```

### creating service for webapp

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  deploy                                                             NAME          READY   UP-TO-DATE   AVAILABLE   AGE                                                                                                   ashu-db       1/1     1            1           73m                                                                                                   ashu-webapp   1/1     1            1           25m                                                                                                   
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl expose deployment ashu-webapp --type LoadBalancer --port 80 --name ashulb   --dry-run=client -o yaml >svcweb.yaml 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f .\svcweb.yaml 
service/ashulb created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  svc
NAME     TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
ashulb   LoadBalancer   10.0.174.111   <pending>     80:32141/TCP   5s
dblb     ClusterIP      10.0.126.7     <none>        3306/TCP       100m
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 



```

### creating LB type service for webapp

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  deploy                                                             NAME          READY   UP-TO-DATE   AVAILABLE   AGE                                                                                                   ashu-db       1/1     1            1           73m                                                                                                   ashu-webapp   1/1     1            1           25m                                                                                                   
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl expose deployment ashu-webapp --type LoadBalancer --port 80 --name ashulb   --dry-run=client -o yaml >svcweb.yaml 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl create -f .\svcweb.yaml 
service/ashulb created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  svc
NAME     TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
ashulb   LoadBalancer   10.0.174.111   <pending>     80:32141/TCP   5s
dblb     ClusterIP      10.0.126.7     <none>        3306/TCP       100m
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  svc
NAME     TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
ashulb   LoadBalancer   10.0.174.111   4.224.1.204   80:32141/TCP   51s
dblb     ClusterIP      10.0.126.7     <none>        3306/TCP       101m
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> kubectl get  svc
NAME     TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
ashulb   LoadBalancer   10.0.174.111   4.224.1.204   80:32141/TCP   85s
dblb     ClusterIP      10.0.126.7     <none>        3306/TCP       102m
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\ashu-two-tierapp> 
```


