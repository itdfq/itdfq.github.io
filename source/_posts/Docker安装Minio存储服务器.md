---
title: Docker安装Minio存储服务器
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-18 11:59:57
password:
summary: 本文介绍如何使用docker进行Minio安装
tags:
- 原创
- docker
categories:
- 软件安装
---

### 拉取镜像的命令

```
docker pull minio/minio
```




### 自定义安装


```
 docker run -p 9000:9000 --name minio -d --restart=always -e "MINIO_ACCESS_KEY=root" -e "MINIO_SECRET_KEY=root" -v /home/data:/data -v /home/config:/root/.minio minio/minio server /data
```

### 查看安装logs信息

```
docker logs ea407f64546c（容器id）
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210417204843917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDE3Mjc2,size_16,color_FFFFFF,t_70)
### 打开浏览器输入地址进行登录
```
http://192.168.1.133:9000
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210417204945244.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDE3Mjc2,size_16,color_FFFFFF,t_70)
