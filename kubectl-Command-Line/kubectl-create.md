##kubectl create


##kubectl create example
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
```
##kubectl scale 
```
[root@master ~]# kubectl scale --replicas=2 deployment/wood-nginx-dep 
deployment.apps/wood-nginx-dep scaled

[root@master ~]# kubectl get pod -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
wood-nginx-dep-8559775479-7s7vr   1/1     Running   0          20s   10.244.2.7   node01.wood.com   <none>           <none>
wood-nginx-dep-8559775479-kb26t   1/1     Running   1          22h   10.244.1.8   node02.wood.com   <none>           <none>

[root@master ~]# curl  10.244.1.8/info.html
wood-nginx-dep-8559775479-kb26t
[root@master ~]# curl  10.244.2.8/info.html
wood-nginx-dep-8559775479-7s7vr

```

##kubectl expose
```
[root@master ~]# kubectl expose deployment wood-nginx-dep --port=80 --target-port=80
service/wood-nginx-dep exposed

[root@master ~]# kubectl get svc
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP   8d
wood-nginx-dep   ClusterIP   10.107.69.38   <none>        80/TCP    46s

[root@master ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 rr
  -> 192.168.1.100:6443           Masq    1      3          0
TCP  10.96.0.10:53 rr
  -> 10.244.0.2:53                Masq    1      0          0
  -> 10.244.0.3:53                Masq    1      0          0
TCP  10.96.0.10:9153 rr
  -> 10.244.0.2:9153              Masq    1      0          0
  -> 10.244.0.3:9153              Masq    1      0          0
TCP  10.107.69.38:80 rr
  -> 10.244.1.8:80                Masq    1      0          0
  -> 10.244.2.8:80                Masq    1      0          0
UDP  10.96.0.10:53 rr
  -> 10.244.0.2:53                Masq    1      0          0
  -> 10.244.0.3:53                Masq    1      0          0

 >>rr    
[root@master ~]# curl 10.107.69.38/info.html
wood-nginx-dep-8559775479-kb26t
[root@master ~]# curl 10.107.69.38/info.html
wood-nginx-dep-8559775479-7s7vr
```

##kubectl edit svc 
修改~~type: ClusterIP~~ --> type: NodePort 支持外部访问
```
[root@master ~]# kubectl edit svc wood-nginx-dep
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:app: {}
      f:spec:
        f:ports:
          .: {}
          k:{"port":80,"protocol":"TCP"}:
            .: {}
            f:port: {}
            f:protocol: {}
            f:targetPort: {}
        f:selector:
          .: {}
          f:app: {}
        f:sessionAffinity: {}
        f:type: {}
    manager: kubectl
    operation: Update
    time: "2020-05-03T15:06:12Z"
  name: wood-nginx-dep
  namespace: default
  resourceVersion: "957565"
  selfLink: /api/v1/namespaces/default/services/wood-nginx-dep
  uid: c52e5318-1ccc-4d26-a23f-9ffc0a953b7d
spec:
  clusterIP: 10.107.69.38
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: wood-nginx-dep
  sessionAffinity: None
  ~~type: ClusterIP~~ 
  type: NodePort

status:
  loadBalancer: {}
```
```
[root@master ~]# kubectl get svc
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP        8d
wood-nginx-dep   NodePort    10.107.69.38   <none>        80:31428/TCP   31m


[root@node01 ~]#  curl 192.168.1.100:31428/info.html
wood-nginx-dep-8559775479-tl98d
[root@node02 ~]#  curl 192.168.1.100:31428/info.html
wood-nginx-dep-8559775479-kb26t
```

