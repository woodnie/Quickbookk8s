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

**start this Pod by running:**
```
[root@master ~]# kubectl apply -f myapp.yaml
pod/myapp-pod created
```
**check on its status with:**
```
[root@master ~]# kubectl get -f myapp.yaml
NAME        READY   STATUS     RESTARTS   AGE
myapp-pod   0/1     Init:0/2   0          2m56s
```
**more details:**
```
[root@master ~]# kubectl describe -f myapp.yaml
Name:         myapp-pod
Namespace:    default
Priority:     0
Node:         node01.wood.com/192.168.1.101
Start Time:   Mon, 11 May 2020 23:01:29 +0800
Labels:       app=myapp
Annotations:  Status:  Pending
IP:           10.244.1.2
IPs:
  IP:  10.244.1.2
Init Containers:
  init-myservice:
    Container ID:  docker://41e6c87820313be890ceee281ba49aec5119757f33940174f004527a28e153e0
    Image:         busybox:1.28
    Image ID:      docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done
    State:          Running
      Started:      Mon, 11 May 2020 23:01:54 +0800
    Ready:          False  ##waiting to discover Services named mydb and myservice.
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2lpkw (ro)
  init-mydb:
    Container ID:
    Image:         busybox:1.28
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False ##waiting to discover Services named mydb and myservice.
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2lpkw (ro)
Containers:
  myapp-container:
    Container ID:
    Image:         busybox:v1
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2lpkw (ro)
Conditions:
  Type              Status
  Initialized       False
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-2lpkw:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-2lpkw
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                      Message
  ----    ------     ----       ----                      -------
  Normal  Scheduled  <unknown>  default-scheduler         Successfully assigned default/myapp-pod to node01.wood.com
  Normal  Pulling    6m51s      kubelet, node01.wood.com  Pulling image "busybox:1.28"
  Normal  Pulled     6m32s      kubelet, node01.wood.com  Successfully pulled image "busybox:1.28"
  Normal  Created    6m32s      kubelet, node01.wood.com  Created container init-myservice
  Normal  Started    6m31s      kubelet, node01.wood.com  Started container init-myservice
[root@master ~]#
```

**Inspect the first init container**
```
[root@master ~]# kubectl logs myapp-pod -c init-myservice
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'myservice.default.svc.cluster.local'
waiting for myservice
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
......
```
**Inspect the second init container**
```
[root@master ~]# kubectl logs myapp-pod -c init-mydb
Error from server (BadRequest): container "init-mydb" in pod "myapp-pod" is waiting to start: PodInitializing
```
**因为myservice和mydb service 没有ready. 本例中myapp-pod一直处于Init状态**

