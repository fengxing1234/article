---
title: 解决Github上clone项目太慢
date: 2020-08-04 21:42:54
tags:
---

## 解决办法一

git clone特别慢是因为github.global.ssl.fastly.net域名被限制了。

只要找到这个域名对应的ip地址，然后在hosts文件中加上ip–>域名的映射，刷新DNS缓存便可。

### 步骤一 搜索对应IP

在 [www.ipaddress.com/](https://www.ipaddress.com/) 上分别搜索

- github.global.ssl.fastly.net
- github.com

这两个域名对应的ip

![image.png](https://upload-images.jianshu.io/upload_images/4118241-eee27d76df7eb2bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/4118241-a0f9eaa7fa2d75ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 步骤二 更改hosts文件

**Windows上使用管理员打开记事本然后打开上述文件路径的文件从而更改。**

- Windows上的hosts文件路径在C:\Windows\System32\drivers\etc\hosts
- Linux的hosts文件路径在：sudo vim /etc/hosts

在文件中添加

199.232.69.194 github.global-ssl.fastly.net

140.82.113.3 github.com

### 步骤三 保存更新DNS

- Winodws系统的做法：打开CMD，输入ipconfig /flushdns

- Linux的做法：在终端输入sudo /etc/init.d/networking restart
- sudo killall -HUP mDNSResponder

### 结果

![image.png](https://upload-images.jianshu.io/upload_images/4118241-1ac9d99c1eca5ca7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 解决办法二

在Gitee 上克隆Github仓库 再从Gitee clone 就很快。然后修改远程地址。

