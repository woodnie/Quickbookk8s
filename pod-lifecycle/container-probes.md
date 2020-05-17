
 
###容器探针
探针 是由 kubelet 对容器执行的定期诊断。要执行诊断，kubelet 调用由容器实现的 Handler。有三种类型的处理程序：

* ExecAction：在容器内执行指定命令。如果命令退出时返回码为 0 则认为诊断成功。
* TCPSocketAction：对指定端口上的容器的 IP 地址进行 TCP 检查。如果端口打开，则诊断被认为是成功的。
* HTTPGetAction：对指定的端口和路径上的容器的 IP 地址执行 HTTP Get 请求。如果响应的状态码大于等于200 且小于 400，则诊断被认为是成功的。

每次探测都将获得以下三种结果之一：

* 成功：容器通过了诊断。
* 失败：容器未通过诊断。
* 未知：诊断失败，因此不会采取任何行动。

Kubelet 可以选择是否执行在容器上运行的三种探针执行和做出反应：

* livenessProbe：指示容器是否正在运行。如果存活探测失败，则 kubelet 会杀死容器，并且容器将受到其 重启策略 的影响。如果容器不提供存活探针，则默认状态为 Success。
* readinessProbe：指示容器是否准备好服务请求。如果就绪探测失败，端点控制器将从与 Pod 匹配的所有 Service 的端点中删除该 Pod 的 IP 地址。初始延迟之前的就绪状态默认为 Failure。如果容器不提供就绪探针，则默认状态为 Success。
* startupProbe: 指示容器中的应用是否已经启动。如果提供了启动探测(startup probe)，则禁用所有其他探测，直到它成功为止。如果启动探测失败，kubelet 将杀死容器，容器服从其重启策略进行重启。如果容器没有提供启动探测，则默认状态为成功Success。

**该什么时候使用存活（liveness）和就绪（readiness）探针?**
如果容器中的进程能够在遇到问题或不健康的情况下自行崩溃，则不一定需要存活探针; kubelet 将根据 Pod 的restartPolicy 自动执行正确的操作。

如果您希望容器在探测失败时被杀死并重新启动，那么请指定一个存活探针，并指定restartPolicy 为 Always 或 OnFailure。

如果要仅在探测成功时才开始向 Pod 发送流量，请指定就绪探针。在这种情况下，就绪探针可能与存活探针相同，但是 spec 中的就绪探针的存在意味着 Pod 将在没有接收到任何流量的情况下启动，并且只有在探针探测成功后才开始接收流量。

如果您希望容器能够自行维护，您可以指定一个就绪探针，该探针检查与存活探针不同的端点。

请注意，如果您只想在 Pod 被删除时能够排除请求，则不一定需要使用就绪探针；在删除 Pod 时，Pod 会自动将自身置于未完成状态，无论就绪探针是否存在。当等待 Pod 中的容器停止时，Pod 仍处于未完成状态。

###重启策略

PodSpec 中有一个 restartPolicy 字段，可能的值为 Always、OnFailure 和 Never。默认为 Always。 restartPolicy 适用于 Pod 中的所有容器。restartPolicy 仅指通过同一节点上的 kubelet 重新启动容器。失败的容器由 kubelet 以五分钟为上限的指数退避延迟（10秒，20秒，40秒…）重新启动，并在成功执行十分钟后重置。如 Pod 文档 中所述，一旦绑定到一个节点，Pod 将永远不会重新绑定到另一个节点。

###Pod 的生命
一般来说，Pod 不会消失，直到人为销毁他们。这可能是一个人或控制器。这个规则的唯一例外是成功或失败的 phase 超过一段时间（由 master 确定）的Pod将过期并被自动销毁。

有三种可用的控制器：

使用 Job 运行预期会终止的 Pod，例如批量计算。Job 仅适用于重启策略为 OnFailure 或 Never 的 Pod。

对预期不会终止的 Pod 使用 ReplicationController、ReplicaSet 和 Deployment ，例如 Web 服务器。 ReplicationController 仅适用于具有 restartPolicy 为 Always 的 Pod。

提供特定于机器的系统服务，使用 DaemonSet 为每台机器运行一个 Pod 。

所有这三种类型的控制器都包含一个 PodTemplate。建议创建适当的控制器，让它们来创建 Pod，而不是直接自己创建 Pod。这是因为单独的 Pod 在机器故障的情况下没有办法自动复原，而控制器却可以。

如果节点死亡或与集群的其余部分断开连接，则 Kubernetes 将应用一个策略将丢失节点上的所有 Pod 的 phase 设置为 Failed。

###示例
####readinessProbe-httpgget

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-httpget-pod
spec:
  containers:
  - name: readiness-httpget-c
    image: nginx:v1
    readinessProbe:
      httpGet:
        port: 80
        path: /healthz.html
      initialDelaySeconds: 1
      periodSeconds: 3
```
**pod running but not ready:**
```
[root@master ~]# kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
myapp-pod               1/1     Running   6          4d18h
readiness-httpget-pod   0/1     Running   0          47m
```
**add file healthz.html**
```
[root@master ~]# kubectl exec -f readinessProbe-httpgget.yaml -it -- /bin/bash
root@readiness-httpget-pod:/# cd /usr/share/nginx/
root@readiness-httpget-pod:/usr/share/nginx/html#
root@readiness-httpget-pod:/usr/share/nginx/html# echo healthz > healthz.html
```
**pod running and ready:**
```
[root@master ~]# kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
myapp-pod               1/1     Running   6          4d18h
readiness-httpget-pod   1/1     Running   0          47m
```
####livenessProbe-exec
```
[root@master ~]# vi livenessProbe-exec.yaml
apiVersion: v1
kind: Pod
metadata:
 name: liveness-exec-pod
spec:
 containers:
 - name: liveness-exec-c
   image: busybox:v1
   command: ["/bin/sh","-c","touch /tmp/live;sleep 60;rm -rf /tmp/live; sleep 3600"]
   livenessProbe:
    exec:
     command: ["test","-e", "/tmp/live"]
    initialDelaySeconds: 1
    periodSeconds: 3
```
**查看pod状态:**
```
[root@master ~]# kubectl get pod -w
NAME                    READY   STATUS    RESTARTS   AGE
liveness-exec-pod       1/1     Running   0          105s
liveness-exec-pod       1/1     Running   1          108s
liveness-exec-pod       1/1     Running   2          3m27s
liveness-exec-pod       1/1     Running   3          5m5s
liveness-exec-pod       1/1     Running   4          6m45s
liveness-exec-pod       1/1     Running   5          8m26s
```
####livenessProbe-httpget
```
[root@master ~]# vim livenessProbe-httpget.yaml
apiVersion: v1
kind: Pod
metadata:
 labels:
  test: liveness
 name: liveness-httpget-pod
spec:
 containers:
  - name: liveness-httpget-c
    image: nginx:v1
    ports:
    - name: http
      containerPort: 80
    livenessProbe:
     httpGet:
      port: 80
      path: /index.html
     initialDelaySeconds: 1
     periodSeconds: 3
```
**查看pod状态:**
```
[root@master ~]# kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
liveness-httpget-pod   1/1     Running   0          4m43s   10.244.1.3   node01.wood.com   <none>           <none>
```
**index.html页面存在**
```
[root@master ~]# curl 10.244.1.3
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[root@master ~]#


**删除index.html页面**
```
[root@master ~]#  kubectl exec liveness-httpget-pod  -- /bin/rm /usr/share/nginx/html/index.html
```
**index.html页面不存在**
```
[root@master ~]# curl  10.244.1.3
```
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.17.10</center>
</body>
</html>

**pod重启**
```
[root@master ~]# kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
liveness-httpget-pod   1/1     Running   1          13m   10.244.1.3   node01.wood.com   <none>           <none>
```
####livenessProbe-tcp
```
[root@master ~]# vi livenessProbe-tcpsocket.yaml
apiVersion: v1
kind: Pod
metadata:
 labels:
  test: liveness
 name: liveness-tcpsocket-pod
spec:
 containers:
  - name: liveness-tcpsocket-c
    image: nginx:v1
    livenessProbe:
     tcpSocket:
      port: 8080
     initialDelaySeconds: 1
     periodSeconds: 3
```
**默认nginx监听TCP 80， 该pod中对于 port 8080 会一直处于失败状态**
```
[root@master ~]# kubectl get pod -w
NAME                     READY   STATUS    RESTARTS   AGE
liveness-tcpsocket-pod   1/1     Running   5          118s
liveness-tcpsocket-pod   0/1     CrashLoopBackOff   5          2m2s
liveness-tcpsocket-pod   1/1     Running            6          3m23s
liveness-tcpsocket-pod   0/1     CrashLoopBackOff   6          3m34s
```
