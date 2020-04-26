通过dockerhub 已有轮子访问gcr.io镜像

版权声明：本文为CSDN博主「醉生梦死一笑惊魂」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/lfm940624/java/article/details/86629147

https://hub.docker.com/u/mirrorgooglecontainers
https://hub.docker.com/u/kubeimage


#!/bin/bash
K8S_VERSION=v1.13.1
ETCD_VERSION=3.2.24
DASHBOARD_VERSION=v1.8.3
FLANNEL_VERSION=v0.10.0-amd64
DNS_VERSION=1.2.0
PAUSE_VERSION=3.1
# 基本组件
docker pull mirrorgooglecontainers/kube-apiserver-amd64:$K8S_VERSION
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:$K8S_VERSION
docker pull mirrorgooglecontainers/kube-scheduler-amd64:$K8S_VERSION
docker pull mirrorgooglecontainers/kube-proxy-amd64:$K8S_VERSION
docker pull mirrorgooglecontainers/etcd-amd64:$ETCD_VERSION
docker pull mirrorgooglecontainers/pause:$PAUSE_VERSION
docker pull coredns/coredns:$DNS_VERSION

# 网络组件
docker pull quay.io/coreos/flannel:$FLANNEL_VERSION

# 修改tag
docker tag mirrorgooglecontainers/kube-apiserveramd64:$K8S_VERSION k8s.gcr.io/kube-apiserver-amd64:$K8S_VERSION
docker tag mirrorgooglecontainers/kube-controller-manageramd64:$K8S_VERSION k8s.gcr.io/kube-controller-manager-amd64:$K8S_VERSION
docker tag mirrorgooglecontainers/kube-scheduleramd64:$K8S_VERSION k8s.gcr.io/kube-scheduler-amd64:$K8S_VERSION
docker tag mirrorgooglecontainers/kube-proxyamd64:$K8S_VERSION k8s.gcr.io/kube-proxy-amd64:$K8S_VERSION
docker tag mirrorgooglecontainers/etcd-amd64:$ETCD_VERSION k8s.gcr.io/etcd-amd64:$ETCD_VERSION
docker tag mirrorgooglecontainers/pause:$PAUSE_VERSION k8s.gcr.io/pause:$PAUSE_VERSION
docker tag coredns/coredns:$DNS_VERSION k8s.gcr.io/coredns:$DNS_VERSION

#删除冗余的images
docker rmi mirrorgooglecontainers/kube-apiserver-amd64:$K8S_VERSION
docker rmi mirrorgooglecontainers/kube-controller-manager-amd64:$K8S_VERSION
docker rmi mirrorgooglecontainers/kube-scheduler-amd64:$K8S_VERSION
docker rmi mirrorgooglecontainers/kube-proxy-amd64:$K8S_VERSION
docker rmi mirrorgooglecontainers/etcd-amd64:$ETCD_VERSION
docker rmi mirrorgooglecontainers/pause:$PAUSE_VERSION
docker rmi coredns/coredns:$DNS_VERSION
————————————————

