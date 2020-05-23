

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
 //需要和Pod的标准一致
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


   
```