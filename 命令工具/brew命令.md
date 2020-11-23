---
title: brew命令
date: 2020-08-04 22:11:21
tags:
---

## 安装brew

### 安装

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

### 卸载

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
```

### 本地安装

如果发现因为"墙"的原因连接不上，如果发现因为"墙"的原因连接不上，

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

也可以使用国内镜像采用如下步骤进行本地安装：

```bash
# 下载源到本地
curl -fsSL https:/raw.githubusercontent.com/Homebrew/install/master/install >> brew_install
# 显修改镜像源
vi brew_install
```

打开 /usr/bin/ruby/brew_install 文件，进行如下修改：

```
BREW_REPO     = "https:/mirrors.tuna.tsinghua.edu.cn/brew.git".freeze
CORE_TAP_REPO = "https:/mirrors.tuna.tsinghua.edu.cn/homebrew-core.git".freeze
```

修改完毕，返回系统环境，安装

```
/usr/bin/ruby brew_install
```

### 镜像

如果速度实在太慢，可以考虑使用中科大镜像，使用如下命令：

```bash
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
```

### 清除镜像

清除镜像，使用原始链接，可以使用如下命令

```bash
# 复原
git -C "$(brew --repo)" remote set-url origin https:/github.com/Homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https:/github.com/Homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https:/github.com/Homebrew/homebrew-cask.git
```



## 常用命令

```
# 可以看到brew update工作的具体步骤.详细的下载信息
brew update -verbose
# 安装
brew install [package_name]
# 卸载
brew uninstall [package_name]
# 升级自身
brew update
# 升级
brew upgrade [package_name]
# 升级全部
brew upgrade
# 查看已安装列表
brew list
# 查看可升级列表
brew outdated
# 查找
brew search [package_name]
# 清理缓存
brew cleanup
# 查看信息
brew info
# 自身诊断
brew doctor
```



## 常见问题

### 一、 Brew 升级更新错误"Failed to install vendor Ruby."

```
Already downloaded: /Users/fengxing/Library/Caches/Homebrew/portable-ruby-2.6.3_2.yosemite.bottle.tar.gz
Error: Checksum mismatch.
Expected: b065e5e3783954f3e65d8d3a6377ca51649bfcfa21b356b0dd70490f74c6bd86
  Actual: f6211bd218c8ca23cbfe00faa644c92c9c5b0818b5cf280c44062e9e76d0725d
 Archive: /Users/fengxing/Library/Caches/Homebrew/portable-ruby-2.6.3_2.yosemite.bottle.tar.gz
To retry an incomplete download, remove the file above.
Error: Failed to upgrade Homebrew Portable Ruby!
➜  ~ rm -rf /Users/fengxing/Library/Caches/Homebrew/portable-ruby-2.6.3_2.yosemite.bottle.tar.gz
```

#### 解决方案

```
删除`Archive:`后面路径下的文件
rm -rf /Users/aici/Library/Caches/Homebrew/portable-ruby--2.6.3.mavericks.bottle.tar.gz
然后再重新使用 brew update 命令
如果还是不行考虑下搭一个全局梯子
```

