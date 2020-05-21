
```
[root@master ~]# cat replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: frontend-rs
 labels:
  app: guestbook
  tier: frontend-rs
spec:
 replicas: 2
 selector:
  matchLabels:
   tier: frontend-rs
 template:
  metadata:
   labels:
    tier: frontend-rs
  spec:
   containers:
   - name: nginx-rs
     image: nginx:v1
```

```
[root@master ~]# kubectl get rs
NAME          DESIRED   CURRENT   READY   AGE
frontend-rs   2         2         2       21h
[root@master ~]#

```

```
[root@master ~]# kubectl describe rs/frontend-rs
Name:         frontend-rs
Namespace:    default
Selector:     tier=frontend-rs
Labels:       app=guestbook
              tier=frontend-rs
Annotations:  <none>
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend-rs
  Containers:
   nginx-rs:
    Image:        nginx:v1
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:           <none>
[root@master ~]#

```
```
[root@master ~]# kubectl get pod -o wide
I0521 21:05:49.183294   44442 request.go:621] Throttling request took 1.324335914s, request: GET:https://192.168.1.100:6443/apis/policy/v1beta1?timeout=32s
NAME                READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
frontend-rs-bfzng   1/1     Running   0          20h   10.244.2.7   node02.wood.com   <none>           <none>
frontend-rs-czgkd   1/1     Running   0          20h   10.244.1.4   node01.wood.com   <none>           <none>
```

```
[root@master ~]# kubectl get pod frontend-rs-bfzng -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-05-20T16:07:05Z"
  generateName: frontend-rs-
  labels:
    tier: frontend-rs
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:generateName: {}
        f:labels:
          .: {}
          f:tier: {}
        f:ownerReferences:
          .: {}
          k:{"uid":"2afac965-8d50-46ef-9af6-6b7dad9b82a2"}:
            .: {}
            f:apiVersion: {}
            f:blockOwnerDeletion: {}
            f:controller: {}
            f:kind: {}
            f:name: {}
            f:uid: {}
      f:spec:
        f:containers:
          k:{"name":"nginx-rs"}:
            .: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:name: {}
            f:resources: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
    manager: kube-controller-manager
    operation: Update
    time: "2020-05-20T16:07:05Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:conditions:
          k:{"type":"ContainersReady"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
        f:containerStatuses: {}
        f:hostIP: {}
        f:phase: {}
        f:podIP: {}
        f:podIPs:
          .: {}
          k:{"ip":"10.244.2.7"}:
            .: {}
            f:ip: {}
        f:startTime: {}
    manager: kubelet
    operation: Update
    time: "2020-05-20T16:09:20Z"
  name: frontend-rs-bfzng
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: frontend-rs
    uid: 2afac965-8d50-46ef-9af6-6b7dad9b82a2
  resourceVersion: "565201"
  selfLink: /api/v1/namespaces/default/pods/frontend-rs-bfzng
  uid: f0f03719-edef-4685-ba07-42e1508a9aa4
spec:
  containers:
  - image: nginx:v1
    imagePullPolicy: IfNotPresent
    name: nginx-rs
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-2lpkw
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: node02.wood.com
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-2lpkw
    secret:
      defaultMode: 420
      secretName: default-token-2lpkw
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2020-05-20T16:09:14Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2020-05-20T16:09:20Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2020-05-20T16:09:20Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2020-05-20T16:09:14Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://efcd2262a62864a7ff5cabeef7d0e4b24228aa2052b5a2540f8b9a0be2742784
    image: nginx:latest
    imageID: docker-pullable://nginx@sha256:86ae264c3f4acb99b2dee4d0098c40cb8c46dcf9e1148f05d3a51c4df6758c12
    lastState: {}
    name: nginx-rs
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2020-05-20T16:09:19Z"
  hostIP: 192.168.1.102
  phase: Running
  podIP: 10.244.2.7
  podIPs:
  - ip: 10.244.2.7
  qosClass: BestEffort
  startTime: "2020-05-20T16:09:14Z"
[root@master ~]#

```