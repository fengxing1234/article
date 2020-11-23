---
title: Android-Gradle-DSL
date: 2020-09-08 16:22:26
tags:
---

### 构建过程

![aaa](https://upload-images.jianshu.io/upload_images/4118241-4e4251782c38a8e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中绿色标注为其中用到的相应工具，蓝色代表的是中间生成的各类文件类型。

- 首先aapt(Android Asset Packaging Tool)工具会将资源文件(如AndroidManifest.xml还有其他Activity的XML文件)编译成二进制文件，生成对应资源ID的R文件和资源文件。
- adil工具会将其中的aidl接口转化成Java的接口
- 至此，Java Compiler开始进行Java文件向class文件的转化，将R文件，Java源代码，由aidl转化来的Java接口，统一转化成.class文件。
- 用dx将.class文件编译成Dalvik虚拟机使用的字符码
- 此时我们得到了经过处理后的资源文件和一个dex文件，当然，还会存在一些其它的资源文件，这个时候，就是将其打包成一个类似apk的文件。但还并不是直接可以安装在Android系统上的APK文件。
- 通过签名工具对其进行签名。
- 通过Zipalign进行优化，提升运行速度。
- 最终，一个可以安装在我们手机上的APK了。

### Android项目构建过程的详细步骤图。

![详细构建过程](https://user-gold-cdn.xitu.io/2018/1/25/1612d1948620cb96?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

