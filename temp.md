集群资源分类：

* 名称空间级别：kubectl get -n default, kubectl get -n kube-system
* 集群级别：role,整个集群可见
* 元数据类型：HPA 

example:

> `[root@master ~]# kubectl run nginx-wood --image=nginx`
>
> `[root@master ~]# kubectl get pod`
>
> `NAME         READY   STATUS    RESTARTS   AGE`
>
> `nginx-wood   1/1     Running   0          18m`
>
> ```bash
> [root@master ~]# kubectl get pod -o wide
> ```
>
> `NAME         READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES`
>
> `nginx-wood   1/1     Running   0          19m   10.244.1.3   node02.wood.com   <none>           <none>`
>
> \[root@node02 ~\]\# docker ps -a
>
> CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                   PORTS               NAMES
>
> d9dc4a824767        nginx                    "nginx -g 'daemon of…"   27 minutes ago      Up 27 minutes                                k8s\_nginx-wood\_nginx-wood\_default\_6785eaf0-c084-4a2b-bb24-48b989608146\_0
>
> 6626937d3193        k8s.gcr.io/pause:3.2     "/pause"                 27 minutes ago      Up 27 minutes                                k8s\_POD\_nginx-wood\_default\_6785eaf0-c084-4a2b-bb24-48b989608146\_0
>
> 8fc9a1113982        nginx                    "nginx -g 'daemon of…"   5 hours ago         Exited \(0\) 5 hours ago                       festive\_liskov



