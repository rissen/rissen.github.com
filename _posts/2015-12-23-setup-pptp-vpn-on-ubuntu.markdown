---
layout: posts
title: "ubuntu搭建PPTP VPN"
description: "ubuntu搭建PPTP VPN"
category: linux
tags: [vpn,pptp]
---

# Uubuntu搭建PPTP VPN

## 安装pptpd
```
sudo apt-get update
sudo apt-get install pptpd
```

## 修改pptpd.conf的localip和remoteip
```
sudo vim /etc/pptpd.conf
```

localip是主机的ip,如果是租用的vps,则是vps的公网ip

```
localip 192.162.59.61
remoteip 192.168.0.234-240,192.168.0.245
```
## 修改dnsu,使用的google dns
```
sudo vim /etc/ppp/pptpd-options
```

```
ms-dns 8.8.8.8
```

## 设置用户账号
```
sudo vim /etc/ppp/chap-secrets
```

配置了2个账号，用户名分别是vpn1,vpn2,密码是vpn1Pwd,分别分配了ip 192.168.0.235,192.168.0.235.

```
# Secrets for authentication using CHAP
# client	server	secret			IP addresses
vpn1	pptpd	vpn1Pwd	192.168.0.235
vpn2	pptpd	vpn2Pwd	192.168.0.236
```

## 重启pptpd服务
```
sudo service pptpd start
```

## 转发设置
```
iptables -t nat -F
```
-s后面的ip是上面remoteip 段的ip
--to后面跟着的ip地址是上面设置的localip

```
iptables -t nat -A POSTROUTING -s 192.168.0.235,192.168.0.236 -j SNAT --to 192.162.59.61
```

## References
[PPTPServer](https://help.ubuntu.com/community/PPTPServer)
