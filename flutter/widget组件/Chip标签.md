---
title: Chip标签
date: 2020-07-17 21:24:56
tags:
	- flutter
categories: 
	- flutter
	- widget组件
---

> Flutter内置了多个标签类控件，但本质上它们都是同一个控件，只不过是属性参数不同而已，在学习的过程中可以将其放在放在一起学习，方便记忆。

使用场景：食物的属性或者标签，历史搜索等。

```
Chip({
    Key key,
    this.avatar,//标签左侧Widget，一般为小图标
    @required this.label,//标签
    this.labelStyle,
    this.labelPadding,//padding
    this.deleteIcon//删除图标,
    this.onDeleted//删除回调，为空时不显示删除图标,
    this.deleteIconColor//删除图标的颜色,
    this.deleteButtonTooltipMessage//删除按钮的tip文字,
    this.shape//形状,
    this.clipBehavior = Clip.none,
    this.backgroundColor//背景颜色,
    this.padding,
    this.materialTapTargetSize//删除图标material点击区域大小,
  })
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-688f0e81da2cb7ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## RawChip

Material风格标签控件，此控件是其他标签控件的基类，通常情况下，不会直接创建此控件，而是使用如下控件：

- Chip

- InputChip

- ChoiceChip

- FilterChip

- ActionChip

如果你想自定义标签类控件时通常使用此控件。

RawChip可以通过设置`onSelected`被选中，设置`onDeleted`被删除，也可以通过设置`onPressed`而像一个按钮，它有一个`label`属性，有一个前置（`avatar`）和后置图标（`deleteIcon`）。

基本用法如下：

```text
RawChip(
  label: Text('老孟'),
)
```

源码：

```
const RawChip({
    Key key,
    this.avatar,
    @required this.label,
    this.labelStyle,
    this.padding,
    this.labelPadding,
    Widget deleteIcon,
    this.onDeleted,
    this.deleteIconColor,
    this.deleteButtonTooltipMessage,
    this.onPressed,
    this.onSelected,
    this.pressElevation,
    this.tapEnabled = true,
    this.selected,
    this.showCheckmark = true,
    this.isEnabled = true,
    this.disabledColor,
    this.selectedColor,
    this.tooltip,
    this.shape,
    this.clipBehavior = Clip.none,
    this.backgroundColor,
    this.materialTapTargetSize,
    this.elevation,
    this.shadowColor,
    this.selectedShadowColor,
    this.avatarBorder = const CircleBorder(),
  }) : assert(label != null),
       assert(isEnabled != null),
       assert(clipBehavior != null),
       assert(pressElevation == null || pressElevation >= 0.0),
       assert(elevation == null || elevation >= 0.0),
       deleteIcon = deleteIcon ?? _kDefaultDeleteIcon,
       super(key: key);

```

## FilterChip

FilterChip可以作为过滤标签，本质上也是一个RawChip，用法如下：

```text
List<String> _filters = [];

Column(
  children: <Widget>[
    Wrap(
      spacing: 15,
      children: List.generate(10, (index) {
        return FilterChip(
          label: Text('老孟 $index'),
          selected: _filters.contains('$index'),
          onSelected: (v) {
            setState(() {
              if(v){
                _filters.add('$index');
              }else{
                _filters.removeWhere((f){
                  return f == '$index';
                });
              }
            });
          },
        );
      }).toList(),
    ),
    Text('选中：${_filters.join(',')}'),
  ],
)
```

![img](https://pic4.zhimg.com/v2-d162efa3acc2988e917d8c66847377bf_b.webp)

## ChoiceChip

允许从一组选项中进行单个选择，创建一个类似于单选按钮的标签，本质上ChoiceChip也是一个RawChip，ChoiceChip本身不具备单选属性。

单选demo如下：

```dart
int _selectIndex = 0;
Wrap(
  spacing: 15,
  children: List.generate(10, (index) {
    return ChoiceChip(
      label: Text('老孟 $index'),
      selected: _selectIndex == index,
      onSelected: (v) {
        setState(() {
          _selectIndex = index;
        });
      },
    );
  }).toList(),
)复制代码
```

效果如下：

![img](https://user-gold-cdn.xitu.io/2020/5/10/171fc2c459eefd0c?imageslim)

## InputChip

以紧凑的形式表示一条复杂的信息，例如实体（人，地方或事物）或对话文本。

InputChip 本质上也是RawChip，用法和RawChip一样。源代码如下：

```dart
@override
Widget build(BuildContext context) {
  assert(debugCheckHasMaterial(context));
  return RawChip(
    avatar: avatar,
    label: label,
    labelStyle: labelStyle,
    labelPadding: labelPadding,
    deleteIcon: deleteIcon,
    onDeleted: onDeleted,
    deleteIconColor: deleteIconColor,
    deleteButtonTooltipMessage: deleteButtonTooltipMessage,
    onSelected: onSelected,
    onPressed: onPressed,
    pressElevation: pressElevation,
    selected: selected,
    tapEnabled: true,
    disabledColor: disabledColor,
    selectedColor: selectedColor,
    tooltip: tooltip,
    shape: shape,
    clipBehavior: clipBehavior,
    focusNode: focusNode,
    autofocus: autofocus,
    backgroundColor: backgroundColor,
    padding: padding,
    materialTapTargetSize: materialTapTargetSize,
    elevation: elevation,
    shadowColor: shadowColor,
    selectedShadowColor: selectedShadowColor,
    showCheckmark: showCheckmark,
    checkmarkColor: checkmarkColor,
    isEnabled: isEnabled && (onSelected != null || onDeleted != null || onPressed != null),
    avatarBorder: avatarBorder,
  );
}
```

## ActionChip

显示与主要内容有关的一组动作，本质上也是一个RawChip，用法如下：

```dart
ActionChip(
    avatar: CircleAvatar(
      backgroundColor: Colors.grey.shade800,
      child: Text('孟'),
    ),
    label: Text('老孟'),
    onPressed: () {
      print("onPressed");
    })复制代码
```

效果如下：

![img](https://user-gold-cdn.xitu.io/2020/5/10/171fc2c4cf6d99c6?imageslim)

效果很像按钮类控件。

