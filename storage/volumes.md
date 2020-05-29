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