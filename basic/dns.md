##DNS

DNS server is import for an company. You can host your business domain at a host service as godaddy, however, you should not host your internal domain in public domain services.

For example, the alibaba.com's business domain is alibaba.com, however, their internal domain is alibaba-inc.com, and their internal git is gitlab.alibaba-inc.com. You can **never** access it from internet without their VPN. Their internal calls, are all hold by their private DNS server.

As devops, we do not setup a dnsmasq or bind.

Here is a choice for it:

[A fast dns server written by go](https://github.com/kenshinx/godns)
[DNS server with per-client targeted responses](https://github.com/abh/geodns)
