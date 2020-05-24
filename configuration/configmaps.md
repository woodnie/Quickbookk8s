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
