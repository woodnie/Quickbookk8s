`kubectl 常用命令：`



\[root@master ~\]\# kubectl api-resources //a complete list of supported resources

`  
kubectl get pods --all-namespaces      //查看kube-flannel 状态    running 为安装正常  
kubectl get pods --all-namespaces -o wide`

kubectl get namespaces`  
kubectl get nodes  
kubectl get cs  
kubectl get csr  
kubectl apply -f  xxxx.yaml  
kubectl create -f xxxx.yaml  
kubectl delete -f xxxxx.yaml`

