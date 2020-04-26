安装flannel

创建目录

```
[root@master]# mkdir -p /root/install-k8s/plugin/flannel
[root@master]# mkdir plugin
[root@master]# mkdir -p /root/install-k8s/core/
[root@master]# mv kubeadm-config.yaml kubeadm-config.log core/
```

获取kube-flannel.yml
`wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

kubectl apply/create
```
[root@master flannel]# kubectl apply -f kube-flannel.yml
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created
```

```
[root@master flannel]# sudo docker pull registry.cn-hangzhou.aliyuncs.com/woodnie/kube-flannel-ds:v0.12.0-amd64
v0.12.0-amd64: Pulling from woodnie/kube-flannel-ds
Digest: sha256:4198ba6f82f642dfd18ecf840ee37afb9df4b596f06eef20e44d0aec4ea27216
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/woodnie/kube-flannel-ds:v0.12.0-amd64
registry.cn-hangzhou.aliyuncs.com/woodnie/kube-flannel-ds:v0.12.0-amd64
```

```
[root@master flannel]# sudo docker tag registry.cn-hangzhou.aliyuncs.com/woodnie/kube-flannel-ds:v0.12.0-amd64 quay.io/coreos/flannel:v0.12.0-amd64
```

kubectl get pod -n kube-system
-n namespace name kube-system by default namespace is default

```
[root@master flannel]# kubectl get pod -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
coredns-66bff467f8-77d6q                  1/1     Running   0          110m
coredns-66bff467f8-bjmsp                  1/1     Running   0          110m
etcd-master.wood.com                      1/1     Running   0          110m
kube-apiserver-master.wood.com            1/1     Running   0          110m
kube-controller-manager-master.wood.com   1/1     Running   9          110m
kube-flannel-ds-amd64-82k48               1/1     Running   0          67m
kube-proxy-tlhmc                          1/1     Running   0          110m
kube-scheduler-master.wood.com            1/1     Running   11         110m
```


```
[root@master flannel]# kubectl get node
NAME              STATUS   ROLES    AGE    VERSION
master.wood.com   Ready    master   111m   v1.18.0
```
