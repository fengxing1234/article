---
title: adb常用命令
date: 2020-05-21 21:21:56
tags:
---

# ADB

[ADB命令大全](https://www.wanandroid.com/blog/show/2310)

[Android官方](https://developer.android.com/studio/command-line/adb.html#test_harness)

[一份超全超详细的 ADB 用法大全](https://github.com/mzlogin/awesome-adb)

## ADB简介

ADB，即 Android Debug Bridge，它是 Android 开发/测试人员不可替代的强大工具，也是 Android 设备玩家的好玩具。安卓调试桥 (Android Debug Bridge, adb)，是一种可以用来操作手机设备或模拟器的命令行工具。它存在于 sdk/platform-tools 目录下。虽然现在 Android Studio 已经将大部分 ADB 命令以图形化的形式实现了，但是了解一下还是有必要的。

## 如何使用

**启动ADB**的2种方式：

- 直接进入**sdk/platform-tools**目录：然后在命令行中输入**adb devices**来验证设备是否连接。
  - 缺点：每次进入platform-tools目录很麻烦。
- 将adb地址写入环境变量（即配置adb为环境变量）

## 常用命令

### 查看当前连接设备：

- 查看当前连接设备：

```
adb devices
```

- 如果发现多个设备：

```adb -s 设备号 其他指令
adb -s 设备号 其他指令
```

### 事件输入

1. 使用**adb shell input**命令向屏幕输入一些信息，

```
adb shell input text "insert%stext%shere"
```

**注意：%s表示空格。**

2. 使用**adb shell input tap**命令来模拟屏幕点击事件，例如：

```
adb shell input tap 500 1450
```

表示在屏幕上（500，1450）的坐标点上进行一次点击。 

3. 使用**adb shell input swipe**命令来模拟手势滑动事件，例如：

```
adb shell input swipe 100 500 100 1450 100
```

表示从屏幕坐标（100，500）开始，滑动到(100,1450)结束，整个过程耗时100ms.

4. 使用上面的命令还可以模拟”**长按（long press）**操作，也就是2个坐标点相同，耗时超过500ms.

```
adb shell input swipe 100 500 100 500 500
```

5. 使用**adb shell input keyevent**命令来模拟点按实体按钮的命令，例如：

```
adb shell input keyevent 25
```

该命令表示调低音量。数字25是在AOSP源码中的**KeyEvent类**里定义的一个事件常量。该类定义了将近300个事件常量。

### 查看顶部Activity

- windows环境下:

```
adb shell dumpsys activity | findstr "mFocusedActivity"

```

- Linux、Mac环境下：

```
adb shell dumpsys activity | grep "mFocusedActivity"
```

### 查看日志：

```
adb logcat
```

### 安装apk文件：
```
adb install xxx.apk
```

- 此安装方式，如果已经存在，无法安装；
  推荐使用**覆盖安装：**

```
adb install -r xxx.apk
```

- 比分直接RUN出来的包是test-onlu的无法安装，推荐使用**-t**

```
adb install -r -t xxx.apk
```

### 卸载App:

```
adb uninstall com.zhy.app
```

- 如果想要保留数据，则：

```
adb uninstall -k com.zhy.app
```

### 传递文件：

- 电脑往手机SDCard传递文件：

```
adb push 文件名 手机端SDCard路径
```

例如：

```
adb push 帅照.jpg /sdcard/
```

- 从手机端拉取文件到电脑：

```
adb pull /sdcard/xxx.txt
```

### 查看手机端安装的所有app包名:

```
adb shell pm list packages
```

### 启动Activity:

```
adb shell am start 包名/完整Activity路径
```

例如：

```
例如：
adb shell am start com.zhyen.android/com.zhyen.android.MainActivity
```

如果需要携带参数(携带一个Intent,Key 为name):

```
adb shell am start com.zhyen.android/com.zhyen.android.MainActivity -e name zhy
```

启动一个隐式的Intent:

```
adb shell am start -a "android.intent.action,VIEW" -d "https://www.google.com"
```

### 发送广播：

```
adb shell am broadcast -a "broadcastactionfilter"
```

- 如果需要携带参数（携带一个Intent,key为name）:

```
adb shell am broadcast -a "broadcastactionfilter" -e name zhy
```

### 启动服务：

```
adb shell am startservice "com.zhy.aaa/com.zhy.aaa.MyService"
```

### 屏幕截图：

- 可以使用screencap命令来进行手机屏幕截图，例如：

```
adb shell screencap /sdcard/screen.png
```

### 录制视频：

可以使用screenrecord[options] filename命令来录制屏幕视频，例如：

```
adb shell screenrecord /sdcard/demo.mp4
```

## 无线ADB

> 为避免使用数据线，可通过wifi通信，前提是**手机与PC处于同一局域网**

**启动方法**：

```
adb tcpip 5555  //这一步，必须通过数据线把手机与PC连接后再执行
adb connect <手机IP>
```

**停止方法**：

```
adb disconnect //断开wifi连接
adb usb //切换到usb模式
```

## Keycode表

```
0 -->  "KEYCODE_UNKNOWN"
1 -->  "KEYCODE_MENU"
2 -->  "KEYCODE_SOFT_RIGHT"
3 -->  "KEYCODE_HOME"     //Home键
4 -->  "KEYCODE_BACK"     //返回键
5 -->  "KEYCODE_CALL"
6 -->  "KEYCODE_ENDCALL"
7 -->  "KEYCODE_0"       //数字键0
8 -->  "KEYCODE_1"
9 -->  "KEYCODE_2"
10 -->  "KEYCODE_3"
11 -->  "KEYCODE_4"
12 -->  "KEYCODE_5"
13 -->  "KEYCODE_6"
14 -->  "KEYCODE_7"
15 -->  "KEYCODE_8"
16 -->  "KEYCODE_9"
17 -->  "KEYCODE_STAR"
18 -->  "KEYCODE_POUND"
19 -->  "KEYCODE_DPAD_UP"
20 -->  "KEYCODE_DPAD_DOWN"
21 -->  "KEYCODE_DPAD_LEFT"
22 -->  "KEYCODE_DPAD_RIGHT"
23 -->  "KEYCODE_DPAD_CENTER"
24 -->  "KEYCODE_VOLUME_UP"    //音量键+
25 -->  "KEYCODE_VOLUME_DOWN"  //音量键-
26 -->  "KEYCODE_POWER"   //Power键
27 -->  "KEYCODE_CAMERA"
28 -->  "KEYCODE_CLEAR"
29 -->  "KEYCODE_A"    //字母键A
30 -->  "KEYCODE_B"
31 -->  "KEYCODE_C"
32 -->  "KEYCODE_D"
33 -->  "KEYCODE_E"
34 -->  "KEYCODE_F"
35 -->  "KEYCODE_G"
36 -->  "KEYCODE_H"
37 -->  "KEYCODE_I"
38 -->  "KEYCODE_J"
39 -->  "KEYCODE_K"
40 -->  "KEYCODE_L"
41 -->  "KEYCODE_M"
42 -->  "KEYCODE_N"
43 -->  "KEYCODE_O"
44 -->  "KEYCODE_P"
45 -->  "KEYCODE_Q"
46 -->  "KEYCODE_R"
47 -->  "KEYCODE_S"
48 -->  "KEYCODE_T"
49 -->  "KEYCODE_U"
50 -->  "KEYCODE_V"
51 -->  "KEYCODE_W"
52 -->  "KEYCODE_X"
53 -->  "KEYCODE_Y"
54 -->  "KEYCODE_Z"
55 -->  "KEYCODE_COMMA"
56 -->  "KEYCODE_PERIOD"
57 -->  "KEYCODE_ALT_LEFT"
58 -->  "KEYCODE_ALT_RIGHT"
59 -->  "KEYCODE_SHIFT_LEFT"
60 -->  "KEYCODE_SHIFT_RIGHT"
61 ->  "KEYCODE_TAB"
62 -->  "KEYCODE_SPACE"
63 -->  "KEYCODE_SYM"
64 -->  "KEYCODE_EXPLORER"
65 -->  "KEYCODE_ENVELOPE"
66 -->  "KEYCODE_ENTER"  //回车键
67 -->  "KEYCODE_DEL"
68 -->  "KEYCODE_GRAVE"
69 -->  "KEYCODE_MINUS"
70 -->  "KEYCODE_EQUALS"
71 -->  "KEYCODE_LEFT_BRACKET"
72 -->  "KEYCODE_RIGHT_BRACKET"
73 -->  "KEYCODE_BACKSLASH"
74 -->  "KEYCODE_SEMICOLON"
75 -->  "KEYCODE_APOSTROPHE"
76 -->  "KEYCODE_SLASH"
77 -->  "KEYCODE_AT"
78 -->  "KEYCODE_NUM"
79 -->  "KEYCODE_HEADSETHOOK"
80 -->  "KEYCODE_FOCUS"
81 -->  "KEYCODE_PLUS"
82 -->  "KEYCODE_MENU"
83 -->  "KEYCODE_NOTIFICATION"
84 -->  "KEYCODE_SEARCH"
```

------