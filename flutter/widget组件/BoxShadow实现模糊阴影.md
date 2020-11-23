---
title: BoxShadow实现模糊阴影
date: 2020-07-02 21:12:37
tags:
	- flutter
categories: 
	- flutter
	- widget组件
---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190108104904699.png)

### 第一个圆形button

```
Widget _circleButton() {
  return Container(
    margin: EdgeInsets.only(left: 20),
    decoration: BoxDecoration(shape: BoxShape.circle, boxShadow: [
      ///阴影颜色/位置/大小等
      BoxShadow(color: Colors.grey[300],offset: Offset(1, 1),
        ///模糊阴影半径
        blurRadius: 5,
      ),
      BoxShadow(color: Colors.grey[300], offset: Offset(-1, -1), blurRadius: 5),
      BoxShadow(color: Colors.grey[300], offset: Offset(1, -1), blurRadius: 5),
      BoxShadow(color: Colors.grey[300], offset: Offset(-1, 1), blurRadius: 5)
    ]),
    child: CircleAvatar(
      radius: 20,
      backgroundColor: Colors.white,
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Icon(
            Icons.launch,
            color: Colors.black87,
            size: 20,
          ),
          Text(
            "分享",
            style: TextStyle(
              color: Colors.grey[400],
              fontSize: 10,
            ),
          )
        ],
      ),
    ),
  );
}

```

### 第二个图片

```
Widget _rectangleButton() {
  return Container(
    margin: EdgeInsets.only(top: 20, left: 20),
    decoration: BoxDecoration(shape: BoxShape.rectangle, boxShadow: [
      BoxShadow(color: Colors.grey[300],offset: Offset(1, 1),blurRadius: 5,),
      BoxShadow(color: Colors.grey[300], offset: Offset(-1, -1), blurRadius: 5),
      BoxShadow(color: Colors.grey[300], offset: Offset(1, -1), blurRadius: 5),
      BoxShadow(color: Colors.grey[300], offset: Offset(-1, 1), blurRadius: 5)
    ]),
    child: FlatButton(
        onPressed: () {},
        color: Colors.white,
        padding: EdgeInsets.only(left: 30, right: 30, top: 10, bottom: 10),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Icon(
              Icons.launch,
              color: Colors.black87,
              size: 20,
            ),
            Text(
              "分享",
              style: TextStyle(
                color: Colors.grey[400],
                fontSize: 10,
              ),
            )
          ],
        )),
  );
}

```

### 第三个图片

```
Widget _flatButton1() {
  return Container(
    margin: EdgeInsets.only(top: 20, left: 20),
    decoration: BoxDecoration(shape: BoxShape.rectangle, boxShadow: [
      //BoxShadow中Offset (1,1)右下，(-1,-1)左上,(1,-1)右上,(-1,1)左下
      BoxShadow(
        color: Colors.red,
        offset: Offset(1, 1),
        blurRadius: 5,
      ),
      BoxShadow(
          color: Colors.blueAccent, offset: Offset(-1, -1), blurRadius: 5),
    ]),
    child: FlatButton(
        onPressed: () {},
        color: Colors.white,
        padding: EdgeInsets.only(left: 30, right: 30, top: 10, bottom: 10),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Icon(
              Icons.launch,
              color: Colors.black87,
              size: 20,
            ),
            Text(
              "分享",
              style: TextStyle(
                color: Colors.grey[400],
                fontSize: 10,
              ),
            )
          ],
        )),
  );
}

```

### 第四个图片

```
Widget _flatButton2() {
  return Container(
    margin: EdgeInsets.only(top: 20, left: 20),
    decoration: BoxDecoration(shape: BoxShape.rectangle, boxShadow: [
      //BoxShadow中Offset (1,1)右下，(-1,-1)左上,(1,-1)右上,(-1,1)左下
      BoxShadow(
        color: Colors.red,
        offset: Offset(-1, 1),
        blurRadius: 5,
      ),
      BoxShadow(color: Colors.blueAccent, offset: Offset(1, -1), blurRadius: 5),
    ]),
    child: FlatButton(
        onPressed: () {},
        color: Colors.white,
        padding: EdgeInsets.only(left: 30, right: 30, top: 10, bottom: 10),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Icon(
              Icons.launch,
              color: Colors.black87,
              size: 20,
            ),
            Text(
              "分享",
              style: TextStyle(
                color: Colors.grey[400],
                fontSize: 10,
              ),
            )
          ],
        )),
  );
}
```

### 第五个图片

```
Widget _materialButton() {
  return Padding(
    padding: EdgeInsets.only(top: 20, left: 20),
    child: MaterialButton(
      padding: EdgeInsets.only(left: 30, right: 30, top: 10, bottom: 10),
      color: Colors.white,
      onPressed: () {},
      elevation: 5,
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Icon(
            Icons.launch,
            color: Colors.black87,
            size: 20,
          ),
          Text(
            "分享",
            style: TextStyle(
              color: Colors.grey[400],
              fontSize: 10,
            ),
          )
        ],
      ),
    ),
  );
}
```

