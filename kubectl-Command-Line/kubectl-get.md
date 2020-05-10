##kubectl get





```
[root@master ~]# kubectl get namespace
NAME              STATUS   AGE
default           Active   24h
kube-node-lease   Active   24h
kube-public       Active   24h
kube-system       Active   24h
```

####默认查看 defaut namespace 视图下的pod
```
[root@master ~]# kubectl get pod
No resources found in default namespace.
```
####kubeadm 创建的pod 在kube-system namespace 视图下
```
[root@master ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-ksz88                  1/1     Running   0          25h
kube-system   coredns-66bff467f8-z7rz2                  1/1     Running   0          25h
kube-system   etcd-master.wood.com                      1/1     Running   0          25h
kube-system   kube-apiserver-master.wood.com            1/1     Running   0          25h
kube-system   kube-controller-manager-master.wood.com   1/1     Running   11         25h
kube-system   kube-flannel-ds-amd64-6wkv6               1/1     Running   0          22h
kube-system   kube-flannel-ds-amd64-n5g87               1/1     Running   0          22h
kube-system   kube-flannel-ds-amd64-zb4qt               1/1     Running   0          25h
kube-system   kube-proxy-f977b                          1/1     Running   0          25h
kube-system   kube-proxy-pkj8z                          1/1     Running   1          22h
kube-system   kube-proxy-td27j                          1/1     Running   1          22h
kube-system   kube-scheduler-master.wood.com            1/1     Running   10 


```