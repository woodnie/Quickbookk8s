kubeadm init troubleshoting过程

## WARNING Firewalld
[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly

# Solution1:
sudo firewall-cmd --permanent --add-port=6443/tcp && sudo firewall-cmd --permanent --add-port=10250/tcp && sudo firewall-cmd --reload


# Solution2:
![a22f3a734da6ef11ac74c329f9cf0452.png](../_resources/858583dc43bd461681050225849b9a13.png)

systemctl disable firewalld && systemctl stop firewalld

## 调整内核参数

![081afb957493260dec8255bb0628e859.png](../_resources/4d6c3929695e4e4b9ba0198ce79beb9d.png)

sudo swapoff -a  
sudo sed -i '/ swap / s/^/#/' /etc/fstab

cat > kubernetes.conf <<EOF
vm.swappiness=0 



