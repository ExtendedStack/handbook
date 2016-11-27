## Init system environment


## Basic adjustment

```shell
## install epel mirror and update system
yum install epel-release -y 
yum update -y
# disable postfix
systemctl disable postfix
chkconfig postfix off
# install denyhosts
yum install denyhosts
# basic system administration tools
yum install net-tools telnet dstat iftop sysstat iotop htop psmisc ntp -y 

# reset system time (be careful)
service ntpd stop
ntpdate time.apple.com
service ntpd start
```


## Environment variables

```shell
# write the following to a file to log every command into /var/log/message
export PROMPT_COMMAND="history -a; history 1|logger -i -t '`who -u am i|awk '{print $1 $NF}'`'; $PROMPT_COMMAND"
# write the following to a file to make shell colorful
export PS1='\n\e[1;37m[\e[m\e[1;32m\u\e[m\e[1;33m@\e[m\e[1;35m\h\e[m \e[4m`pwd`\e[m\e[1;37m]\e[m\e[1;36m\e[m \A\n$ '
# setup grub resolution when using virtualbox or vmware
sed -i -c '/vmlinuz/s/$/ vga=0x340/' /etc/grub2.cfg 
```

## SSH key

```shell
# write a shell public key to /root/.ssh/authorized_keys 
```


