---
title: translate翻译网络问题
top: false
cover: false
toc: true
mathjax: true
date: 2022-10-20 10:01:21
password:
summary: translate翻译插件,谷歌翻译网络异常
tags:
- 原创
- translate
categories:
- IDEA插件
---


## 问题
translate翻译插件,谷歌翻译网络异常,翻译异常,更改hosts文件,设置ip

* 打开 `C:\Windows\System32\drivers\etc` 下的hosts文件夹
* 把下面两个粘贴进去
  ```text
    203.208.40.66 translate.google.com
    203.208.40.66 translate.googleapis.com
    ```