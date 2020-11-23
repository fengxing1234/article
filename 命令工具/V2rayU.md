---
title: V2rayU代理
date: 2020-08-04 22:11:21
tags:
---



### 问题一：突然出发翻墙，日志如下

```
main: failed to read config files: [/Applications/V2rayU.app/Contents/Resources/config.json] > v2ray.com/core/main/json: failed to execute v2ctl to convert config file. > v2ray.com/core/common/platform/ctlcmd: v2ctl doesn't exist > stat /Applications/V2rayU.app/Contents/Resources/v2ray-core/v2ctl: no such file or directory
```

#### 分析

```
 主动更新v2ray出问题了(每次唤醒或启动都有检查),原v2ray-core目录被删除了
```

\##具体原因 原因v2ray-core更改了release下载地址规则,原匹配规则: `var releaseUrl: String = "https://github.com/v2ray/v2ray-core/releases/download/${version}/v2ray-macos.zip"` 新地址规则: `https://github.com/v2ray/v2ray-core/releases/download/v4.27.0/v2ray-macos-64.zip`

#### 临时解决方案,下载最新v2ray-core进行替换

1. 下载最新v2ray-core: https://github.com/v2ray/v2ray-core/releases/download/v4.27.0/v2ray-macos-64.zip
2. 替换到 `/Applications/V2rayU.app/Contents/Resources/v2ray-core/`
3. 下载完后可使用命令行:

```cd
     unzip v2ray-macos-64.zip
     mv ~/Downloads/v2ray-macos-64 /Applications/V2rayU.app/Contents/Resources/v2ray-core/
```

#### 解决方案：

首先 `ls /Applications/V2rayU.app/Contents/Resources/v2ray-core` 如果没有v2ray-core目录 说明v2ray-core不知道为什么被删除了 解决方法是从[v2ray-core](https://github.com/v2ray/v2ray-core/releases/tag/v4.27.0) 下载最新的macos版release，确认hash之后解压到/Applications/V2rayU.app/Contents/Resources 并重命名为v2ray-core.

