####Built-in Secrets
Service Account 使用 API 凭证自动创建和附加 secret
Kubernetes 自动创建包含访问 API 凭据的 secret，并自动修改您的 pod 以使用此类型的 secret。

如果需要，可以禁用或覆盖自动创建和使用 API 凭据。但是，如果您需要的只是安全地访问 apiserver，我们推荐这样的工作流程。

参阅 Service Account 文档获取关于 Service Account 如何工作的更多信息

####Creating a Secret Using kubectl

```
//假设有些 pod 需要访问数据库。这些 pod 需要使用的用户名和密码存放在如下文件：
[root@master Secret]# echo -n 'admin' > ./username.txt
[root@master Secret]# echo -n '1f2d1e2e67df' > ./password.txt

[root@master Secret]# ll
total 8
-rw-r--r--. 1 root root 12 May 25 22:52 password.txt
-rw-r--r--. 1 root root  5 May 25 22:52 username.txt

//kubectl create secret 命令将这些文件打包到一个 Secret 中并在 API server 中创建了一个对象。
[root@master Secret]# kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
secret/db-user-pass created

[root@master Secret]# kubectl get secret
NAME                  TYPE                                  DATA   AGE
db-user-pass          Opaque                                2      13s
default-token-2lpkw   kubernetes.io/service-account-token   3      16d

[root@master Secret]# kubectl describe secret/db-user-pass
Name:         db-user-pass
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password.txt:  12 bytes
username.txt:  5 bytes
[root@master Secret]# kubectl get secret/db-user-pass -o yaml
apiVersion: v1
data:
  password.txt: MWYyZDFlMmU2N2Rm
  username.txt: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2020-05-25T14:53:09Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:password.txt: {}
        f:username.txt: {}
      f:type: {}
    manager: kubectl
    operation: Update
    time: "2020-05-25T14:53:09Z"
  name: db-user-pass
  namespace: default
  resourceVersion: "922976"
  selfLink: /api/v1/namespaces/default/secrets/db-user-pass
  uid: 3153dbe6-ba0b-4110-b9a1-6dbeb3640c5f
type: Opaque

//解码 Secret
[root@master Secret]# echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
1f2d1e2e67df
[root@master Secret]# echo 'YWRtaW4=' | base64 --decode
admin
//对于某些情况，您可能希望改用 stringData 字段。此字段允许您将非 base64 编码的字符串直接放入 Secret 中， 并且在创建或更新 Secret 时将为您编码该字符串。

```

####Creating a Secret manually