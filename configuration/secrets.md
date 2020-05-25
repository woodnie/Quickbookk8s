##Built-in Secrets
Service Account 使用 API 凭证自动创建和附加 secret
Kubernetes 自动创建包含访问 API 凭据的 secret，并自动修改您的 pod 以使用此类型的 secret。

如果需要，可以禁用或覆盖自动创建和使用 API 凭据。但是，如果您需要的只是安全地访问 apiserver，我们推荐这样的工作流程。

参阅 Service Account 文档获取关于 Service Account 如何工作的更多信息

##Creating a Secret Using kubectl

**假设有些 pod 需要访问数据库。这些 pod 需要使用的用户名和密码存放在如下文件**
```
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
```
**解码 Secret**
```
[root@master Secret]# echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
1f2d1e2e67df
[root@master Secret]# echo 'YWRtaW4=' | base64 --decode
admin
```

** 编码注意：** 
+ 对于某些情况，您可能希望改用 stringData 字段。此字段允许您将非 base64 编码的字符串直接放入 Secret 中， 并且在创建或更新 Secret 时将为您编码该字符串。
data 和 stringData 的键必须由字母数字字符 ‘-', ‘_’ 或者 ‘.’ 组成。

+ 秘密数据的序列化 JSON 和 YAML 值被编码为 base64 字符串。换行符在这些字符串中无效，因此必须省略。在 Darwin/macOS 上使用 base64 实用程序时，用户应避免使用 -b 选项来分隔长行。相反，Linux用户 *应该* 在 base64 命令中添加选项 -w 0 ，或者，如果 -w 选项不可用的情况下，执行 base64 | tr -d '\n'。


##Creating a Secret manually

```
[root@master Secret]# echo -n 'admin' | base64
YWRtaW4=
[root@master Secret]# echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm

[root@master Secret]# cat mysecret1.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret1
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm

[root@master Secret]# kubectl apply -f mysecret1.yaml
secret/mysecret1 created


[root@master Secret]# kubectl get secret/mysecret1
NAME       TYPE     DATA   AGE
mysecret1   Opaque   2      29s
[root@master Secret]# kubectl describe secret/mysecret1
Name:         mysecret1
Namespace:    default
Labels:       <none>
Annotations:
Type:         Opaque

Data
====
password:  12 bytes
username:  5 bytes

````
####编辑secret，并允许更新 data 字段中的 base64 编码的 secret：

```
[root@master Secret]# kubectl edit secrets mysecret1
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  password: MWYyZDFlMmU2N2Rm
  username: YWRtaW4=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"password":"MWYyZDFlMmU2N2Rm","username":"YWRtaW4="},"kind":"Secret","metadata":{"annotations":{},"name":"mysecret1","namespace":"default"},"type":"Opaque"}
  creationTimestamp: "2020-05-25T15:17:21Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:password: {}
        f:username: {}
      f:metadata:
        f:annotations:
          .: {}
          f:kubectl.kubernetes.io/last-applied-configuration: {}
      f:type: {}
    manager: kubectl
    operation: Update
    time: "2020-05-25T15:17:21Z"
  name: mysecret1
  namespace: default
  resourceVersion: "926264"
  selfLink: /api/v1/namespaces/default/secrets/mysecret1
  uid: a0b213cc-7955-4c7f-9938-da515acaaa7b
type: Opaque

```

####使用 Secret
[root@master Secret]# cat my-secret-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: myapp:v1
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret1

[root@master Secret]# kubectl apply -f my-secret-pod.yaml
pod/mypod created

[root@master Secret]# kubectl exec mypod -it -- ls -l /etc/foo
total 0
lrwxrwxrwx. 1 root root 15 May 25 15:40 password -> ..data/password
lrwxrwxrwx. 1 root root 15 May 25 15:40 username -> ..data/username

[root@master Secret]# kubectl exec mypod -it -- cat /etc/foo/password
1f2d1e2e67df

[root@master Secret]# kubectl exec mypod -it -- cat /etc/foo/username
admin

**向特性路径映射 secret 密钥**

我们还可以控制 Secret key 映射在 volume 中的路径。您可以使用 spec.volumes[].secret.items 字段修改每个 key 的目标路径：

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
 ```       


username secret 存储在 /etc/foo/my-group/my-username 文件中而不是 /etc/foo/username 中。
password secret 没有被映射

**Secret 文件权限**

您还可以指定 secret 将拥有的权限模式位文件。如果不指定，默认使用 0644。您可以为整个保密卷指定默认模式，如果需要，可以覆盖每个密钥。

例如，您可以指定如下默认模式：
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      defaultMode: 256
```
然后，secret 将被挂载到 /etc/foo 目录，所有通过该 secret volume 挂载创建的文件的权限都是 0400。

请注意，JSON 规范不支持八进制符号，因此使用 256 值作为 0400 权限。如果您使用 yaml 而不是 json 作为 pod，则可以使用八进制符号以更自然的方式指定权限。

您还可以使用映射，如上一个示例，并为不同的文件指定不同的权限，如下所示：
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
        mode: 511
```
在这种情况下，导致 /etc/foo/my-group/my-username 的文件的权限值为 0777。由于 JSON 限制，必须以十进制格式指定模式。

####Secret 作为环境变量
将 secret 作为 pod 中的环境变量使用：

创建一个 secret 或者使用一个已存在的 secret。多个 pod 可以引用同一个 secret。
修改 Pod 定义，为每个要使用 secret 的容器添加对应 secret key 的环境变量。消费secret key 的环境变量应填充 secret 的名称，并键入 env[x].valueFrom.secretKeyRef。
修改镜像并／或者命令行，以便程序在指定的环境变量中查找值。
这是一个使用 Secret 作为环境变量的示例：
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```
消费环境变量里的 Secret 值

在一个消耗环境变量 secret 的容器中，secret key 作为包含 secret 数据的 base-64 解码值的常规环境变量。这是从上面的示例在容器内执行的命令的结果：
```
echo $SECRET_USERNAME
admin
echo $SECRET_PASSWORD
1f2d1e2e67df
```