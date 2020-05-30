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
####hostPath
hostPath 卷能将主机节点文件系统上的文件或目录挂载到您的 Pod 中。 虽然这不是大多数 Pod 需要的，但是它为一些应用程序提供了强大的逃生舱。

例如，hostPath 的一些用法有：

运行一个需要访问 Docker 引擎内部机制的容器；请使用 hostPath 挂载 /var/lib/docker 路径。
在容器中运行 cAdvisor 时，以 hostPath 方式挂载 /sys。
允许 Pod 指定给定的 hostPath 在运行 Pod 之前是否应该存在，是否应该创建以及应该以什么方式存在。
除了必需的 path 属性之外，用户可以选择性地为 hostPath 卷指定 type。

支持的 type 值如下：

|取值	|行为|
||空字符串（默认）用于向后兼容，这意味着在安装 hostPath 卷之前不会执行任何检查。|
DirectoryOrCreate	如果在给定路径上什么都不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 Kubelet 相同的组和所有权。
Directory	在给定路径上必须存在的目录。
FileOrCreate	如果在给定路径上什么都不存在，那么将在那里根据需要创建空文件，权限设置为 0644，具有与 Kubelet 相同的组和所有权。
File	在给定路径上必须存在的文件。
Socket	在给定路径上必须存在的 UNIX 套接字。
CharDevice	在给定路径上必须存在的字符设备。
BlockDevice	在给定路径上必须存在的块设备。
当使用这种类型的卷时要小心，因为：

具有相同配置（例如从 podTemplate 创建）的多个 Pod 会由于节点上文件的不同而在不同节点上有不同的行为。
当 Kubernetes 按照计划添加资源感知的调度时，这类调度机制将无法考虑由 hostPath 使用的资源。
基础主机上创建的文件或目录只能由 root 用户写入。您需要在 特权容器 中以 root 身份运行进程，或者修改主机上的文件权限以便容器能够写入 hostPath 卷。

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