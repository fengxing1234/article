---
title: gradle命令行
date: 2020-09-08 10:21:49
tags:
---

## 概述

Gradle是一个灵活和高效自动化构建工具，它的构建脚本采用Groovy或kotlin语言编写,Groovy或Kotlin都是基于JVM的语言，它们的语法和java的语法有很多的类似并且兼容java的语法，同时Gradle也是Android官方的构建工具。目前Android Studio中建立的工程都是基于gradle进行构建的。Gradle的与其他构建工具（ant、maven）的特性主要包括：

- 强大的DSL和丰富的gradle的API
- gradle就是groovy
- 强大的依赖管理
- 可拓展性
- 与其他构建工具的集成




- 在 Windows 上：

  ```
  gradlew task-name
  ```

- 在 Mac 或 Linux 上：

  ```
  ./gradlew task-name
  ```

## 一、命令清单

```
# 帮助信息
gradle --help

# 查看版本 
gradle -v

# 执行特定的任务
gradle [taskName]

# 构建 
gradle build

# 跳过测试构建构建
gradle build -x test

# 继续执行任务而忽略前面失败的任务 
gradle build --continue

# 试运行build 
gradle -m build

# 产生build运行时间的报告 结果存储在build/report/profile目录，名称为build运行的时间。
gradle build --profile 

# 显示任务间的依赖关系 
gradle tasks --all

# 查看testCompile的依赖关系 
gradle -q dependencies --configuration testCompile

# 清空所有编译、打包生成的文件(即：清空build目录) 
gradle clean

# 使用指定的Gradle文件调用任务 
gradle -b [file_path] [task]

# 使用指定的目录调用任务 
gradle -q -p [dir] helloWorld

# Gradle的图形界面
gradle --gui


```

## gradle打包实战

- 操作系统：Mac
- 打包后的apk文件在`app–>build–>outputs—>apk`中
- 使用gradlew时可能出现没有找到该命令，需要`chmod 755 gradlew`

### 1.1 查看当前项目所用的Gradle版本

`./gradlew -v`

```
nmcp_android git:(pluginApk) ./gradlew -v

------------------------------------------------------------
Gradle 5.4.1
------------------------------------------------------------

Build time:   2019-04-26 08:14:42 UTC
Revision:     261d171646b36a6a28d5a19a69676cd098a4c19d

Kotlin:       1.3.21
Groovy:       2.5.4
Ant:          Apache Ant(TM) version 1.9.13 compiled on July 10 2018
JVM:          1.8.0_40 (Oracle Corporation 25.40-b25)
OS:           Mac OS X 10.13.5 x86_64

```

### 1.2 清除app目录下的build文件夹

`./gradlew clean`

```
➜  nmcp_android git:(pluginApk) ./gradlew clean

> Configure project :flutter
Observed package id 'ndk-bundle' in inconsistent location '/Users/fengxing/Library/Android/sdk/ndk-bundle-19' (Expected '/Users/fengxing/Library/Android/sdk/ndk-bundle')

BUILD SUCCESSFUL in 14s
11 actionable tasks: 10 executed, 1 up-to-date
➜  nmcp_android git:(pluginApk)
```

### 1.3 编译项目并生成相应的apk文件

`./gradlew build --stacktrace`

- --stacktrace 可选参数，表示打印详细信息

```
➜  nmcp_android git:(pluginApk) ./gradlew build --stacktrace

> Task :flutter:compileFlutterBuildDebug
  ╔════════════════════════════════════════════════════════════════════════════╗
  ║ A new version of Flutter is available!                                     ║
  ║                                                                            ║
  ║ To update to the latest version, run "flutter upgrade".                    ║
  ╚════════════════════════════════════════════════════════════════════════════╝
> Task :common:compileDebugJavaWithJavac
...
> Task :camera:compileDebugJavaWithJavac
...
> Task :library:compileDebugJavaWithJavac
...
> Task :login:compileDebugJavaWithJavac
...
> Task :media:compileDebugJavaWithJavac
> Task :tmf:compileDebugJavaWithJavac
注: 某些输入文件使用或覆盖了已过时的 API。
注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
> Task :app:processDebugManifest
/Users/fengxing/picc/nmcp/nmcp_android/app/src/main/AndroidManifest.xml:106:9-116:20 Warning:
        provider#androidx.core.content.FileProvider@android:authorities was tagged at AndroidManifest.xml:106 to replace other declarations but no other declaration present
/Users/fengxing/picc/nmcp/nmcp_android/app/src/main/AndroidManifest.xml:112:13-115:52 Warning:
        meta-data#android.support.FILE_PROVIDER_PATHS@android:resource was tagged at AndroidManifest.xml:112 to replace other declarations but no other declaration present

> Task :app:compileDebugJavaWithJavac
...
> Task :camera:transformClassesWithObjectBoxAndroidTransformForDebug
...	
> Task :common:transformClassesWithObjectBoxAndroidTransformForDebug
...
> Task :app:transformNativeLibsWithStripDebugSymbolForDebug
...
...
...
BUILD SUCCESSFUL in 2m 33s
597 actionable tasks: 81 executed, 516 up-to-date
```

apk默认位置：/app/build/outputs/apk/

### 1.4 编译并打Debug包

`./gradlew assembleDebug`

```
➜  nmcp_android git:(pluginApk) ./gradlew assembleDebug
> Task :test:compileDebugJavaWithJavac
...
BUILD SUCCESSFUL in 2m 41s
270 actionable tasks: 73 executed, 197 up-to-date
```

- apk默认位置：/app/build/outputs/apk/debug/***.apk
- apk输入位置可以在gradle文件中修改
- 默认打出的是所有渠道的debug版本
- 可指定具体渠道

### 1.5 编译并打Release的包

`./gradlew assembleRelease`

```
➜  nmcp_android git:(pluginApk) ./gradlew assembleRelease
```

- apk默认位置：/app/build/outputs/apk/release/***.apk
- apk输入位置可以在gradle文件中修改

### 1.6 Release 模式打包并安装

`./gradlew installRelease`

### 1.7  Debug 模式打包并安装

`./gradlew installDebug`

### 1.8 卸载Release模式包

`./gradlew uninstallRelease`

## 构建基础

