---
title: Scaffold
date: 2020-07-01 15:51:09
tags:
	- flutter
categories: 
	- flutter
	- widget组件
---



```
//脚手架
Scaffold({
    Key key,
    this.appBar,//设置应用栏，显示在脚手架顶部
    this.body,//设置脚手架内容区域控件
    this.floatingActionButton,//设置显示在上层区域的按钮，默认位置位于右下角
    this.floatingActionButtonLocation,//设置floatingActionButton的位置
    this.floatingActionButtonAnimator,//floatingActionButtonAnimator 动画 动画，但是设置了没有效果？
    this.persistentFooterButtons,//一组显示在脚手架底部的按钮(在bottomNavigationBar底部导航栏的上面)
    this.drawer,//设置左边侧边栏
    this.endDrawer,//设置右边侧边栏
    this.bottomNavigationBar,//设置脚手架 底部导航栏
    this.bottomSheet,//底部抽屉栏
    this.backgroundColor,//设置脚手架内容区域的颜色
    this.resizeToAvoidBottomPadding = true,// ? 控制界面内容 body 是否重新布局来避免底部被覆盖，比如当键盘显示的时候，重新布局避免被键盘盖住内容。
    this.primary = true,//脚手架是否显示在最低部
  })
```

