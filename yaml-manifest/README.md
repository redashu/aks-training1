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
