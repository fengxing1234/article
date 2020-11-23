---
title: ListView
date: 2020-06-24 13:13:57
tags:
	- flutter
categories: 
	- flutter
	- widget组件
---

## ListView常用构造

### ListView

我们可以直接使用**ListView** 它的实现也是直接返回最简单的列表结构，粗糙没有修饰。

#### ListView 默认构建

```
 ///默认构建
  Widget listViewDefault(List<BaseBean> list) {
    List<Widget> _list = new List();
    for (int i = 0; i < list.length; i++) {
      _list.add(new Center(
        child: new Text(list[i].age.toString()),
      ));
    }

// 添加分割线
    var divideList =
        ListTile.divideTiles(context: context, tiles: _list).toList();
    return new Scrollbar(
      child: new ListView(
        // 添加ListView控件
        children: _list, // 无分割线
//        children: divideList, // 添加分割线/
      ),
    );
  }
```

### ListView ListTile

**ListTile** 是**Flutter** 给我们准备好的**widget** 提供非常常见的构造和定义方式，包括文字，**icon**，点击事件，一般是能够满足基本需求，但是就不能自己定义了

#### ListTile 属性

```
this.leading,              // item 前置图标
this.title,                // item 标题
this.subtitle,             // item 副标题
this.trailing,             // item 后置图标
this.isThreeLine = false,  // item 是否三行显示
this.dense,                // item 直观感受是整体大小
this.contentPadding,       // item 内容内边距
this.enabled = true,
this.onTap,                // item onTap 点击事件
this.onLongPress,          // item onLongPress 长按事件
this.selected = false,     // item 是否选中状态
```

##### ListTile 使用

```
  /// ListTile代码
  Widget listViewListTile(List<BaseBean> list) {
    List<Widget> _list = new List();
    for (int i = 0; i < list.length; i++) {
      _list.add(new Center(
        child: ListTile(
          leading: new Icon(Icons.list),
          title: new Text(list[i].name),
          trailing: new Icon(Icons.keyboard_arrow_right),
        ),
      ));
    }
    return new ListView(
      children: _list,
    );
  }
```

### ListView builder

**builder** 顾名思义 构造 可以非常方便的构建我们自己定义的**child**布局，所以在**Flutter**中非常的常用。

#### builder属性详细介绍

```
   //设置滑动方向 Axis.horizontal 水平  默认 Axis.vertical 垂直
        scrollDirection: Axis.vertical,
        //内间距
        padding: EdgeInsets.all(10.0),
        //是否倒序显示 默认正序 false  倒序true
        reverse: false,
        //false，如果内容不足，则用户无法滚动 而如果[primary]为true，它们总是可以尝试滚动。
        primary: true,
        //确定每一个item的高度 会让item加载更加高效
        itemExtent: 50.0,
        //内容适配
        shrinkWrap: true,
        //item 数量
        itemCount: list.length,
        //滑动类型设置
        physics: new ClampingScrollPhysics(),
         //cacheExtent  设置预加载的区域 
         cacheExtent: 30.0, 
        //滑动监听
//        controller ,
```

#### 分析几个比较难理解的属性

**shrinkWrap**特别推荐
**child** 高度会适配 **item**填充的内容的高度,我们非常的不希望**child**的高度固定，因为这样的话，如果里面的内容超出就会造成布局的溢出。
**shrinkWrap**多用于嵌套**listView**中 内容大小不确定 比如 垂直布局中 先后放入文字 **listView** （需要**Expend**包裹否则无法显示无穷大高度 但是需要确定**listview**高度 **shrinkWrap**使用内容适配不会有这样的影响）

**primary**
**If the [primary] argument is true, the [controller] must be null.**
在构造中默认是false 它的意思就是为主的意思，**primary**为true时，我们的**controller** 滑动监听就不能使用了

**physics**
这个属性几个滑动的选择
**AlwaysScrollableScrollPhysics**() 总是可以滑动
**NeverScrollableScrollPhysics**禁止滚动
**BouncingScrollPhysics** 内容超过一屏 上拉有回弹效果
**ClampingScrollPhysics** 包裹内容 不会有回弹

**cacheExtent**
这个属性的意思就是预加载的区域
设置预加载的区域 **cacheExtent** 强制设置为了 0.0，从而关闭了“预加载”

**controller**
滑动监听，我们多用于上拉加载更多，通过监听滑动的距离来执行操作。



```
 ///listView builder 构建
  Widget listViewLayoutBuilder(List<BaseBean> list) {
    return ListView.builder(
        scrollDirection: Axis.vertical,   
        padding: EdgeInsets.all(10.0),
        reverse: false,
        primary: true,
        itemExtent: 50.0,
        shrinkWrap: true,
        itemCount: list.length,     
        cacheExtent: 30.0, 
        physics: new ClampingScrollPhysics(),
//        controller ,
        itemBuilder: (context, i) => new Container(
              child: new Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: <Widget>[
                  new Text(
                    "${list[i].name}",
                    style: new TextStyle(fontSize: 18.0, color: Colors.red),
                  ),
                  new Text(
                    "${list[i].age}",
                    style: new TextStyle(fontSize: 18.0, color: Colors.green),
                  ),
                  new Text(
                    "${list[i].content}",
                    style: new TextStyle(fontSize: 18.0, color: Colors.blue),
                  ),
                ],
              ),
            ));
  }
```

##### builder模式来设置分割线

我们在正常的需求中大部分是需要**item**的分割线的，而在**builder**模式中使用**divide** 会有种情况（**divide**放在**item**的布局中 通过**Column**），我们会发现**divide**并没有直接延时到**item**两端而是会有左右**padding**。
所以我们可以通过另外一种方式去实现。

```
1.扩大list容积 为什么是两倍，因为我们给了divide的index
 Widget listView = new ListView.builder(
              itemCount: list.length * 2 ,
              itemBuilder: (context, index) => itemDividerRow(context, index));
2. 根据下标分配item类型
itemDividerRow(context, int i) {
if (i.isOdd) {//是奇数
        return new Divider( //返回分割线
          height: 1.0,
        );
      } else {
        i = i ~/ 2;
        return getRowWidget(context, orderList[i]);  //返回item 布局
      }
     } 
     
这样我们就可以去实现了，builder模式分割线 
```

### ListView separated

**separated** 有分离的意思，其实它就相当于我们**Android**中的多类型**adapter**，那么关键就是在我们的这个属性上**separatorBuilder**

#### separatorBuilder

它和**itemBuilder**同时进行渲染，在同一个**item**下标中可以额外的修饰或者区分

```
  separatorBuilder: (content, index) 
  
  itemBuilder: (content, index) 
```

##### separated设置分割线

```
separated设置分割线就非常的简单了，我们直接在separatorBuilder进行操作
 separatorBuilder: (content, index) {
   
        return new Divider(）
        ｝
```

```
///  listView separated 构建 用于多类型 分割
Widget listViewLayoutSeparated(List<BaseBean> list) {
  return ListView.separated(
    itemCount: list.length,
    separatorBuilder: (content, index) {
      //和itemBuilder 同级别的执行
      if (index == 2) {
        return new Container(
          height: 40.0,
          child: new Center(
            child: new Text("类型1"),
          ),
          color: Colors.red,
        );
      } else if (index == 7) {
        return new Container(
          height: 40.0,
          child: new Center(
            child: new Text("类型2"),
          ),
          color: Colors.blue,
        );
      } else if (index == 14) {
        return new Container(
          height: 40.0,
          child: new Center(
            child: new Text("类型3"),
          ),
          color: Colors.yellow,
        );
      } else {
        return new Container();
      }
    },
    itemBuilder: (content, i) {
      return new InkWell(
        child: new Container(
            height: 30.0,
            child: new Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <Widget>[
                new Text(
                  "${list[i].name}",
                  style: new TextStyle(fontSize: 18.0, color: Colors.red),
                ),
                new Text(
                  "${list[i].age}",
                  style: new TextStyle(fontSize: 18.0, color: Colors.green),
                ),
                new Text(
                  "${list[i].content}",
                  style: new TextStyle(fontSize: 18.0, color: Colors.blue),
                ),
              ],
            )),
        onTap: () {
          print("1111");
        },
      );
//      return ;
    },
  );
}
```

### ListView custom

大家可能对前两种比较熟悉，分别是传入一个子元素列表或是传入一个根据索引创建子元素的函数。
其实前两种方式都是第三种方式的“快捷方式”。因为 **ListView** 内部是靠这个 **childrenDelegate** 属性动态初始化子元素的。
我们使用**builder**和**separated**比较多，这个**custom**相对来说就比较少了。但是我们是需要了解的。

```
  ///listView custom 构建
  Widget listViewLayoutCustom(list) {
//    return ListView.custom(childrenDelegate: new MyChildrenDelegate());
    return ListView.custom(
      itemExtent: 40.0,
      childrenDelegate: MyChildrenDelegate(
        (BuildContext context, int i) {
          return new Container(
              child: new Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: <Widget>[
              new Text(
                "${list[i].name}",
                style: new TextStyle(fontSize: 18.0, color: Colors.red),
              ),
              new Text(
                "${list[i].age}",
                style: new TextStyle(fontSize: 18.0, color: Colors.green),
              ),
              new Text(
                "${list[i].content}",
                style: new TextStyle(fontSize: 18.0, color: Colors.blue),
              ),
            ],
          ));
        },
        childCount: list.length,
      ),
      cacheExtent: 0.0,
    );
  }
}
```

##### childrenDelegate

自定义childrenDelegate 当然我们可以对ListView中的child进行自己需要的操作。最难定义的也就是这个而已。

```
// ignore: slash_for_doc_comments
/**
 * 继承SliverChildBuilderDelegate  可以对列表的监听
 */
class MyChildrenDelegate extends SliverChildBuilderDelegate {
  MyChildrenDelegate(
    Widget Function(BuildContext, int) builder, {
    int childCount,
    bool addAutomaticKeepAlive = true,
    bool addRepaintBoundaries = true,
  }) : super(builder,
            childCount: childCount,
            addAutomaticKeepAlives: addAutomaticKeepAlive,
            addRepaintBoundaries: addRepaintBoundaries);

  ///监听 在可见的列表中 显示的第一个位置和最后一个位置
  @override
  void didFinishLayout(int firstIndex, int lastIndex) {
    print('firstIndex: $firstIndex, lastIndex: $lastIndex');
  }

  ///可不重写 重写不能为null  默认是true  添加进来的实例与之前的实例是否相同 相同返回true 反之false
  ///listView 暂时没有看到应用场景 源码中使用在 SliverFillViewport 中
  @override
  bool shouldRebuild(SliverChildBuilderDelegate oldDelegate) {
    // TODO: implement shouldRebuild
    print("oldDelegate$oldDelegate");
    return super.shouldRebuild(oldDelegate);
  }
}
```