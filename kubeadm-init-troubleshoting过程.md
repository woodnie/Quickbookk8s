`## WARNING Firewalld`

`[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly`

`# Solution1:开放端口`

`sudo firewall-cmd --permanent --add-port=6443/tcp && sudo firewall-cmd --permanent --add-port=10250/tcp && sudo firewall-cmd --reload`

`# Solution2:`

`systemctl disable firewalld && systemctl stop firewalld`

~~iptables 是否需要？~~

`## 调整内核参数`

![](/assets/k8s-kernel.png)

`sudo swapoff -a`

`sudo sed -i '/ swap / s/^/#/' /etc/fstab`

`cat > kubernetes.conf <<EOF`

`vm.swappiness=0`

