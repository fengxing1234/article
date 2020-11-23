---
title: WebView相关
date: 2020-11-16 13:52:26
tags:
---



#### 问：CookieSyncManager与CookieManager

答：早期的cookie是由CookieSyncManager进行管理的，之后CookieSyncManager被抛弃了，换成了CookieManager来进行管理。两个版本的分割线就是Android SDK -- 21。

#### 问：Android中Cookie的存储位置？

答：目前Android系统WebView是将cookie存储data/data/package_name/app_webview这个目录下的一个叫Cookies的数据中。



#### 问：说说cookie同步

答：CookieSyncManager在内存和存储器之间同步浏览器的cookie。另外，CookieSyncManager的同步策略是在一个独立的线程里定时进行同步。每次同步的时间间隔是5分钟。



#### 问：