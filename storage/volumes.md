####emptyDir
```
[root@master volume]# cat emptydir1.yaml
apiVersion: v1
kind: Pod
metadata:
 name: emptydir-pod1
spec:
 containers:
 - image: myapp:v1
   name: myapp-c
   volumeMounts:
   - mountPath: /cache
     name: cache-volume
 volumes:
 - name: cache-volume
   emptyDir: {}
[root@master volume]#

```
####emptyDir 两个容器共享emptyDir
```
[root@master volume]# cat emptydir2.yaml
apiVersion: v1
kind: Pod
metadata:
 name: emptydir-pod2
spec:
 containers:
 - image: myapp:v1
   name: myapp-c
   volumeMounts:
   - mountPath: /cache
     name: cache-volume
 - image: busybox:v1
   name: busybox-c
   command: ["/bin/sh","-c","sleep 36000s"]
   volumeMounts:
   - mountPath: /cache
     name: cache-volume
 volumes:
 - name: cache-volume
   emptyDir: {}

[root@master volume]# kubectl exec emptydir-pod2 -c myapp-c -it -- ls -l /cache
total 0

[root@master volume]# kubectl exec emptydir-pod2 -c myapp-c -it -- touch /cache/index.html

[root@master volume]# kubectl exec emptydir-pod2 -c myapp-c -it -- ls -l /cache
total 0
-rw-r--r--. 1 root root 0 May 29 14:54 index.html

[root@master volume]# kubectl exec emptydir-pod2 -c busybox-c -it -- ls -l /cache
total 0
-rw-r--r--    1 root     root             0 May 29 14:54 index.html

```

```
[root@master volume]# cat hostpath1.yaml
apiVersion: v1
kind: Pod
metadata:
 name: hostpath-pod
spec:
 containers:
 - image: nginx:v1
   name: nginx-c
   volumeMounts:
    - mountPath: /hostpath
      name: hostpath
 volumes:
 - name: hostpath
   hostPath:
    path: /data
    type: Directory

[root@master volume]# kubectl apply -f hostpath1.yaml
pod/hostpath-pod created

**node02 上没有/data, 容器在等待/data**
[root@master volume]# kubectl get pod -o wide
NAME           READY   STATUS              RESTARTS   AGE    IP       NODE              NOMINATED NODE   READINESS GATES
hostpath-pod   0/1     ContainerCreating   0          8m3s   <none>   node02.wood.com   <none>           <none>

**node02上创建/data**
[root@node02 ~]# mkdir /data

** 容器开始运行**
[root@master volume]# kubectl get pod -o wide
NAME           READY   STATUS    RESTARTS   AGE     IP            NODE              NOMINATED NODE   READINESS GATES
hostpath-pod   1/1     Running   0          8m54s   10.244.2.37   node02.wood.com   <none>           <none>

** 在pod里 创建一个/hostpath/hostpath.test 文件**
[root@master volume]# kubectl exec hostpath-pod -it -- touch /hostpath/hostpath.test

** node01 上可以看到相应文件
[root@node02 ~]# ll /data/
total 0
-rw-r--r--. 1 root root 0 May 30 11:07 hostpath.test

**写测试**
[root@node02 ~]# echo test > /data/hostpath.test
[root@master volume]# kubectl exec hostpath-pod -it -- cat /hostpath/hostpath.test
test

```