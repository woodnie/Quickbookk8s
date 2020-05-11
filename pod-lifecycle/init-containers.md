example:myapp.yaml
This example defines a simple Pod that has two init containers. The first waits for **myservice**, and the second waits for **mydb**. Once both init containers complete, the Pod runs the app container from its spec section.
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```
因为myservice和mydb service 没有ready ,所有本例chu

```
[root@master ~]# kubectl apply -f myapp.yaml
pod/myapp-pod created
```

```
[root@master ~]# kubectl get -f myapp.yaml
NAME        READY   STATUS     RESTARTS   AGE
myapp-pod   0/1     Init:0/2   0          2m56s
```