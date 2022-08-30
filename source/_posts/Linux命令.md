---
title: Linux命令
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-18 17:02:51
password:
summary: 本文主要介绍一些服务器命令
tags:
- 原创
- Linux
categories:
- cmd命令
---
 - 查看防火墙开放端口

```
firewall-cmd --list-all
```

 - 开放9000端口

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

```
命令含义：

–zone #作用域

–add-port=80/tcp #添加端口，格式为：端口/通讯协议

–permanent #永久生效，没有此参数重启后失效

firewall-cmd --reload 并不中断用户连接，即不丢失状态信息
```

 - 重载防火墙

 

```
firewall-cmd --reload
```


 