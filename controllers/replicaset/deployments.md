Deployments
```
[root@master configmap]# cat ../controllers/myapp-deployment.yaml
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


[root@master ~]# kubectl apply -f nginx-deployment.yaml --record
deployment.apps/nginx-deployment created

[root@master ~]# kubectl get rs
NAME          DESIRED   CURRENT   READY   AGE
frontend-rs   2         2         2       23h

[root@master ~]# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/2     0            0           26s

//deployment 会自动生成rs:
[root@master ~]# kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-7c5b6964f7   2         2         0       7s

[root@master ~]# kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7c5b6964f7-cz8z5   1/1     Running   0          8m11s
nginx-deployment-7c5b6964f7-lnbfw   1/1     Running   0          8m10s


[root@master ~]# kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
nginx-deployment-7c5b6964f7-cz8z5   1/1     Running   0          9m31s   10.244.1.5   node01.wood.com   <none>           <none>
nginx-deployment-7c5b6964f7-lnbfw   1/1     Running   0          9m30s   10.244.2.9   node02.wood.com   <none>           <none>
```
```
[root@master ~]# curl 10.244.1.5
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

```
[root@master ~]# curl 10.244.2.9
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


###scale 扩容
```
[root@master ~]# kubectl scale deployment nginx-deployment --replicas=4
deployment.apps/nginx-deployment scaled

[root@master ~]# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/4     2            2           18m
```
###自动扩展 （需要群集支持HPA）
```
# kubectl autoscale deployment nginx-deployment --min=10  --max=15 --cpu-percent=80--reversion
```

###更新：通过镜像更新
```
//通过自建docker file 更新nginx 
[root@node01 ~]# cat mynginx.Dockerfile
from nginx:v1
run echo version=v1.1 > /usr/share/nginx/html/version.html

[root@node01 ~]# docker build -f ./mynginx.Dockerfile -t nginx:v1.1 .
Sending build context to Docker daemon  18.94kB
Step 1/2 : from nginx:v1
 ---> 602e111c06b6
Step 2/2 : run echo version=v1.1 > /usr/share/nginx/html/version.html
 ---> Running in 0a3df2170857
Removing intermediate container 0a3df2170857
 ---> 874b46c02afc
Successfully built 874b46c02afc
Successfully tagged nginx:v1.1
//新的镜像nginx:v1.1
[root@node02 ~]# docker images
REPOSITORY                                                          TAG                 IMAGE ID            CREATED             SIZE
nginx                                                               v1.1                a1b7621ed708        36 seconds ago      127MB
nginx                                                               v1                  602e111c06b6        4 weeks ago         127MB

// 第一次 更新nginx image nginx=nginx:v1.1
[root@master ~]#  kubectl set image deployment/nginx-deployment nginx=nginx:v1.1
deployment.apps/nginx-deployment image updated

//过程中会生成新的deployment 
[root@master ~]# kubectl describe deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 21 May 2020 22:39:36 +0800
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 2
                        kubernetes.io/change-cause: kubectl apply --filename=nginx-deployment.yaml --record=true
Selector:               app=nginx
Replicas:               4 desired | 2 updated | 5 total | 3 available | 2 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:v1.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  nginx-deployment-7c5b6964f7 (3/3 replicas created)
NewReplicaSet:   nginx-deployment-798f64d544 (2/2 replicas created)
Events:          <none>
[root@master ~]# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/4     2            3           24h

//会生成一个新的的rs
[root@master ~]# kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-798f64d544   2         2         2       83m
nginx-deployment-7c5b6964f7   3         3         3       24h
[root@master ~]#

//新旧pod更替中
[root@master ~]# kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
nginx-deployment-798f64d544-2862x   1/1     Running   0          75m   10.244.2.12   node02.wood.com   <none>           <none>
nginx-deployment-798f64d544-792vq   1/1     Running   0          75m   10.244.2.11   node02.wood.com   <none>           <none>
nginx-deployment-7c5b6964f7-cz8z5   1/1     Running   0          24h   10.244.1.5    node01.wood.com   <none>           <none>
nginx-deployment-7c5b6964f7-lnbfw   1/1     Running   0          24h   10.244.2.9    node02.wood.com   <none>           <none>
nginx-deployment-7c5b6964f7-pkzxn   1/1     Running   0          93m   10.244.1.6    node01.wood.com   <none>           <none>


//第二次 更新nginx image nginx=nginx:v2.1
[root@master ~]#  kubectl set image deployment/nginx-deployment nginx=nginx:v2.1
deployment.apps/nginx-deployment image updated

//新pod完成生产
[root@master ~]# kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
nginx-deployment-5f955cd7bf-62x6m   1/1     Running   0          8h    10.244.2.15   node02.wood.com   <none>           <none>
nginx-deployment-5f955cd7bf-kmj7s   1/1     Running   0          8h    10.244.1.9    node01.wood.com   <none>           <none>
nginx-deployment-5f955cd7bf-kvflb   1/1     Running   0          9h    10.244.1.8    node01.wood.com   <none>           <none>
nginx-deployment-5f955cd7bf-s9ht2   1/1     Running   0          9h    10.244.2.14   node02.wood.com   <none>           <none>

//查看 收到更新的 nginx 的页面，确认 image 已经跟新
[root@master ~]# curl 10.244.2.14/version.html
version=v2.1
[root@master ~]# curl 10.244.2.15/version.html
version=v2.1
[root@master ~]# curl 10.244.1.9/version.html
version=v2.1
[root@master ~]# curl 10.244.1.8/version.html
version=v2.1

//交互式编辑
[root@master ~]# kubectl edit deployment/nginx-deployment

```

###回退
```
//查看历史版本, depoyment 创建时需要指定 --recode
[root@master ~]# kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=nginx-deployment.yaml --record=true
2         kubectl apply --filename=nginx-deployment.yaml --record=true
3         kubectl apply --filename=nginx-deployment.yaml --record=true
//.spec.revisionHistoryLimit
//指定保留的revison记录数量，默认保留所有revion记录
//如果将该值设置为0，deplpymnet将不能回退


//回退到上一个版本
[root@master ~]# kubectl rollout undo deployment/nginx-deployment
deployment.apps/nginx-deployment rolled back
//查看回退过程
[root@master ~]# kubectl rollout status deployment/nginx-deployment
Waiting for deployment spec update to be observed...
Waiting for deployment spec update to be observed...
Waiting for deployment "nginx-deployment" rollout to finish: 0 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 0 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 0 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 4 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 4 new replicas have been updated...


//回退到指定版本
[root@master ~]# kubectl rollout undo deployment/nginx-deployment --to-revison

//暂停回退
[root@master ~]# kubectl rollout pause deployment/nginx-deployment --reversion


```