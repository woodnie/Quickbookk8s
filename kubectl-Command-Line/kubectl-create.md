```
[root@master ~]# kubectl create deployment wood-nginx-dep --image=busybox
deployment.apps/wood-nginx-dep created

[root@master ~]# kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
wood-nginx-dep     0/1     0            0           113m

[root@master ~]# kubectl get pod -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
wood-nginx-dep-8559775479-kb26t   1/1     Running   1          22h   10.244.1.8   node02.wood.com   <none>           <none>

[root@master ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS             RESTARTS   AGE
default       wood-nginx-dep-8559775479-kb26t           1/1     Running            0          19m
kube-system   coredns-66bff467f8-77d6q                  1/1     Running            0          7d
kube-system   coredns-66bff467f8-bjmsp                  1/1     Running            0          7d
kube-system   etcd-master.wood.com                      1/1     Running            1          7d
kube-system   kube-apiserver-master.wood.com            1/1     Running            2          7d
kube-system   kube-controller-manager-master.wood.com   1/1     Running            1191       7d
kube-system   kube-flannel-ds-amd64-58crd               1/1     Running            0          6d22h
kube-system   kube-flannel-ds-amd64-82k48               1/1     Running            0          6d23h
kube-system   kube-flannel-ds-amd64-mbx4l               1/1     Running            0          6d22h
kube-system   kube-proxy-tlhmc                          1/1     Running            0          7d
kube-system   kube-proxy-tnthx                          1/1     Running            1          6d22h
kube-system   kube-proxy-x9n7z                          1/1     Running            1          6d22h
kube-system   kube-scheduler-master.wood.com            1/1     Running            923        7d

[root@master ~]# kubectl scale --replicas=2 deployment/wood-nginx-dep
deployment.apps/wood-nginx-dep scaled

[root@master ~]# kubectl get pod -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
wood-nginx-dep-8559775479-7s7vr   1/1     Running   0          20s   10.244.2.7   node01.wood.com   <none>           <none>
wood-nginx-dep-8559775479-kb26t   1/1     Running   1          22h   10.244.1.8   node02.wood.com   <none>           <none>
[root@master ~]# curl -I 10.244.2.7
[root@master ~]# curl -I 10.244.1.8



```



