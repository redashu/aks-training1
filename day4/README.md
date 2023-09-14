### Revision 

<img src="rev.png">

### on client machine location where k8s control plane creds are stored

<img src="cred.png">

### controller in k8s 

<img src="contr.png">

## Introduction to RC 

<img src="rc.png">

### sample RC manifest file 

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: ashu-rc1 # name of my RC 
spec:
  replicas: 1 # number of pod we want 
  template: # pod info 
    metadata:
      labels:
        run: ashupodnew
    spec:
      containers:
      - image: dockerashu/ashu-customer1:releasev1
        name: ashupodnew
        ports:
        - containerPort: 80

```

### deploy manfiest

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl   create  -f   ashu-rc.yaml 
replicationcontroller/ashu-rc1 created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl  get  replicationcontroller
NAME       DESIRED   CURRENT   READY   AGE
ashu-rc1   1         1         1       16s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl  get   rc                  
NAME       DESIRED   CURRENT   READY   AGE
ashu-rc1   1         1         1       21s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl   get  pods
NAME             READY   STATUS    RESTARTS   AGE
ashu-rc1-cdl69   1/1     Running   0          40s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl   get  pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP           NODE                                NOMINATED NODE   READINESS GATES
ashu-rc1-cdl69   1/1     Running   0          47s   10.244.1.2   aks-agentpool-18505526-vmss000004   <none>           <none>
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> 
```

