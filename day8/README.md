## REvision 

### step 1 -- APp containerization using docker or any CR

<img src="rev1.png">

### step 2 setup aks and start sending request to control plane API server

<img src="rev2.png">

### step 3 -- revision understanding apiresource and their manifesting 

<img src="rev3.png">

### final step -- to revsion 

<img src="finalrev.png">

### understanding pv and pvc concept 

<img src="pv.png">

### AKS with storageclass 

<img src="sc.png">

### Implementing DYnamic storage 

### cleaning all the namespace component 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl  delete all,cm,secret,ingress   --all
pod "ashu-webapp-6ccf6f7765-b226l" deleted
pod "ashudb-68656bb49-ggdmj" deleted
service "dblb1" deleted
service "weblb" deleted
deployment.apps "ashu-webapp" deleted
deployment.apps "ashudb" deleted
configmap "kube-root-ca.crt" deleted
ingress.networking.k8s.io "ashu-app-routing-rule" deleted

```

## Making Database deployment work

### creating storage class first 

```
apiVersion:  storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ashu-azure-storageclass # name of sc 
provisioner: disk.csi.azure.com # csi driver to connect azure Disk 
parameters:
  skuName: Premium_ZRS
reclaimPolicy: Retain # if disk got free then other pod can use it 
volumeBindingMode: WaitForFirstConsumer # wait disk to be created when some pod is calling
allowVolumeExpansion: true
```

### creating it 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> cd .\storage-apps\
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl  create -f .\mystorage_class.yaml  
storageclass.storage.k8s.io/ashu-azure-storageclass created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl get  sc
NAME                      PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
ashu-azure-storageclass   disk.csi.azure.com   Retain          WaitForFirstConsumer   true                   5s
azurefile                 file.csi.azure.com   Delete          Immediate              true                   9d
azurefile-csi             file.csi.azure.com   Delete          Immediate              true                   9d
azurefile-csi-premium     file.csi.azure.com   Delete          Immediate              true                   9d
```

### create pvc and point it above Storage class we  created 

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ashu-db-diskpvc
spec:
  accessModes:
  - ReadWriteOnce # only single node can do read and write 
  storageClassName: ashu-azure-storageclass
  resources:
    requests:
      storage: 10Gi # my for database i want 10Gb space 
```

### creating it 

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl create -f .\pvc.yaml 
persistentvolumeclaim/ashu-db-diskpvc created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl get  pvc
NAME              STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS              AGE
ashu-db-diskpvc   Pending                                      ashu-azure-storageclass   4s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 


```

### creating Deployment and using pvc as volume 

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
        persistentVolumeClaim:  # calling pvc which we created 
          claimName: ashu-db-diskpvc # name of already created PVC 
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

### creating it ...

```

PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl  create -f .\mystorage_class.yaml  
storageclass.storage.k8s.io/ashu-azure-storageclass created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl create -f .\db_deploy.yaml 
deployment.apps/ashudb created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl.exe  get  deploy                                            NAME     READY   UP-TO-DATE   AVAILABLE   AGE                                                                                        ashudb   0/1     1            0           4s                                                                                         PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl  get  po                                                    
NAME                      READY   STATUS              RESTARTS   AGE
ashudb-7b6cc6656b-s5jrg   0/2     ContainerCreating   0          11s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl  get  po
NAME                      READY   STATUS              RESTARTS   AGE
ashudb-7b6cc6656b-s5jrg   0/2     ContainerCreating   0          16s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl  get  po
NAME                      READY   STATUS    RESTARTS   AGE
ashudb-7b6cc6656b-s5jrg   2/2     Running   0          27s
=========>>

PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl get  pvc
NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              AGE
ashu-db-diskpvc   Bound    pvc-abef8d06-2446-4f40-a1f8-2767011c3c3b   10Gi       RWO            ashu-azure-storageclass   8m35s      
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps>
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps>
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps>
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> 
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest\storage-apps> kubectl get  pv 
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                          STORAGECLASS              REASON   AGE
pvc-abef8d06-2446-4f40-a1f8-2767011c3c3b   10Gi       RWO            Retain           Bound    ashu-project/ashu-db-diskpvc   ashu-azure-storageclass            59s
```



