通过dockerhub 自造轮子访问gcr.io镜像

通过dockerhub访问gcr.io镜像

https://segmentfault.com/a/1190000015377624



前言
在docker的学习的过程中，遇到的第一个拦路虎就是很多gcr.io的镜像无法下载，原因很简单，就是google被墙了,常用的解决方案有：
1、找一台国外的服务器（各种公有云服务器），必须有公网IP，通过docker pull镜像后，然后docker save保存为tar包，再传输到本地，通过docker load导入；缺点：首先得有一台服务器，而且每次操作都很麻烦。
2、通过国内一些公开的镜像服务器，寻找别人上传的镜像下载，缺点：匹配的镜像版本很难找，而且镜像本身的信息是否安全， 也不好说。
3、通过翻墙软件去访问国外服务器，缺点：目前打击力度很大，免费的很难找到，专业的价格又高。
4、通过dockerhub曲线救国，因为dockerhub的服务器在国外，通过dockerhub的自动构建功能，结合github仓库，基于Dockerfile，构建docker镜像，然后可以从dockerhub获取docker pull镜像到本地。

本文我们主要介绍一下第四种方法。


## pull
docker pull registry.cn-hangzhou.aliyuncs.com/woodnie/kube-apiserver:v1.18.2
docker pull registry.cn-hangzhou.aliyuncs.com/woodnie/kube-controller-manager:v1.18.2
docker pull registry.cn-hangzhou.aliyuncs.com/woodnie/kube-scheduler:v1.18.2
docker pull registry.cn-hangzhou.aliyuncs.com/woodnie/kube-proxy:v1.18.2
docker pull registry.cn-hangzhou.aliyuncs.com/woodnie/pause:3.2
docker pull registry.cn-hangzhou.aliyuncs.com/woodnie/etcd:3.4.3-0
docker pull registry.cn-hangzhou.aliyuncs.com/woodnie/coredns:1.6.7

## Tag
sudo docker tag  registry.cn-hangzhou.aliyuncs.com/woodnie/kube-apiserver:v1.18.2 k8s.gcr.io/kube-apiserver:v1.18.2
sudo docker tag  registry.cn-hangzhou.aliyuncs.com/woodnie/kube-controller-manager:v1.18.2 k8s.gcr.io/kube-controller-manager:v1.18.2
sudo docker tag  registry.cn-hangzhou.aliyuncs.com/woodnie/kube-scheduler:v1.18.2 k8s.gcr.io/kube-scheduler:v1.18.2
sudo docker tag  registry.cn-hangzhou.aliyuncs.com/woodnie/kube-proxy:v1.18.2 k8s.gcr.io/kube-proxy:v1.18.2
sudo docker tag  registry.cn-hangzhou.aliyuncs.com/woodnie/pause:3.2 k8s.gcr.io/pause:3.2
sudo docker tag  registry.cn-hangzhou.aliyuncs.com/woodnie/etcd:3.4.3-0 k8s.gcr.io/etcd:3.4.3-0
sudo docker tag  registry.cn-hangzhou.aliyuncs.com/woodnie/coredns:1.6.7 k8s.gcr.io/coredns:1.6.7

## rmi
sudo docker rmi registry.cn-hangzhou.aliyuncs.com/woodnie/kube-apiserver:v1.18.2
sudo docker rmi registry.cn-hangzhou.aliyuncs.com/woodnie/kube-controller-manager:v1.18.2
sudo docker rmi registry.cn-hangzhou.aliyuncs.com/woodnie/kube-scheduler:v1.18.2
sudo docker rmi registry.cn-hangzhou.aliyuncs.com/woodnie/kube-proxy:v1.18.2
sodu docker rmi registry.cn-hangzhou.aliyuncs.com/woodnie/pause:3.2
sodu docker rmi registry.cn-hangzhou.aliyuncs.com/woodnie/etcd:3.4.3-0
sodu docker rmi registry.cn-hangzhou.aliyuncs.com/woodnie/coredns:1.6.7

