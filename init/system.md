##Init a system

Init a basic system is very simple following our guide.

###First of all

What we suggest for host server is CentOS 7. And we do our work compeletely on CentOS 7.
Never use CentOS 6 (the kernel is too old for docker).

In private, I suggest gentoo if possible.<(￣︶￣)>

###Basic serivces

First of all, we need the following services:

* A firewall
* A Container Engine
* A Network Layer

###Firewall

Firewall is needed for most conditions. Here we suggest iptables in place of firewalld because:

* iptables is simpler than firewalld.
* firewalld is based on iptables and, is an abstract layer over iptables.<(￣︶￣)>
* We want native firewalld, however we have some problems when dealing with the backend iptables rule `FORWARD reject`.(╯‵□′)╯︵┻━┻

Here is an script for auto-replace firewalld with iptables on CentOS 7.
You **should**  execute it by a `bash init.sh` and **should not** execute it line by line.


```
# stop and set auto-start off
service firewalld stop  
chkconfig firewalld off

# install iptables
yum install iptables-services  -y

# start iptables and set auto-start on
service iptables start 
chkconfig iptables on

# reset all rules and init rules
iptables -F
iptables -X
iptables -P INPUT DROP
iptables -t nat -F
iptables -t nat -X
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
# ping
iptables -A INPUT -p icmp -j ACCEPT
# lo device must be accepted
iptables -A INPUT -i lo -j ACCEPT
# port tcp/22 is for SSH
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
# port tcp/80 is for HTTP server, you can turn it off
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
# etcd server need port tcp/2379
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 2379 -j ACCEPT
# flannel data port for data transfering, which is udp 8285
iptables -I INPUT -p udp --dport 8285 -j ACCEPT
# flannel data port for data transfering through VXLAN, whch is considered better than 8285, choose one
iptables -I INPUT -p udp --dport 8472 -j ACCEPT
# save the rules
iptables-save > /etc/sysconfig/iptables
```