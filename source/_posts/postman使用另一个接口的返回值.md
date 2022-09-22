---
title: postman使用另一个接口的返回值
top: false
cover: false
toc: true
mathjax: true
date: 2022-09-22 09:35:08
password:
summary: 本文是为了解决，postman请求的参数是从另一个接口获取，通过javaScript代码，将另一个接口的返回值存储于全局变量，使用postman获取变量的命令获取参数值，具体请看本文详细介绍
tags:
- 原创
- postman
categories:
- 工具
---
### 需求

1. 首先调用接口，可以拿到token
   ![请求测试](https://img-blog.csdnimg.cn/bf83685a5cca418d9b363e0e53bf9cc9.png)


2. 首先调用接口，可以拿到token

   ![请求需要的参数](https://img-blog.csdnimg.cn/9608b94554e340c29fcf5c4b5310c22f.png)






### 实现逻辑

代码：

```javascript
// #将返回值存在一个data里 pm为postman的内置函数 pm.response意思为获取这个请求的返回值 .json()为存为json格式 后续可以通过键来取值
var jsondata = pm.response.json()
 
// # 获取变量jsondata的data里的path值 存为变量
var token=jsondata.access_token
 
// # 打印 可以省略
console.log(token)
 
// # 设置全局变量 也可以设置环境变量  
pm.globals.set("token", token);
 
// # 设置环境变量 
// # pm.environment.set("variable_key", "variable_value");
```

生成的token可以在 全局变量中查到

使用`{{token}}`可以获取到设置的变量信息