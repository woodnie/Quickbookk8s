###通过--from-file指定目录创建

```
[root@master ~]# ll install-k8s/configmap/dir/
total 8
-rw-r--r--. 1 root root 158 May 24 13:24 game.properties
-rw-r--r--. 1 root root  83 May 24 13:25 ui.properties


[root@master dir]# cat ui.properties
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice
[root@master dir]# cat game.properties
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
[root@master dir]#

//--from-file 指定目录下的所有文件都会在CinfigMap里创建一个键值对，键的名字的文件名，键的值就是文件内容：
[root@master dir]# kubectl create configmap game-config --from-file=./
configmap/game-config created
[root@master dir]# kubectl get configmap
NAME          DATA   AGE
game-config   2      17s

[root@master dir]# kubectl describe configmap/game-config
Name:         game-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30

ui.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

Events:  <none>
```

###通过--from-file指定文件创建
```
[root@master dir]# kubectl create configmap game-config-f1 --from-file=game.properties
configmap/game-config-f1 created
[root@master dir]# kubectl create configmap game-config-f2 --from-file=ui.properties
configmap/game-config-f2 created

[root@master dir]# kubectl get configmap
NAME             DATA   AGE
game-config-f1   1      23s
game-config-f2   1      14s
[root@master dir]#
```

###通过--from-literal指定
```
[root@master dir]# kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
configmap/special-config created
[root@master dir]# kubectl get congfigmap
error: the server doesn't have a resource type "congfigmap"
[root@master dir]# kubectl get configmap
NAME             DATA   AGE
game-config      2      113m
game-config-f1   1      10m
game-config-f2   1      10m
special-config   2      28s
```

###yaml 文件指定
```
[root@master configmap]# cat env-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
 name: env-config
data:
 log_level: INFO

[root@master configmap]# kubectl apply -f env-config.yaml
configmap/env-config created
```
###Pod 中使用ConfigMap ( special-config and env )
####1.使用ConfigMap 来替代环境变量
```
[root@master configmap]# kubectl apply -f configmap-pod.yaml
pod/configmap-pod created
[root@master configmap]# cat configmap-pod1.yaml
apiVersion: v1
kind: Pod
metadata:
 name: configmap-pod1
spec:
 containers:
  - name: myapp-configmap-c
    image: myapp:v1
    command: ["/bin/sh","-c","env"]
    env:
     - name: SPECIAL_LEVEL_KEY
       valueFrom:
        configMapKeyRef:
         name: special-config
         key: special.how
     - name: SPECIAL_TYPE_KEY
       valueFrom:
        configMapKeyRef:
         name: sepcail-config
         key: specail.type
    envFrom:
     - configMapRef:
        name: env-config
 restartPolicy: Never
[root@master configmap]#

[root@master configmap]# kubectl logs configmap-pod
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=configmap-pod
HOME=/root
PKG_RELEASE=1~buster
SPECIAL_TYPE_KEY=charm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NGINX_VERSION=1.17.10
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
NJS_VERSION=0.3.9
KUBERNETES_PORT_443_TCP_PROTO=tcp
SPECIAL_LEVEL_KEY=very
log_level=INFO
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/

```

####2.通过ConfigMap 设置命令行参数
```
[root@master configmap]# cat configmap-pod2.yaml
apiVersion: v1
kind: Pod
metadata:
 name: configmap-pod2
spec:
 containers:
  - name: myapp-configmap-c
    image: myapp:v1
    command: ["/bin/sh","-c","echo $(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)"]
    env:
     - name: SPECIAL_LEVEL_KEY
       valueFrom:
        configMapKeyRef:
         name: special-config
         key: special.how
     - name: SPECIAL_TYPE_KEY
       valueFrom:
        configMapKeyRef:
         name: special-config
         key: special.type
    envFrom:
     - configMapRef:
        name: env-config
 restartPolicy: Never

[root@master configmap]# kubectl get pod
NAME             READY   STATUS      RESTARTS   AGE
configmap-pod2    0/1     Completed   0          42m

[root@master configmap]# kubectl logs configmap-pod2
very charm
```
####3.通过数据卷插件使用ConfigMap
3.1
```
[root@master configmap]# cat configmap-pod3.yaml
apiVersion: v1
kind: Pod
metadata:
 name: configmap-pod3
spec:
 containers:
  - name: myapp-configmap-c
    image: myapp:v1
    command: ["/bin/sh","-c","cat /etc/configmap/special.how /etc/configmap/special.type "]
    volumeMounts:
     - name: config-volume
       mountPath: /etc/configmap
 volumes:
  - name: config-volume
    configMap:
     name: special-config
 restartPolicy: Never

[root@master configmap]# kubectl get pod
NAME             READY   STATUS      RESTARTS   AGE
configmap-pod3   0/1     Completed   0          116s

[root@master configmap]# kubectl logs configmap-pod3
verycharm
```
3.2
```
[root@master configmap]# kubectl exec -it configmap-pod4 -- /bin/sh
# ls /etc/configmap
special.how  special.type
# cat /etc/configmap/special.how
very#
# cat /etc/configmap/special.type
charm#
#
```

####ConfigMap 热跟新


```
[root@master configmap]# cat configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
 name: log-config
data:
 log_level: INFO
---
apiVersion: apps/v1
kind: Deployment
metadata:
 name: my-nginx
spec:
 replicas: 1
 selector:
  matchLabels:
   run: my-nginx
 template:
  metadata:
   labels:
    run: my-nginx
  spec:
   containers:
   - name: my-nginx
     image: nginx:v1
     ports:
     - containerPort: 80
     volumeMounts:
     - name: config-volume
       mountPath: /etc/configmap
   volumes:
    - name: config-volume
      configMap:
       name: log-config

[root@master configmap]# kubectl get pod
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-85dbd4bf4b-rsv5c   1/1     Running   0          20s

[root@master configmap]# kubectl get pod -l run=my-nginx
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-85dbd4bf4b-rsv5c   1/1     Running   0          15m


[root@master configmap]# kubectl exec -it my-nginx-85dbd4bf4b-rsv5c -- /bin/sh
# ls -l /etc/configmap
total 0
lrwxrwxrwx. 1 root root 16 May 24 16:45 log_level -> ..data/log_level
# cat /etc/configmap/log_level
INFO# exit

//log_level: INFO --> log_level: DEBUG
//ConfigMap
[root@master configmap]# kubectl edit configmap log-config
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  log_level: DEBUG
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"log_level":"DEBUG"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"log-config","namespace":"default"}}
  creationTimestamp: "2020-05-24T16:44:54Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:log_level: {}
      f:metadata:
        f:annotations:
          .: {}
          f:kubectl.kubernetes.io/last-applied-configuration: {}
    manager: kubectl
    operation: Update
    time: "2020-05-24T16:44:54Z"
  name: log-config
  namespace: default
  resourceVersion: "880866"
  selfLink: /api/v1/namespaces/default/configmaps/log-config
  uid: a3918f1d-9778-458c-9f76-ad3990735ba9

DEBUG[root@master configmap]# kubectl exec -it `kubectl get pod -l run=my-nginx -o=name | cut -d "/" -f2` -- cat /etc/configmap/log_level
DEBUG

ConfigMap 跟新后不会触发相关Pod的滚动更新，可以通过修改pod annotations 的方式强制触发滚动更新
[root@master configmap]# kubectl patch deployment my-nginx --patch '{"spec": {"template": {"{metadata": {"annotations": {"version/config":"20200525"}}}}}'
deployment.apps/my-nginx patched (no change)

使用该ConfigMap 挂载的Env 不会同步跟新
使用该ConfigMap 挂载的volume 中的数据需要一段时间 （大概10秒）才能同步更新

```