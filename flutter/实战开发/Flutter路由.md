---
title: Flutter路由
date: 2020-06-21 16:05:42
tags:
	- Flutter
	- Flutter路由
	- Flutter实战
categories: 
	- Flutter
	- Flutter实战
---

# 路由管理控制

- 路由是一个应用程序抽象的屏幕或页面；
- 路由管理就是管理页面之间如何跳转；
- 路由入栈指打开一个新页面；
- 路由出栈指一个页面关闭操作；
- 路由管理指如何来管理路由栈；
- `Navigator`是一个管理路由的`widget`；
- `NavigatorKey`是一个管理路由的`Key`；

# Navigator

| 方法                    | 作用                                                         |
| ----------------------- | ------------------------------------------------------------ |
| pushNamed               | 按路由名字路由入栈                                           |
| pushReplacementNamed    | 按路由名字替换当前路由栈                                     |
| popAndPushNamed         | 将当前路线从导航器中弹出，并在其中推入已命名的路由位置       |
| pushNamedAndRemoveUntil | 按路由名称将具有给定名称的路由推入导航器，然后删除所有       |
| push                    | 直接路由入栈                                                 |
| pushReplacement         | 替换当前路由栈                                               |
| pushAndRemoveUntil      | 将具有给定名称的路由推入导航器，然后删除所有                 |
| replace                 | 用新路由替换导航器上的路由                                   |
| replaceRouteBelow       | 用新路由替换导航器上的路由。 路由是替换为给定`anchorRoute`下面的那个 |
| canPop                  | 导航器是否可以弹出。                                         |
| maybePop                | 导航器是否可以弹出，可以的话弹出                             |
| pop                     | 弹出路由                                                     |
| popUntil                | 一直弹出直到指定路由                                         |
| removeRoute             | 删除指定路由                                                 |
| removeRouteBelow        | 立即从导航器中删除一条路由，然后[Route.dispose]的要替换的路线是给定的“ anchorRoute”下方的路线。 |

