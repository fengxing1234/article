---
title: 输入框TextField
date: 2020-07-14 22:11:36
tags:
	- flutter
categories: 
	- flutter
	- widget组件
---





```
const TextField({
  Key key,
  TextEditingController controller,            //输入框的控制器
  FocusNode focusNode,           //控制输入框是否获得焦点
  InputDecoration decoration: const InputDecoration(),    //设置输入框的外观
  TextInputType keyboardType,          //键盘类型
  TextInputAction textInputAction,         //回车按钮的类型
  TextCapitalization textCapitalization: TextCapitalization.none,    // 键盘大小写
  TextStyle style,             //文本样式
  StrutStyle strutStyle,
  TextAlign textAlign: TextAlign.start,     //水平方向对齐方式
  TextAlignVertical textAlignVertical,      //垂直方向对齐方式
  TextDirection textDirection,           //文本书写顺序
  bool readOnly: false,          //是否只读
  bool showCursor,        //是否显示光标
  bool autofocus: false,         //是否自动获得焦点
  bool obscureText: false,     //是否隐藏正在输入的文本，内容显示为圆点
  bool autocorrect: true,    //是否自动修正
  int maxLines: 1,        //输入框的最大行数
  int minLines,            //最小行数
  bool expands: false,    //是否填充父级
  int maxLength,             //文本最大长度
  bool maxLengthEnforced: true,          //超过文本最大长度是否限制输入
  ValueChanged<String> onChanged,      //输入文本时的回调
  VoidCallback onEditingComplete,          //完成输入时触发（按回车）
  ValueChanged<String> onSubmitted,        //完成输入时触发，接收当前输入文本为参数
  List<TextInputFormatter> inputFormatters,      //指定输入格式
  bool enabled,             //是否可用
  double cursorWidth: 2.0,            //光标宽度
  Radius cursorRadius,                //光标圆角
  Color cursorColor,                      //光标颜色
  Brightness keyboardAppearance,        //键盘色调，只在ios上有效
  EdgeInsets scrollPadding: const EdgeInsets.all(20.0),   //当输入框获得焦点并且有一部分被滚动出屏外时，将自动滚动。
  DragStartBehavior dragStartBehavior: DragStartBehavior.start,
  bool enableInteractiveSelection,      //是否允许选中
  GestureTapCallback onTap,            //点击回调
  InputCounterWidgetBuilder buildCounter,
  ScrollController scrollController,
  ScrollPhysics scrollPhysics
})
复制代码
```

先来一个最简单的demo

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
        home: Scaffold(
            body: Center(
              child: TextField(

              )
            )
        )
    );
  }
}
复制代码
```



```
return TextField(
      //控制器 清楚 默认值
      controller: _controller,
      focusNode: widget.currentFocusNode,
      autofocus: false,
//      maxLines: 10,
//      minLines: 4,
      expands: false,
      //maxLength: 4,
      keyboardType: TextInputType.text,
      //每次 输入/删除 都回调
      onChanged: (text) {
        print('$text');
      },
      //每次点击都回调
      onTap: () {
        print('onTop');
      },
      onEditingComplete: () {
        print('onEditingComplete');
        FocusScope.of(context).requestFocus(widget.nextFocusNode);
      },
      onSubmitted: (text) {
        print('onSubmitted = $text');
      },
      //只读模式
      //readOnly: true,
      //装饰器
      decoration: InputDecoration(
        //左侧 输入框前的图标
        icon: Icon(Icons.text_fields),
        //上方文字
        labelText: '请输入你的姓名)',
        //帮助类型文字提示在输入框下方，默认为灰色
        helperText: '请输入你的真实姓名',
        //为false时，当输入框获得焦点labelText不会缩小到左上角，而是消失
        hasFloatingPlaceholder: true,
        hintText: '这是一个提示信息这是一个提示信息这是一个提示信息这是一个提示信息这是一个提示信息',
        //错误提示，和helperText位置相同，默认为红色
        //errorText: '错误信息',
        //hintText最大行数，如果文本超出这个最大行数后边会显示为...
        hintMaxLines: 1,
        suffixIcon: Icon(Icons.chevron_right),
        //输入框左侧图标
        prefixIcon: Icon(Icons.skip_previous),
        //输入框内左侧图标
        contentPadding: EdgeInsets.zero,
        //作用是在较小空间时，使组件正常渲染（包括文本垂直居中）。
        isDense: true,
        //是否填充
        //filled: true,
        enabled: true,
        //填充颜色
        fillColor: Colours.Border_Color,
        focusColor: Colours.App_Main,
        hoverColor: Colours.App_FF_D7_D7,
      ),
      //键盘右下方的文字如：搜索、完成、下一项
      textInputAction: TextInputAction.next,
      //字母大写的选项 键盘必须是text的 否则无效
      textCapitalization: TextCapitalization.sentences,
      cursorWidth: 1,
      //光标圆角
      //cursorRadius: Radius.circular(16.0),
      //不显示光标
      showCursor: true,
      //是否密文，一般用于密码输入。
      obscureText: false,
      //是否自动修正输入的文本，默认为true。
      autocorrect: false,
      //BlacklistingTextInputFormatter：黑名单
      //WhitelistingTextInputFormatter：白名单
      inputFormatters: [
        DecimalPlacesFormatter(decimalRange: 2),
      ],
      //输入的文本属否允许选中，默认为true，为false时不能选中。
      enableInteractiveSelection: false,
    );
```



## 1、controller

输入框文本发生改变时触发，但不限于这个功能。
 先看demo

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
        home: Scaffold(
            body: TextFieldDemo()
        )
    );
  }
}

class TextFieldDemo extends StatefulWidget {
  @override
  _TextFieldState createState() => new _TextFieldState();
}

class _TextFieldState extends State<TextFieldDemo> {
  TextEditingController _controller = TextEditingController();

  @override
  void initState() {
    _controller.addListener(() {
      print(_controller.text);
    });
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: TextField(
        controller: _controller
      )
    );
  }
}
复制代码
```

和前几个组件类似，但不相同，不只是一个状态值，而是有一个controller，并且在initState里监听文本变化。
 是不是感觉和`onChange`重复了，继续往下看，除了监听文本变化，还有一些其他的功能。

### 设置默认值

initState方法修改如下

```
@override
void initState() {
  _controller.text = 'default value';      //输入框的默认值
  _controller.addListener(() {
    print(_controller.text);
  });
  super.initState();
}
复制代码
```

### 清空输入框

initState方法修改如下

```
@override
void initState() {
  _controller.text = 'default value';
  _controller.addListener(() {
    print(_controller.text);
    _controller.clear();
  });
  super.initState();
}
复制代码
```

当输入框获得焦点时，会触发`_controller`这个时候调用`clear`方法会清空输入框。

### selection

将initState方法修改如下：

```
void initState() {
  _controller.text = 'default value';
  _controller.selection = TextSelection(
    baseOffset: 3,
    extentOffset: 5,
    affinity: TextAffinity.upstream,
    isDirectional: true
  );
  _controller.addListener(() {
    print(_controller.text);
  });
  super.initState();
}
复制代码
```

`baseOffset`和`extentOffset`配对使用，demo中的写法为从文本中第四个和第五个字符为选中状态。
 另外的两个我不知道是干啥的。。。

## 2、focusNode

焦点控制

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

/// This Widget is the main application widget.
class MyApp extends StatelessWidget {
  static const String _title = 'Flutter Code Sample';

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: _title,
      home: Scaffold(
        appBar: AppBar(title: const Text(_title)),
        body: FocusNodeDemo(),
      ),
    );
  }
}

class FocusNodeDemo extends StatefulWidget {
  @override
  _FocusNodeState createState() => new _FocusNodeState();
}

class _FocusNodeState extends State<FocusNodeDemo> {
  FocusNode focusNode1 = new FocusNode();
  FocusNode focusNode2 = new FocusNode();
  FocusScopeNode focusScopeNode;

  @override
  Widget build(BuildContext context) {
    return Container (
      child: Column(
        children: <Widget>[
          TextField(
            //关联focusNode1
            focusNode: focusNode1,     
          ),
          TextField(
             //关联focusNode2
            focusNode: focusNode2,             
          ),
          RaisedButton(
            child: Text('移动焦点到1'),
            onPressed: () {
              //没有输入框获得焦点时初始值为null
              if (focusScopeNode == null) {
                focusScopeNode = FocusScope.of(context);
              }
              //focusNode1获得焦点
              focusScopeNode.requestFocus(focusNode1);
              //focusNode1和focusNode2焦点状态
              print(focusNode1.hasFocus);
              print(focusNode2.hasFocus);
            }
          ),
          RaisedButton(
            child: Text('移动焦点到2'),
            onPressed: () {
               //没有输入框获得焦点时初始值为null
              if (focusScopeNode == null) {
                focusScopeNode = FocusScope.of(context);
              }
              //focusNode2获得焦点
              focusScopeNode.requestFocus(focusNode2);
              //focusNode1和focusNode2焦点状态
              print(focusNode1.hasFocus);
              print(focusNode2.hasFocus);
            }
          ),
          RaisedButton(
            child: Text('隐藏键盘'),
            onPressed: () {
              //focusNode1和focusNode2失去焦点
              focusNode1.unfocus();
              focusNode2.unfocus();
            }
          ),
          RaisedButton(
            child: Text('焦点状态'),
            onPressed: () {
              //focusNode1和focusNode2焦点状态
              print(focusNode1.hasFocus);
              print(focusNode2.hasFocus);
            },
          )
        ]
      )
    );
  }
}
复制代码
```



![image.png](https://user-gold-cdn.xitu.io/2019/12/31/16f5c94807b9cc69?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



输入框的焦点通过`FocusNode`和`focusScopeNode`控制，`focusScopeNode`可以通过`focusScope.of`方法来获取初始值，这个方法不会让元素获得焦点，`focusScopeNode.requestFocus`方法可以让元素获得焦点。`focusNode`还有一个`unFocus`方法，可以让输入框失去焦点，另外`focusNode`还有一个`hasFocus`属性，输入框获得焦点时为`true`，失去焦点时为`false`。

## 3、decoration

先看一下constructor

```
const InputDecoration({
  Widget icon,         //输入框前的图标
  String labelText,    //与placeholder类似，当输入框获得焦点时会缩小到左上角
  TextStyle labelStyle,   //labelText的字体样式
  String helperText,    //帮助类型文字提示在输入框下方，默认为灰色
  TextStyle helperStyle,    //helperText的文字样式
  String hintText,      //相当于html里的placeholder，当设置了labelText时输入框未获得焦点时不显示，获得焦点之后labelText缩小后会显示
  TextStyle hintStyle,    //hintText文字样式
  int hintMaxLines,      //hintText最大行数，如果文本超出这个最大行数后边会显示为...
  String errorText,      //错误提示，和helperText位置相同，默认为红色
  TextStyle errorStyle,  //helperText的文字样式
  int errorMaxLines,    //errorText的最大行数，超出部分显示为...
  bool hasFloatingPlaceholder: true,  //为false时，当输入框获得焦点labelText不会缩小到左上角，而是消失
  bool isDense,
  EdgeInsetsGeometry contentPadding,    //输入框内边距
  Widget prefixIcon,        //输入框前缀图标
  Widget prefix,              //输入框前缀
  String prefixText,          //输入框前缀文本
  TextString prefixStyle,    //前缀样式
  Widget suffixIcon,        //后缀图标
  Widget suffix,              //后缀文本
  String suffixText,        //后缀
  TextStyle suffixStyle,    //后缀样式
  Widget counter,            //输入框右下方组件
  String counterText,        //输入框右下方文本
  TextStyle counterStyle,      //右下方文本样式
  bool filled,                  //是否填充
  Color fillColor,          //填充色（相当于背景色）
  Color focusColor,    //
  Color hoverColor,      //鼠标移入的填充色
  InputBorder errorBorder,      //错误时的边框
  InputBorder focusedBorder,    //获得焦点时边框
  InputBorder focusedErrorBorder,    //错误并获得焦点时的边框
  InputBorder disabledBorder,        //禁用时的边框
  InputBorder enabledBorder,        //可用时的边框
  InputBorder border,                //边框
  bool enabled: true,              //是否可用  
  String semanticCounterText,
  bool alignLabelWithHint
})
复制代码
```

这里的属性都很简单，没啥细说的而且有很多都是重复的，只不过是在不同状态下和不同位置。没注释的暂时还没整明白是干啥的。

## 4、keyboardType

输入框获得焦点时弹出键盘的类型。
 `number`、`phone`、`url`、`datetime`、`multiline`、`emailAddress`、`numberWithOptions()`、`visiblePassword`、`text`
 看着挺多，其实有好几个都是重复的，可能是为了语义化。

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: Scaffold(
        body: Center(
          child: TextField(
            keyboardType: TextInputType.number,
          )
        )
      )
    );
  }
}
复制代码
```



![image.png](https://user-gold-cdn.xitu.io/2019/12/31/16f5c9480c5df417?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 5、textInputAction

回车按钮的类型
 `continueAction`、`done`、`emergencyCall`、`go`、`join`、`newline`、`next`、`route`、`search`、`send`、`unspecified`、`none`、`previous`
 `unspecified`和`newline`是回车，`none`和`previous`不知道干啥，android和ios都报错不支持，其他的值是什么按钮就显示什么

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: Scaffold(
        body: Center(
          child: TextField(
            textInputAction: TextInputAction.send,
          )
        )
      )
    );
  }
}
复制代码
```



![image.png](https://user-gold-cdn.xitu.io/2019/12/31/16f5c9480caa54c7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 6、textCapitalization

配置平台键盘如何选择大写或小写键盘，只有文本键盘有效。
 `none`：默认键盘
 `characters`：大写键盘
 `sentences`：第一个字母大写
 `words`：单词首字母大写

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: Scaffold(
        body: Center(
          child: TextField(
            textCapitalization: TextCapitalization.words
          )
        )
      )
    );
  }
}
复制代码
```



![image.png](https://user-gold-cdn.xitu.io/2019/12/31/16f5c9480cc81b7b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 7、style

文本样式，这里不说了，之前的笔记有介绍。

## 8、strutStyle

这个也不说了，看之前的笔记[flutter笔记（二）-----hello world和文本组件Text、TextSpan](https://juejin.im/post/5e096bc5e51d45583d427396)

## 9、textAlign

对齐方式[flutter笔记（二）-----hello world和文本组件Text、TextSpan](https://juejin.im/post/5e096bc5e51d45583d427396)

## 10、TextAlignVertical

垂直方向的对齐方式，有三个值
 `top`、`center`、`bottom`
 和css的`vertical-align`一个作用

## 11、textDirection

文本的书写顺序，不详细介绍了看之前的笔记[flutter笔记（二）-----hello world和文本组件Text、TextSpan](https://juejin.im/post/5e096bc5e51d45583d427396)

## 12、readOnly

是否只读，值为`boolean`类型，为`true`时只读，但是输入框可获得焦点。

## 13、showCursor

是否显示光标，值为`boolean`类型。

## 14、autofocus

是否自动获得焦点。

## 15、obscureText

是否密文，一般用于密码输入。

## 16、autocorrect

是否自动修正输入的文本，默认为`true`。

## 17、maxLines

输入框的最当行数，默认为1，为`null`时不做限制。

## 18、minLines

输入框最小行数，和`maxLines`同时使用，并且不能大于`maxLines`。

## 19、expands

是否调整输入框高度，以填充父级，`maxLines`和`minLines`必须为null。

## 20、maxLength

文本最大长度，输入框右下角有长度显示。

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: Scaffold(
        body: Center(
          child: TextField(
            maxLength: 20
          )
        )
      )
    );
  }
}
复制代码
```



![image.png](https://user-gold-cdn.xitu.io/2019/12/31/16f5c94817366309?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 21、maxLengthEnforced

文本长度超过最大长度是否限制输入，为false时不禁止输入，但是输入框会变红。

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: Scaffold(
        body: Center(
          child: TextField(
            maxLength: 5,
            maxLengthEnforced: false
          )
        )
      )
    );
  }
}
复制代码
```



![image.png](https://user-gold-cdn.xitu.io/2019/12/31/16f5c9484cc4ce2a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 22、onChanged

文本输入回调

## 23、onEditingComplete

文本输入完成时的回调，即按回车时的回调。

## 24、onSubmitted

文本输入完成的回调，当前输入的文本为参数。

## 25、inputFormatters

指定输入文本的格式，需要引入`package:flutter/services.dart`，值是`List`类型，可以有多个配置。
 `BlacklistingTextInputFormatter`：黑名单
 `WhitelistingTextInputFormatter`：白名单

```
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: Scaffold(
        body: Center(
          child: TextField(
            inputFormatters: [
              BlacklistingTextInputFormatter(RegExp("[abc]"))
            ],
          )
        )
      )
    );
  }
}
复制代码
```

将`abc`添加黑名单，禁止输入。

## 26、enabled

是否可用，为`false`时不可用，输入框不能获得焦点。

## 27、cursorWidth

光标宽度，默认是2.0。

## 28、cursorRadius

光标圆角。

## 29、cursorColor

光标颜色

## 30、keyboardAppearance

键盘色调，`Brightness.light`为亮色，`Brightness.dark`为暗，只在`ios`上有效。

## 31、scrollPadding

当输入框获得焦点，并有一部分滚动到可视区之外时，自动将输入框滚动到可视区之内。

## 32、dragStartBehavior

好像和拖动有关，还没整明白怎么用。

## 33、enableInteractiveSelection

输入的文本属否允许选中，默认为`true`，为`false`时不能选中。

## 34、onTap

点击的回调。

## 35、buildCounter

没看明白怎么用。

## 36、scrollController & scrollPhysics

这两个放一起是因为不知道怎么用，看文档`scrollController`是创建一个滚动控制器，`scrollPhysics`是对滚动的物理特性做处理，应该是可以控制滚动惯性、滚动回弹的东西。