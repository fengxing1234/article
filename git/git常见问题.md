---
title: git常见问题
date: 2020-06-03 11:22:41
tags: 
	- 命令行
	- git常见问题
categories: 
	- git
---

# git常见问题记录



## Empty reply from server

问题：fatal: unable to access 'http://xxx.git/': Empty reply from server

原因：因为设置了代理，所以找不到地址

解决：取消全局代理

```
git config --global --unset http.proxy
```

## error: src refspec xxx matches more than one.

问题：error: src refspec v330 matches more than one.

原因：出现这个错误之前，是在远程服务器上创建了一个tag v330，同时本地分支也是 v330，

解决：删除tag，或者另起一名

```
git tag -d v330
```

