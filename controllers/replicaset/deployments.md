Deployments
```
[root@master ~]# kubectl apply -f nginx-deployment.yaml --record
deployment.apps/nginx-deployment created

[root@master ~]# kubectl get rs
NAME          DESIRED   CURRENT   READY   AGE
frontend-rs   2         2         2       23h

[root@master ~]# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/2     0            0           26s

#deployment 会自动生成rs:
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


scale 扩容
```
[root@master ~]# kubectl scale deployment nginx-deployment --replicas=4
deployment.apps/nginx-deployment scaled

[root@master ~]# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/4     2            2           18m


```