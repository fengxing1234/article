---
title: Flutter快捷开发MacAndroidStudio篇
date: 2020-07-21 23:32:58
tags:
	- Flutter
categories: 
	- Flutter
	- Flutter实战
---

## 一、快捷命令

### 1.1 快速创建一个新的Stateless or Stateful组件

#### 创建新的 Stateless 组件，输入stless，回车：

![img](https://user-gold-cdn.xitu.io/2020/7/16/17354caffe6cf9ab?imageslim)

#### 创建新的 Stateful 组件，输入 stful，回车：

![img](https://user-gold-cdn.xitu.io/2020/7/16/17354cb01fa2b87b?imageslim)

### 1.2 创建新的 动画组件，输入 stanim，回车：

![img](https://user-gold-cdn.xitu.io/2020/7/16/17354cb08e651fd9?imageslim)

### 1.3其它

还有其他的一些快捷方式，这里不一一介绍，这些快捷方式在 **Preferences** 中可以找到，路径：Preferences -> Editor -> Live Templates:

![image.png](https://upload-images.jianshu.io/upload_images/4118241-e268441840514553.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 二、File And Code Templates

创建 Dart 文件时，生成默认代码，打开 Preferences -> Editor -> File And Code Templates，选中右侧的 Files 标签，默认里面是空的

加入自动生成代码：

```
import 'package:flutter/material.dart';

/// 项目：${PROJECT_NAME}
/// 包层次结构：${PACKAGE_NAME}
/// 文件： ${NAME}
/// 创建者：${USER}
/// 创建时间：${DATE} ${TIME}
/// 文件描述:
class ${NAME} extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}

```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-efcaba5ef79a90a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

新创建的 dart 文件自动生成了预置的代码。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-73c8f0bbd40f16ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 三、格式化代码

快捷键：option + command + L（字母 L 键）

![img](https://user-gold-cdn.xitu.io/2020/7/16/17354cb199a0b208?imageslim)

## 四、清除无用引用

快捷键：control + option + O（字母 O 键）

![img](https://user-gold-cdn.xitu.io/2020/7/16/17354cb1b4728b5e?imageslim)

## 五、前进/后退光标的位置

当跟踪代码的时候，经常跳转到其他类，后退快捷键：option+command+方向左键，前进快捷键：option+command+方向右键，

![img](https://user-gold-cdn.xitu.io/2020/7/16/17354cb1e7c47387?imageslim)



## 六、查看当前类的继承关系

快捷键： control + H

![img](https://user-gold-cdn.xitu.io/2020/7/16/17354cb28acd7668?imageslim)

## 七、自动定位

右侧进入一个代码文件时，左侧自定定位到此文件，在 project 标签 设置中勾选 Autoscroll to source 和 Autoscroll from source。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-267dee58ec7a2320.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Flutter 资源网站

1. 官网：[flutter.dev/](https://flutter.dev/)
2. 中文网：[flutterchina.club/](https://flutterchina.club/)
3. Flutter 中文社区资源：[flutter-io.cn/](https://flutter-io.cn/)
4. pub(国内)：[pub.flutter-io.cn/](https://pub.flutter-io.cn/)
5. pub：[pub.dev/](https://pub.dev/)
6. DartPad：[dartpad.dartlang.org/](https://dartpad.dartlang.org/)
7. Dart 官网：[dart.dev/](https://dart.dev/)
8. CodePen：[codepen.io/](https://codepen.io/)
9. Json 转实体类：[javiercbk.github.io/json_to_dar…](https://javiercbk.github.io/json_to_dart/)