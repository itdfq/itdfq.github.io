---
title: 校验url图片的ä»大小和分辨率
top: false
cover: false
toc: true
mathjax: true
date: 2022-09-01 10:59:03
password:
summary: 校验url类型的图片的分辨率与大小
tags:
- 原创
categories:
- java
---
### 校验代码如下
```java

校验Url图片的大小和尺寸


public void checkImageUrlSize(String url) throws Exception {

    URL urls = new URL(url);
    URLConnection connection = urls.openConnection();
    connection.setDoOutput(true);
    //总大小字节
    int contentLength = connection.getContentLength();
    if (contentLength > ONE_M_BYTES) {
        throw new BaseAppException("图片大小不能超过1M");
    }
    BufferedImage sourceImg = ImageIO.read(connection.getInputStream());
    if (sourceImg.getWidth()<=400||sourceImg.getHeight()<=400){
        throw new BaseAppException("图片分辨率必须大于400*400");
    }
}
```