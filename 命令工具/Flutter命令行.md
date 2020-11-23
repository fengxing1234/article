---
title: Flutter命令行
date: 2020-10-27 17:59:31
tags:
---

工欲善其事，必先利其器。 工匠想要使他的工作做好，一定要**先**让工具锋利。 比喻要做好一件事，准备工具非常重要.在软件开发领域也是一样的,如果想入门一个框架或者语言,都必须先了解附加的工具如何使用.flutter命令是flutter框架重要部分之一

## 一、环境相关

### 1.1 flutter channel

切换flutter不同的版本,在执行flutter channel会输出不同分支信息,默认使用stable分支

```bash
flutter channel # 输出channel
flutter channel dev # 切换到dev channel
```

### 1.2 flutter doctor

检查开发工具链是否完整安装,对于安装环境非常有用处,检查andorid licenses需要连接外网,请[科学上网](https://www.phantomsocks.club/)

```bash
flutter doctor

Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, v1.2.1, on Linux, locale en_US.UTF-8)
[✓] Android toolchain - develop for Android devices (Android SDK version 28.0.3)
[✓] Android Studio (version 3.3)
[✓] VS Code (version 1.33.1)
[✓] Connected device (2 available)

• No issues found!
```

### 1.3 flutter upgrade

更新flutter代码,实质就是git代码更新拉取,下载flutter sdk是git仓库的打包

## 二、项目相关

### 2.1 flutter create

在指定的目录中,创建新的flutter项目,如果没有指定目录,则在当前目录下创建项目

```bash
flutter create ~/flutter #在家目录下的flutter目录项目
flutter create . #在当前目录下创建
```

### 2.2 flutter build

构建应用程序的apk,appbundle,aot,ios,IOS应用需要在Mac osx上构建

```bash
flutter build apk
```

### 2.3 flutter clean

删除`build/`和`.dart_tool/`目录,清除缓存信息,避免之前不同版本代码的影响

### 2.4 flutter run 

运行项目



## 三、开发相关

### 3.1 flutter analyze

编辑Flutter代码时，使用分析器检查代码是非常重要,默认是分析整个项目的代码,你也可以通过使用`analysis_options.yaml`文件来排除不需要的代码分析

```text
analysis_options.yaml
analyzer:
  exclude:
    - flutter/**
```

有时候你可能需要代码分析一直在运行,可以使用--watch选项

```bash
flutter analyze --watch
```

当运行分析命令flutter都执行一次`pub get`命令,如果不需要运行,可以执行以下命令

```bash
flutter analyze --no-pub 
```





## 四、配置相关

