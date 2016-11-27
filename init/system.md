##Init a system

Init a basic system is very simple following our guide.

###First of all

What we suggest for host server is CentOS 7. And we do our work compeletely on CentOS 7.
Never use CentOS 6 (the kernel is too old for docker).

In private, I suggest gentoo if possible.<(￣︶￣)>

Basic system setup method is :

```
# install epel mirror for CentOS
yum install epel-release -y
# update all system
yum update -y
# if possible, restart the system
init 6
```

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
# flannel data port for data transfering through VXLAN
iptables -I INPUT -p udp --dport 8472 -j ACCEPT
# save the rules, and notice, we do not open any tcp port for flannel
iptables-save > /etc/sysconfig/iptables
```


###Security Enhanced Linux

Under most conditions, we consider it as useless.

Here is a small script for disabling selinux:

```
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
setenforce 0 
``` 

###Network layer

When using docker, the dockerd arrange a private ip for ever container and no one can access it out from its host server.

We must set an network layer for docker before dockerd start.

OVS(OpenVSwitch), flannel, weave are the best choices.

Here is some opinions for choose between them:

* If possible, or you can handle this, choose OVS.
* If every server have an standalone IP (for example, internat IP), and they can access each other direactly, use flannel. It's simple.

We choose flannel with VXLAN mode. Container Tcp connections are based on flannel udp connections.

Flannel still have an mode like `master - slave` mode, which works the mesh network is only partially connected.

Here is an script to setup a flannel registration server. Only the registration server need to do this. 

```
## Here we commented the firewall setting.
## ensure tcp/2379 on tcp server.
## only etcd server need to do this. 
## etcd server need port tcp/2379.
## iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 2379 -j ACCEPT
## flannel VXLAN client need port udp/8472  
## iptables -I INPUT -p udp --dport 8472 -j ACCEPT

# install etcd
yum install etcd -y
# set up network device for etcd to listen (from localhost 127.0.0.1 to all device 0.0.0.0)
sed -i "s/ETCD_LISTEN_CLIENT_URLS=\"http\:\/\/localhost\:2379\"/ETCD_LISTEN_CLIENT_URLS=\"http\:\/\/0.0.0.0\:2379\"/g" /etc/etcd/etcd.conf
# start etcd
service etcd restart
chkconfig etcd on
# sleep for etcd start
sleep 2
# setup domain to use 
# (replace aaa.com with your domain, 
# replace network with the ip range you want to use or not corrupt with your IDC, 
# set up flannel network mode VXLAN)
etcdctl set /aaa.com/network/config '{"Network": "10.0.0.0/8", "Backend": {"Type": "vxlan"}}'
```

Here is an script to setup a flannel network.

```
### you must ensure you have an etcd server with ip or domain, for example 8.8.8.8:2379 or etcd.aaa.com:2379
# install flannel
yum install flannel -y
# setup flannel etcd server for flannel
sed -i "s/127.0.0.1/etcd.aaa.com/g" /etc/sysconfig/flanneld
# start and and auto-start
service flanneld restart
chkconfig flanneld on
```

It's over.

###Container

Just use docker.

Installation and setup guide:

```
yum install docker -y
service docker restart
chkconfig docker on
```

Yes it's over, with no more need to set docker network or docker-flannel compatibility.


