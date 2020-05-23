Kubernetes ServiceTypes 允许指定一个需要的类型的 Service，Type 的取值以及行为如下：

ClusterIP：通过集群的内部 IP 暴露服务，选择该值，服务只能够在集群内部可以访问，这也是默认的 ServiceType。
NodePort：通过每个 Node 上的 IP 和静态端口（NodePort）暴露服务。NodePort 服务会路由到 ClusterIP 服务，这个 ClusterIP 服务会自动创建。通过请求 <NodeIP>:<NodePort>，可以从集群的外部访问一个 NodePort 服务。
LoadBalancer：使用云提供商的负载局衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 NodePort 服务和 ClusterIP 服务。
ExternalName：通过返回 CNAME 和它的值，可以将服务映射到 externalName 字段的内容（例如， foo.bar.example.com）。 没有任何类型代理被创建。

```
[root@master ~]# cat myapp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: myapp-deployment
spec:
 replicas: 2
 selector:
  matchLabels:
   app: myapp
   release: stabel
 template:
  metadata:
   labels:
    app: myapp
    release: stabel
    env: test
  spec:
   containers:
   - name: myapp
     image: myapp:v1
     ports:
     - containerPort: 80

[root@master ~]# kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
myapp-deployment-775488867c-6dkvl   1/1     Running   0          13m   10.244.2.24   node02.wood.com   <none>           <none>
myapp-deployment-775488867c-t56vb   1/1     Running   0          13m   10.244.1.20   node01.wood.com   <none>           <none>

```

###Cluster IP
```
[root@master ~]# cat myapp-service.yaml
apiVersion: v1
kind: Service
metadata:
 name: myapp-service
spec:
 type: ClusterIP
 selector:
  app: myapp
  release: stabel
 ports:
 - name: http
   port: 80
   targetPort: 80
   
[root@master ~]# kubectl get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP   14d
myapp-service   ClusterIP   10.105.69.158   <none>        80/TCP    3s

[root@master ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 rr
  -> 192.168.1.100:6443           Masq    1      1          0
TCP  10.96.0.10:53 rr
  -> 10.244.1.10:53               Masq    1      0          0
  -> 10.244.2.16:53               Masq    1      0          0
// IPVS 转发规则  
TCP  10.96.0.10:9153 rr
  -> 10.244.1.10:9153             Masq    1      0          0
  -> 10.244.2.16:9153             Masq    1      0          0

TCP  10.105.69.158:80 rr
  -> 10.244.1.20:80               Masq    1      0          0
  -> 10.244.2.24:80               Masq    1      0          0
UDP  10.96.0.10:53 rr
  -> 10.244.1.10:53               Masq    1      0          0
  -> 10.244.2.16:53               Masq  

[root@master ~]# curl 10.105.69.158/appversion.html
appversion=v1
[root@master ~]# curl 10.105.69.158/appversion.html
appversion=v1
   
```
###NodePort
