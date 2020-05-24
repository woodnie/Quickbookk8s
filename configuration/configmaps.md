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

[root@master dir]# kubectl describe configmap
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