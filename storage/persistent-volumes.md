
```
[root@master pvc]# cat pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 name: my-nfs-pv
spec:
 capacity:
  storage: 10Gi
 accessModes:
  - ReadWriteOnce
 persistentVolumeReclaimPolicy: Recycle
 storageClassName: nfs
 nfs:
  path: /nfsdata
  server: 192.168.1.100

```
