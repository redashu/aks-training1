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
