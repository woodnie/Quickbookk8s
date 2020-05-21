

```
[root@master ~]# vi replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: frontend-rs
 labels:
  app: guestbook
  tier: frontend-rs
spec:
 replicas: 2
 selector:
  matchLabels:
   tier: frontend-rs
 template:
  metadata:
   labels:
    tier: frontend-rs
  spec:
   containers:
   - name: nginx-rs
     image: nginx:v1
 ```
 
 ```
 [root@master ~]# kubectl get rs
NAME          DESIRED   CURRENT   READY   AGE
frontend-rs   2         2         2       21h
[root@master ~]#

 ```
 