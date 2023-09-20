### Revision 

<img src="rev.png">

### ephemral nature of container / pod 

<img src="st1.png">

### Introduction to Container storage Interface (CSI)

<img src="csi.png">

### some CNCF standard project for k8s 

<img src="std.png">


### CSI 

<img src="st111.png">

### storage sources 

<img src="ss.png">

## Implementing Internal storage in mysql pod 

### pod manifest 
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ashudb
  name: ashudb
spec:
  containers:
  - image: mysql:8.01
    name: ashudb
    ports:
    - containerPort: 3306
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
### updating with emptyDir volume 

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ashudb
  name: ashudb
spec:
  volumes: # to create volumes 
  - name: ashu-volume1 
    emptyDir: {} # k8s nodes will give a random directory inside them 
  containers: # to create contaienrs 
  - image: mysql:8.01
    name: ashudb
    ports:
    - containerPort: 3306
    resources: {}
    volumeMounts: # to mount volume inside pod container 
    - name: ashu-volume1 
      mountPath: /var/lib/mysql/ 
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### final mysql yaml 

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ashudb
  name: ashudb
spec:
  volumes: # to create volumes 
  - name: ashu-volume1 
    emptyDir: {} # k8s nodes will give a random directory inside them 
  containers: # to create contaienrs 
  - image: mysql:8.0
    name: ashudb
    ports:
    - containerPort: 3306
    resources: {}
    env: 
    - name: MYSQL_ROOT_PASSWORD
      value: RedDb@1234
    volumeMounts: # to mount volume inside pod container 
    - name: ashu-volume1 
      mountPath: /var/lib/mysql/ 
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### deploy it 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl replace -f  .\pod1.yaml --force
pod "ashudb" deleted
pod/ashudb replaced
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl get  pods  
NAME     READY   STATUS    RESTARTS   AGE
ashudb   1/1     Running   0          6s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 

```

### accessing database pod 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl get  pods        
NAME     READY   STATUS    RESTARTS   AGE
ashudb   1/1     Running   0          2m2s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl  exec -it  ashudb -- bash 
bash-4.4#
bash-4.4# 
bash-4.4# 
bash-4.4# mysql -u root -p
Enter password: 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
bash-4.4# mysql -u root -pRedDb@1234
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.34 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> create database hello;
Query OK, 1 row affected (0.02 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| hello              |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> exit;
Bye
bash-4.4# exit
exit
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 
```

### understanding usage of multi container pod 

<img src="mc.png">

### demo of deployment with single volume and single container 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ashudb
  name: ashudb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ashudb
  strategy: {}
  template: # pod template
    metadata:
      creationTimestamp: null
      labels:
        app: ashudb
    spec:
      volumes: 
      - name: ashu-dbvol
        hostPath:  # is for defining storage options in nodes 
          path: /ashu/data
          type: DirectoryOrCreate 
      containers:
      - image: mysql:8.0
        name: mysql
        ports:
        - containerPort: 3306
        resources: {}
        volumeMounts: # mounting volume created above 
        - name: ashu-dbvol 
          mountPath: /var/lib/mysql/ # default mysql db location 
        env: 
        - name: MYSQL_ROOT_PASSWORD
          value: RedDb@1234
status: {}

```

### deploy it

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl  create -f .\db_deploy.yaml                                                 deployment.apps/ashudb created                                                                                                             PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps>                                                                                PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl.exe  get  pod
NAME                      READY   STATUS    RESTARTS   AGE
ashudb-7d78fbd578-t7ckt   1/1     Running   0          4s
```

### adding another container

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ashudb
  name: ashudb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ashudb
  strategy: {}
  template: # pod template
    metadata:
      creationTimestamp: null
      labels:
        app: ashudb
    spec:
      volumes: 
      - name: ashu-dbvol
        hostPath:  # is for defining storage options in nodes 
          path: /ashu/data
          type: DirectoryOrCreate 
      containers:
      - image: alpine 
        name: ashuc1 
        command: ['sh','-c','sleep 10000'] 
        volumeMounts: 
        - name: ashu-dbvol 
          mountPath: /mnt/data
          readOnly: True 
      - image: mysql:8.0
        name: mysql
        ports:
        - containerPort: 3306
        resources: {}
        volumeMounts: # mounting volume created above 
        - name: ashu-dbvol 
          mountPath: /var/lib/mysql/ # default mysql db location 
        env: 
        - name: MYSQL_ROOT_PASSWORD
          value: RedDb@1234
status: {}

```

### deploy it

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl replace -f .\db_deploy.yaml
deployment.apps/ashudb replaced
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl get  pod
NAME                     READY   STATUS    RESTARTS   AGE
ashudb-68656bb49-c562r   2/2     Running   0          35s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl.exe  get  deploy
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
ashudb   1/1     1            1           8m12s
```


