
```
[root@master ~]# cat replicaset.yaml
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

```
[root@master ~]# kubectl describe rs/frontend-rs
Name:         frontend-rs
Namespace:    default
Selector:     tier=frontend-rs
Labels:       app=guestbook
              tier=frontend-rs
Annotations:  <none>
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend-rs
  Containers:
   nginx-rs:
    Image:        nginx:v1
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:           <none>
[root@master ~]#

```
```
[root@master ~]# kubectl get pod -o wide
I0521 21:05:49.183294   44442 request.go:621] Throttling request took 1.324335914s, request: GET:https://192.168.1.100:6443/apis/policy/v1beta1?timeout=32s
NAME                READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
frontend-rs-bfzng   1/1     Running   0          20h   10.244.2.7   node02.wood.com   <none>           <none>
frontend-rs-czgkd   1/1     Running   0          20h   10.244.1.4   node01.wood.com   <none>           <none>
```