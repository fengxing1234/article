---
title: 矩阵变换Transform类基本使用
date: 2020-07-22 14:43:29
tags:
	- flutter
categories: 
	- flutter
	- widget组件
---

和尚在学习矩阵变换时需要用到 **Transform** 类，可以实现子 **Widget** 的 **scale 缩放 / translate 平移 / rotate 旋转 / skew 斜切** 等效果，对应于 **Canvas** 绘制过程中的矩阵变换等；和尚今对此进行初步整理；

### 一、scale 缩放

 **scale 缩放** 可以通过 **Transform** 提供的构造方法或 **Matrix4** 矩阵变化来实现；

#### **Transform.scale 构造方法**

```javascript
Transform.scale({
    Key key,
    @required double scale,     // 缩放比例
    this.origin,    // 缩放原点
    this.alignment = Alignment.center,  // 对齐方式
    this.transformHitTests = true,
    Widget child,
}) : transform = Matrix4.diagonal3Values(scale, scale, 1.0),
    super(key: key, child: child);
```

**Tips:**

1. 设置缩放比例后，水平/竖直方向按同比例缩放，**z** 轴方向不缩放；
2. 对齐方式是与初始位置为准;

```javascript
Center(
    child: Transform.scale(
        scale: 1.2,
//      origin: Offset(120.0, 120.0),
        alignment: Alignment.bottomRight,
        child: ClipOval(
            child: SizedBox(
                width: 120.0,
                height: 80.0,
                child: Container(
                    color: Colors.blueAccent))))),
```

#### **Transform Matrix4 方式**

```javascript
void scale(dynamic x, [double y, double z]) {}
```

 **Matrix4** 为 **4D** 矩阵，使用更灵活，可以分别设置 **x / y / z** 轴方向缩放比例；若只设置一个则水平/垂直方向同比例缩放；

```javascript
Center(
    child: Transform(
        transform: Matrix4.identity()
          ..scale(1.2, 1.0, 0.5),
        alignment: Alignment.bottomRight,
        child: ClipOval(
            child: SizedBox(
                width: 120.0,
                height: 80.0,
                child: Container(
                    color: Colors.blueAccent)))))
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-6db544c72d1c62df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 二、translate 平移

 **translate 平移** 可通过构造方法或 **Matrix4** 矩阵变化来实现；

#### **Transform.translate 构造方法**

```javascript
Transform.translate({
    Key key,
    @required Offset offset,
    this.transformHitTests = true,
    Widget child,
}) : transform = Matrix4.translationValues(offset.dx, offset.dy, 0.0),
    origin = null,
    alignment = null,
    super(key: key, child: child);
```

 **translate** 按坐标点 **Offset** 平移，水平向右为正向，竖直向下为正向；**z** 轴方向不平移；

```javascript
Center(
    child: Transform.translate(
        offset: Offset(60.0, -40.0),
        child: ClipOval(
            child: SizedBox(
                width: 120.0,
                height: 80.0,
                child: Container(
                    color: Colors.blueAccent))))),
```

#### Transform Matrix4 方式

```javascript
void translate(dynamic x, [double y = 0.0, double z = 0.0]) {}
```

 **Matrix4** 平移方式可分别设置 **x / y / z** 轴方向平移量，必须设置 **x** 轴方向平移量；

```javascript
Center(
    child: Transform(
        transform: Matrix4.identity()
          ..translate(60.0, -40.0, 100.0),
        alignment: Alignment.bottomRight,
        child: ClipOval(
            child: SizedBox(
                width: 120.0,
                height: 80.0,
                child: Container(
                    color: Colors.blueAccent))))),
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-45808af4509c7d2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 三、rotate 旋转

 **rotate 旋转**可通过构造方法或 **Matrix4** 矩阵变化来实现；

#### **Taransform.rotate 构造方法**

```javascript
Transform.rotate({
    Key key,
    @required double angle,     // 旋转角度
    this.origin,
    this.alignment = Alignment.center,
    this.transformHitTests = true,
    Widget child,
}) : transform = Matrix4.rotationZ(angle),
       super(key: key, child: child);
```

​      由此可看出旋转是沿 **z** 轴旋转，即垂直手机屏幕方向，视觉上的正常旋转；

```javascript
Center(
    child: Transform.rotate(
        angle: pi / 4,
        child: ClipOval(
            child: SizedBox(
                width: 120.0,
                height: 80.0,
                child: Container(
                    color: Colors.blueAccent))))),
```

#### **Transform Matrix4 方式**

```javascript
void rotate(Vector3 axis, double angle) {}
```

 **Matrix4** 可灵活设置旋转方向，包括沿 **x / y / z** 轴方向立体旋转，且旋转效果可以重叠；而 **Matrix4** 也提供了两种旋转方式；

1. **Matrix4.rotationZ**
2. **Matrix4.identity()..rotateZ**

​      对于单轴旋转，两种方式实际是完全相同的，且第一种方式只可进行单轴旋转；第二种方式更灵活，可以多个轴叠加；

```javascript
Center(
    child: Transform(
        transform: Matrix4.rotationZ(pi / 4),
        alignment: Alignment.center,
        child: ClipOval(
            child: SizedBox(
                width: 120.0,
                height: 80.0,
                child: Container(
                    color: Colors.blueAccent))))),
Center(
    child: Transform(
        transform: Matrix4.identity()..rotateX(pi / 4),
        alignment: Alignment.center,
        child: ClipOval(
            child: SizedBox(
                width: 120.0,
                height: 80.0,
                child: Container(
                    color: Colors.greenAccent))))),
Center(
    child: Transform(
        transform: Matrix4.identity()..rotateY(pi / 4),
        alignment: Alignment.center,
        child: ClipOval(
            child: SizedBox(
                width: 120.0,
                height: 80.0,
                child: Container(color: Colors.brown))))),
Center(
    child: ClipOval(
        child: SizedBox(
            width: 120.0,
            height: 80.0,
            child: Container(color: Colors.black12))))
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-59710ac9b087e311.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![image.png](https://upload-images.jianshu.io/upload_images/4118241-d3716bf6327ce8f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 四、skew 斜切

 **Transform** 未提供关于 **skew** 斜切的构造方法，只能用 **Matrix4** 方式构建；

1. **skewX** 沿水平方向斜切；
2. **skewY** 沿竖直方向斜切；
3. **skew** 与 **x / y** 轴共同矩阵转换产生斜切；

```javascript
Center(
    child: Transform(
        transform: Matrix4.skewY(pi / 4),
        alignment: Alignment.topLeft,
        child: Container(
            width: 120.0,
            height: 80.0,
            color: Colors.brown))),
Center(
    child: Transform(
        transform: Matrix4.skewX(pi / 4),
        alignment: Alignment.topLeft,
        child: Container(
            width: 120.0,
            height: 80.0,
            color: Colors.redAccent))),
Center(
    child: Transform(
        transform: Matrix4.skew(pi / 6, pi / 6),
        alignment: Alignment.topLeft,
        child: Container(
            width: 120.0,
            height: 80.0,
            color: Colors.white70))),
Center(
    child: Transform(
        transform: Matrix4.skew(0.0, 0.0),
        alignment: Alignment.topLeft,
        child: Container(
            width: 120.0,
            height: 80.0,
            color: Colors.black12))),
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-7ae245fad8193302.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)      所有的矩阵变换均可通过 **Matrix4** 叠加，在实际应用中更加灵活，下节会重点学习 **Matrix4** 矩阵方面的小知识点；