---
title: 矩阵变换Transform类Matrix4
date: 2020-07-22 15:18:44
tags:
	- flutter
categories: 
	- flutter
	- widget组件
---

 和尚刚学习了 **Transform** 类，其核心部分在于矩阵变换，而矩阵变换是由 **Matrix4** 处理的，且无论是如何的平移旋转等操作，根本上还是一个四阶矩阵操作的；接下来和尚学习一下 **Matrix4** 的基本用法；

### **基本构造**

#### **Matrix4(double arg0, … double arg15)**

 **Matrix4** 默认构造函数由 **16** 个参数，从左到右从上到下依此排列为一个四阶矩阵；

```javascript
transform: Matrix4(1.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 1.0),
```

#### **Matrix4.identity()**

 **Matrix4.identity()** 会初始化一个如上的 **Matrix4**，可在此基础上进行其他矩阵操作；

```javascript
transform: Matrix4.identity(),
transform: Matrix4.identity()..rotateZ(pi / 4),
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-bd2a9a1f88f4d6bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)Matrix4.zero()

 **Matrix4.zero()** 会初始化一个所有参数值为 **0** 的空矩阵，在此基础上设置具体的变化矩阵；查看源码所有的初始化都是从 **Matrix4.zero()** 开始的；

```javascript
transform: Matrix4.zero(),
transform: Matrix4.zero()..setIdentity(),
```

#### ![image.png](https://upload-images.jianshu.io/upload_images/4118241-f5338c7a8bf2737d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Matrix4.fromList()

 **Matrix4.fromList()** 将 **List** 列表中数据赋值进入 **Matrix4(double arg0, … double arg15)** 类似；

```javascript
List<double> list = [1.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 1.0];
transform: Matrix4.fromList(list),
```

#### **Matrix4.copy()**

 **Matrix4.copy()** 拷贝一个已有的 **Matrix4**；

```javascript
transform: Matrix4.copy(Matrix4.identity()),
```

#### **Matrix4.columns()**

 **Matrix4.columns()** 由四个 **4D** 列向量组成；

```javascript
import 'package:vector_math/vector_math_64.dart' as v;

transform: Matrix4.columns(
    v.Vector4(1.0, 0.0, 0.0, 0.0),
    v.Vector4(0.0, 1.0, 0.0, 0.0),
    v.Vector4(0.0, 0.0, 1.0, 0.0),
    v.Vector4(0.0, 0.0, 0.0, 1.0)),
```

#### **Matirx4.inverted()**

 **Matrix4.inverted()** 为逆向矩阵，与原 **Matrix4** 矩阵相反(矩阵坐标沿着左对角线对称)；

```javascript
transform: Matrix4.inverted(Matrix4.fromList(list)),
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-e175dacdf64d04d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### **Matrix4.outer()**

 **Matrix4.outer()** 为两个四阶矩阵的合并乘积，注意两个四阶矩阵的先后顺序决定最终合并后的矩阵数组；

```javascript
transform: Matrix4.outer(v.Vector4(1.0, 1.0, 1.0, 1.20), v.Vector4.identity()),
transform: Matrix4.outer(v.Vector4.identity(), v.Vector4(1.0, 1.0, 1.0, 1.20)),
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-d3dc7ac48dd73d6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **Scale 缩放构造方法**

#### **Matrix4.diagonal3()**

 **Matrix4.diagonal3()** 通过 **Vector3** 设置缩放矩阵；

```javascript
transform: Matrix4.diagonal3(v.Vector3(2.0, 1.0, 1.0)),
transform: Matrix4.diagonal3(v.Vector3.array([2.0, 2.0, 2.0])),
```

​      分析 **Vector3** 两种构造方法，三个参数分别对应 **x / y /z** 轴方向缩放；

```javascript
factory Vector3(double x, double y, double z) =>
      new Vector3.zero()..setValues(x, y, z);

factory Vector3.array(List<double> array, [int offset = 0]) =>
      new Vector3.zero()..copyFromArray(array, offset);
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-fff2193496ad52f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### **Matirx4.diagonal3Values()**

 **Matrix4.diagonal3Values()** 类似于将上述构造方法提取出来，直接对三个参数进行缩放赋值；

```javascript
transform: Matrix4.diagonal3Values(2.0, 1.0, 1.0),
```

​      分析 **diagonal3Values** 源码，发现 **Matrix4** 矩阵中坐对角线上的值分别对应 **x / y / z** 轴方向的缩放；

```javascript
factory Matrix4.diagonal3Values(double x, double y, double z) =>
    new Matrix4.zero()
      .._m4storage[15] = 1.0
      .._m4storage[10] = z
      .._m4storage[5] = y
      .._m4storage[0] = x;
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-8158e2702d1fef33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **Transform 平移构造方法**

#### **Matrix4.translation()**

 **Matrix4.translation** 同样通过 **Vector3** 构造方法的各参数设置矩阵平移量；水平向右为 **x** 轴正向，竖直向下为 **y** 轴正向；

```javascript
transform: Matrix4.translation(v.Vector3(10.0, 10.0, 10.0)),
transform: Matrix4.translation(v.Vector3.array([-10.0, -10.0, 10.0])),
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-9406610d41b87665.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### **Matrix4.translationValues()**

 **Matrix4.translationValues()** 将矩阵平移量直接赋值展示；

```javascript
transform: Matrix4.translationValues(10.0, 10.0, 10.0),
```

​      分析 **translationValues** 源码，**Matirx4** 四阶矩阵中第四行前三列分别对应 **x / y / z** 轴方向的偏移量；

```javascript
factory Matrix4.translationValues(double x, double y, double z) =>
    new Matrix4.zero()
      ..setIdentity()
      ..setTranslationRaw(x, y, z);

void setTranslationRaw(double x, double y, double z) {
  _m4storage[14] = z;
  _m4storage[13] = y;
  _m4storage[12] = x;
}
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-402f0f91a58edca0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **Rotation 旋转构造方法**

#### **Matrix4.rotationX()**

 **Matrix4.rotationX()** 沿 **x** 轴方向旋转；

```javascript
transform: Matrix4.rotationX(pi / 3),
```

​      分析源码，四阶矩阵中，**index** 为 **5/6/9/10** 共同操作旋转弧度；

```javascript
void setRotationX(double radians) {
    final double c = math.cos(radians);
    final double s = math.sin(radians);
    _m4storage[0] = 1.0;  _m4storage[1] = 0.0;
    _m4storage[2] = 0.0;  _m4storage[4] = 0.0;
    _m4storage[5] = c;    _m4storage[6] = s;
    _m4storage[8] = 0.0;  _m4storage[9] = -s;
    _m4storage[10] = c;   _m4storage[3] = 0.0;
    _m4storage[7] = 0.0;  _m4storage[11] = 0.0;
}
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-bb4e5d57588aac65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### **Matrix4.rotationY()**

 **Matrix4.rotationY()** 沿 **y** 轴方向旋转；

```javascript
transform: Matrix4.rotationY(pi / 3),
```

​      分析源码，四阶矩阵中，**index** 为 **0/2/8/10** 共同操作旋转弧度；

```javascript
void setRotationY(double radians) {
    final double c = math.cos(radians);
    final double s = math.sin(radians);
    _m4storage[0] = c;    _m4storage[1] = 0.0;
    _m4storage[2] = -s;   _m4storage[4] = 0.0;
    _m4storage[5] = 1.0;  _m4storage[6] = 0.0;
    _m4storage[8] = s;    _m4storage[9] = 0.0;
    _m4storage[10] = c;   _m4storage[3] = 0.0;
    _m4storage[7] = 0.0;  _m4storage[11] = 0.0;
}
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-04c2be60d2729d62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### **Matrix4.rotationZ()**

 **Matrix4.rotationZ()** 沿 **z** 轴方向旋转；

```javascript
transform: Matrix4.rotationZ(pi / 3),
```

​      分析源码，四阶矩阵中，**index** 为 **0/1/4/5** 共同操作旋转弧度；

```javascript
void setRotationZ(double radians) {
    final double c = math.cos(radians);
    final double s = math.sin(radians);
    _m4storage[0] = c;
    _m4storage[1] = s;
    _m4storage[2] = 0.0;   _m4storage[4] = -s;
    _m4storage[5] = c;     _m4storage[6] = 0.0;
    _m4storage[8] = 0.0;   _m4storage[9] = 0.0;
    _m4storage[10] = 1.0;  _m4storage[3] = 0.0;
    _m4storage[7] = 0.0;   _m4storage[11] = 0.0;
}
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-cb30be9ef0cd5a5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **Skew 斜切**

#### **Matrix4.skewX()**

​      上一篇博客稍稍介绍过，**skewX()** 是沿 **x** 轴方向斜切；

```javascript
transform: Matrix4.skewX(pi / 6),
```

​      分析源码，四阶矩阵中，第二行第一列元素对应斜切值，即三角函数中 **tan**；

```javascript
factory Matrix4.skewX(double alpha) {
    final Matrix4 m = new Matrix4.identity();
    m._m4storage[4] = math.tan(alpha);
    return m;
}
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-b5f1a24e22c5147a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### **Matrix4.skewY()**

 **skewY()** 是沿 **y** 轴方向斜切；

```javascript
transform: Matrix4.skewY(pi / 6),
```

​      分析源码，四阶矩阵中，第一行第二列元素对应斜切值，即三角函数中 **tan**；

```javascript
factory Matrix4.skewY(double beta) {
    final Matrix4 m = new Matrix4.identity();
    m._m4storage[1] = math.tan(beta);
    return m;
}
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-ec0812ef169da235.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### **Matrix4.skew()**

 **skew()** 是沿 **x / y** 轴方向斜切；

```javascript
transform: Matrix4.skew(pi / 6, pi / 6),
```

​      分析源码，四阶矩阵中，将上述两个元素结合展示效果；

```javascript
factory Matrix4.skew(double alpha, double beta) {
    final Matrix4 m = new Matrix4.identity();
    m._m4storage[1] = math.tan(beta);
    m._m4storage[4] = math.tan(alpha);
    return m;
}
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-f34d93e4968df09c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **组合构造**

#### **Matrix4.compose()**

 **Matrix4.compose()** 可以将平移/旋转/缩放共同组合操作绘制；

```javascript
transform: Matrix4.compose(v.Vector3(10.0, 10.0, 10.0), v.Quaternion.random(math.Random(10)), v.Vector3(1.5, 1.0, 1.0)),
```

​      分析源码，三个参数分别对应平移量/旋转量/缩放量；旋转量用到了欧拉旋转，和尚还不是很理解，只是在测试中用了 **v.Quaternion.random()** 的一个构造方法，还有待深入探索；

```javascript
factory Matrix4.compose(
        Vector3 translation, Quaternion rotation, Vector3 scale) =>
    new Matrix4.zero()
      ..setFromTranslationRotationScale(translation, rotation, scale);
```

![image.png](https://upload-images.jianshu.io/upload_images/4118241-dafa28c1288dcaab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

------

 **Matirx4** 涉及的范围很广泛，还有很多方法和尚没有研究到，只是尝试了一些常用的构造方法，若有错误的地方请多多指导！