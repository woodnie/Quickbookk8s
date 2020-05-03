##kubectl


```
[root@master ~]# kubectl

kubectl controls the Kubernetes cluster manager.

Find more information at: [https://kubernetes.io/docs/reference/kubectl/overview/](https://kubernetes.io/docs/reference/kubectl/overview/)

Basic Commands \(Beginner\):

create        Create a resource from a file or from stdin.

expose        Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service

run           Run a particular image on the cluster

set           Set specific features on objects

Basic Commands \(Intermediate\):

explain       Documentation of resources

get           Display one or many resources

edit          Edit a resource on the server

delete        Delete resources by filenames, stdin, resources and names, or by resources and label selector

Deploy Commands:

rollout       Manage the rollout of a resource

scale         Set a new size for a Deployment, ReplicaSet or Replication Controller

autoscale     Auto-scale a Deployment, ReplicaSet, or ReplicationController

Cluster Management Commands:

certificate   Modify certificate resources.

cluster-info  Display cluster info

top           Display Resource \(CPU/Memory/Storage\) usage.

cordon        Mark node as unschedulable

uncordon      Mark node as schedulable

drain         Drain node in preparation for maintenance

taint         Update the taints on one or more nodes

Troubleshooting and Debugging Commands:

describe      Show details of a specific resource or group of resources

logs          Print the logs for a container in a pod

attach        Attach to a running container

exec          Execute a command in a container

port-forward  Forward one or more local ports to a pod

proxy         Run a proxy to the Kubernetes API server

cp            Copy files and directories to and from containers.

auth          Inspect authorization

Advanced Commands:

diff          Diff live version against would-be applied version

apply         Apply a configuration to a resource by filename or stdin

patch         Update field\(s\) of a resource using strategic merge patch

replace       Replace a resource by filename or stdin

wait          Experimental: Wait for a specific condition on one or many resources.

convert       Convert config files between different API versions

kustomize     Build a kustomization target from a directory or a remote url.

Settings Commands:

label         Update the labels on a resource

annotate      Update the annotations on a resource

completion    Output shell completion code for the specified shell \(bash or zsh\)

Other Commands:

alpha         Commands for features in alpha

api-resources Print the supported API resources on the server

api-versions  Print the supported API versions on the server, in the form of "group/version"

config        Modify kubeconfig files

plugin        Provides utilities for interacting with plugins.

version       Print the client and server version information

Usage:

kubectl \[flags\] \[options\]

Use "kubectl &lt;command&gt; --help" for more information about a given command.

Use "kubectl options" for a list of global command-line options \(applies to all commands\).
```

[root@master ~]# kubectl

[root@master ~\]\# kubectl api-resources //a complete list of supported resources

kubectl get pods --all-namespaces      //查看kube-flannel 状态    running 为安装正常          
kubectl get pods --all-namespaces -o wide`

kubectl get namespaces`kubectl get nodes          
kubectl get cs          
kubectl get csr          
kubectl apply -f  xxxx.yaml          
kubectl create -f xxxx.yaml          
kubectl delete -f xxxxx.yaml`

4个十分有用的命令可以帮助你排查Pod的故障：

`kubectl logs <pod name>`能够帮助检索Pod的容器日志

`kubectl describe pod <pod name>`能够有效地检索与Pod相关的事件列表

`kubectl get pod <pod name>`对于提取存储在Kubernetes中的Pod的YAML定义十分有用

`kubectl exec -ti <pod name>bash`可以用于在Pod其中一个容器中运行一个交互式命令

