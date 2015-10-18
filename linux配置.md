#####1. ip配置
`vi /etc/sysconfig/network-scripts/ifcfg-eno1677756`  
BOOTPROTO=none  
IPADDR=192.168.116.130  
NETMASK=255.255.255.0  
GATEWAY=192.168.116.1  
HWADDR=00:50:56:25:05:dc  

#####2.防火墙  
cent7使用fireware而不是iptables
systemctl start firewalld.service	#启动firewall
systemctl stop firewalld.service	#停止firewall
systemctl disable firewalld.service	#禁止firewall开机启动
