### pod manifest day4

```
apiVersion: v1
kind: Pod
metadata:
  name: ashupod111
spec: 
  containers:
  - name: ashuc1
    image: dockerashu/ashu-customer1:releasev1 # image from dockerhub
    ports: # optional field 
    - containerPort: 80 
```

### deploy pod manifest

```
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl  get  pods                                                                            No resources found in default namespace.                                                                                                          PS C:\Users\humanfirmware\Desktop\my-yaml-manifest>                                                                                               PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl  create  -f   ashupod1.yaml                                                           
pod/ashupod111 created
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl get  pods 
NAME         READY   STATUS              RESTARTS   AGE
ashupod111   0/1     ContainerCreating   0          6s
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl get  pods -o wide
NAME         READY   STATUS              RESTARTS   AGE   IP       NODE                                NOMINATED NODE   READINESS GATES
ashupod111   0/1     ContainerCreating   0          10s   <none>   aks-agentpool-18505526-vmss000003   <none>           <none>
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> kubectl get  nodes
NAME                                STATUS   ROLES   AGE   VERSION
aks-agentpool-18505526-vmss000002   Ready    agent   31m   v1.26.6
aks-agentpool-18505526-vmss000003   Ready    agent   31m   v1.26.6
PS C:\Users\humanfirmware\Desktop\my-yaml-manifest> 

```
