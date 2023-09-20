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
