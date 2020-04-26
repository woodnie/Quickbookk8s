开启ipvs的前置条件


```
modprobe br_netfiltr

[root@master ~]# cat > /etc/sysconfig/modules/ipvs.modules <<EOF
> #!/bin/bash
> modprobe -- ip_vs
> modprobe -- ip_vs_rr
> modprobe -- ip_vs_wrr
> modprobe -- ip_vs_sh
> modprobe -- nf_conntrack_ipv4
> EOF
[root@master ~]# chmod 755 /etc/sysconfig/modules/ipvs.modules
[root@master ~]# ll /etc/sysconfig/modules/ipvs.modules
-rwxr-xr-x. 1 root root 124 Apr 25 22:26 /etc/sysconfig/modules/ipvs.modules
[root@master ~]#
```