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

