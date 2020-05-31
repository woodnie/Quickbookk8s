
```
[root@master pvc]# cat /etc/exports
/nfsdata1 *(rw,no_root_squash,no_all_squash,sync)
/nfsdata2 *(rw,no_root_squash,no_all_squash,sync)
/nfsdata3 *(rw,no_root_squash,no_all_squash,sync)
[root@master pvc]#
```
3个 PersistentVolume
```
[root@master pvc]# cat pvc.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 name: my-nfs-pv1
spec:
 capacity:
  storage: 1Gi
 accessModes:
  - ReadWriteOnce
 persistentVolumeReclaimPolicy: Recycle
 storageClassName: nfs
 nfs:
  path: /nfsdata1
  server: 192.168.1.100
---
apiVersion: v1
kind: PersistentVolume
metadata:
 name: my-nfs-pv2
spec:
 capacity:
  storage: 5Gi
 accessModes:
  - ReadOnlyMany
 persistentVolumeReclaimPolicy: Recycle
 storageClassName: nfs
 nfs:
  path: /nfsdata2
  server: 192.168.1.100
---
apiVersion: v1
kind: PersistentVolume
metadata:
 name: my-nfs-pv3
spec:
 capacity:
  storage: 10Gi
 accessModes:
  - ReadWriteMany
 persistentVolumeReclaimPolicy: Recycle
 storageClassName: slow
 nfs:
  path: /nfsdata3
  server: 192.168.1.100
[root@master pvc]#
```


```
[root@master pvc]# cat pod.yaml
apiVersion: v1
kind: Service
metadata:
 name: nginx
 labels:
  app: nginx
spec:
 ports:
 - port: 80
   name: web
 clusterIP: None
 selector:
  app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
 name: web
spec:
 selector:
  matchLabels:
   app: nginx
 serviceName: "nginx"
 replicas: 2
 template:
  metadata:
   labels:
    app: nginx
  spec:
   containers:
   - name: nginx
     image: myapp:v1
     ports:
     - containerPort: 80
       name: web
     volumeMounts:
     - name: www
       mountPath: /usr/share/nginx/html
 volumeClaimTemplates:
 - metadata:
    name: www
   spec:
    accessModes: [ "ReadWriteOnce" ]
    storageClassName: "nfs"
    resources:
     requests:
      storage: 1Gi

```
[root@master pvc]# kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP            NODE              NOMINATED NODE   READINESS GATES
web-0   1/1     Running   0          4h52m   10.244.2.39   node02.wood.com   <none>           <none>
web-1   0/1     Pending   0          4h51m   <none>        <none>            <none>           <none>
[root@master pvc]#
```

//- ReadOnlyMany
  - ReadWriteOnce

```
[root@master pvc]# kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   REASON   AGE
my-nfs-pv1   1Gi        RWO            Recycle          Bound       default/www-web-0   nfs                     5h13m
my-nfs-pv2   5Gi        ROX            Recycle          Available                       nfs                     5h13m
my-nfs-pv3   10Gi       RWX            Recycle          Available                       slow                    5h13m
[root@master pvc]# kubectl edit pv/my-nfs-pv2

error: persistentvolumes "my-nfs-pv2" is invalid
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
# persistentvolumes "my-nfs-pv2" was not valid:
# * spec.accessModes: Unsupported value: "ReadOnlyOnce": supported values: "ReadOnlyMany", "ReadWriteMany", "ReadWriteOnce"
# * spec.accessModes: Unsupported value: "ReadOnlyOnce": supported values: "ReadOnlyMany", "ReadWriteMany", "ReadWriteOnce"
#
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolume","metadata":{"annotations":{},"name":"my-nfs-pv2"},"spec":{"accessModes":["ReadOnlyMany"],"capacity":{"storage":"5Gi"},"nfs":{"path":"/nfsdata2","server":"192.168.1.100"},"persistentVolumeReclaimPolicy":"Recycle","storageClassName":"nfs"}}
  creationTimestamp: "2020-05-31T08:29:01Z"
  finalizers:
  - kubernetes.io/pv-protection
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:kubectl.kubernetes.io/last-applied-configuration: {}
      f:spec:
        f:accessModes: {}
        f:capacity:
          .: {}
          f:storage: {}
        f:nfs:
          .: {}
          f:path: {}
          f:server: {}
        f:persistentVolumeReclaimPolicy: {}
        f:storageClassName: {}
        f:volumeMode: {}
    manager: kubectl
    operation: Update
    time: "2020-05-31T08:29:01Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:phase: {}
    manager: kube-controller-manager
    operation: Update
    time: "2020-05-31T08:30:04Z"
  name: my-nfs-pv2
  resourceVersion: "1484062"
  selfLink: /api/v1/persistentvolumes/my-nfs-pv2
  uid: f1da2a88-6a88-4f02-a444-74f88b095c60
spec:
  accessModes:
//- ReadOnlyMany
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  nfs:
    path: /nfsdata2
    server: 192.168.1.100
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  volumeMode: Filesystem
status:
  phase: Available


"/tmp/kubectl-edit-hvd2p.yaml" 65L, 2117C

```