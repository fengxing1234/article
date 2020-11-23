---
title: DateTime使用
date: 2020-08-11 20:19:02
tags:
---

# DateTime 类

  DateTime类用来标识一个瞬时的时间节点，可以通过构造函数，从标准格式（符合ISO 8601标准）的字符串中构造一个时间对象。DateTIme使用24小时计时法。以下是最基础的例子：

```
var now = new DateTime.now();
var berlinWallFell = new DateTime.utc(1989, 11, 9);
var moonLanding = DateTime.parse("1969-07-20 20:18:04Z");  // 8:18pm
复制代码
```

DateTime对象会锚定UTC(通用协调时Universal Time Coordinated)时区或者设备的本地时区。在创建之后，DateTime的值和所属的时区都不会改变。可以通过对象属性读取具体的时间值：

```
assert(berlinWallFell.month == 11); //  柏林墙倒塌的月份
assert(moonLanding.hour == 20); //  登月时的时间（小时）
复制代码
```

出于便捷性和可读性的考量，DateTIme类提供了星期和月份的常量值可供调用，你可以使用很多常量来提高代码的可读性，示例如下：

```
var berlinWallFell = new DateTime.utc(1989, DateTime.november, 9);
assert(berlinWallFell.weekday == DateTime.thursday);
复制代码
```

星期和月份的常量都是从1开始的，每周的开始从周一开始计算，所以january和monday的值都是1。

## 构造方法

`DateTime(int year, [ int month = 1 int day = 1 int hour = 0 int minute = 0 int second = 0 int millisecond = 0 int microsecond = 0 ])`
 该方法创建一个依托于本地时区的DateTime实例
 `DateTime.fromMicrosecondsSinceEpoch(int microsecondsSinceEpoch, { bool isUtc: false })`
 该方法输入距离1970年1月1日0时0分0秒的微秒数，得到一个DateTime实例。
 `DateTime.fromMillisecondsSinceEpoch(int millisecondsSinceEpoch, { bool isUtc: false })`
 该方法输入距离1970年1月1日0时0分0秒的毫秒数，得到一个DateTime实例。
 `DateTime.now()`
 该方法返回一个依托于本地时区的代表当前时间的DateTime实例。
 `DateTime.utc(int year, [ int month = 1 int day = 1 int hour = 0 int minute = 0 int second = 0 int millisecond = 0 int microsecond = 0 ])`
 该方法返回一个依托于UTC的DateTime实例

## 实例方法

`add(Duration duration) → DateTime`
 加上传入的Duration实例所表示的时间间隔，返回一个新的DateTime实例，新实例与调用实例之间的时间差为duration

```
  var now = DateTime.now();
  var span = Duration(days: 1);
  print(now);
  print(now.add(span));
复制代码
```

输出：
 2019-12-15 09:29:59.605510
 2019-12-16 09:29:59.605510

`compareTo(DateTime other) → int`
 比较两个DateTime实例，如果二者相等，则返回0
 `difference(DateTime other) → Duration`
 返回当前实例与传入实例之间的时间差，返回一个Duration实例
 `isAfter(DateTime other) → bool`
 同传入的DateTime进行比较，如果标识的时间在传入实例之后，则返回true
 `isAtSameMomentAs(DateTime other) → bool`
 同传入的DateTime进行比较，如果二者表示的时间在同一时刻，则返回true
 `isBefore(DateTime other) → bool`
 同传入的DateTime进行比较，如果标识的时间在传入实例之前，则返回true
 `subtract(Duration duration) → DateTime`
 减去传入的Duration实例所表示的时间间隔，返回一个新的DateTime实例，新实例与调用实例之间的时间差为duration

```
  var now = DateTime.now();
  var span = Duration(days: 1);
  print(now);
  print(now.subtract(span));
复制代码
```

输出：
 2019-12-15 09:32:02.811829
 2019-12-14 09:32:02.811829

`toIso8601String() → String`
 返回依照IOS-8601标准的全精度表示的时间格式
 `toLocal() → DateTime`
 返回该实例在本地时区下表示的实例
 `toString() → String`
 返回该实例的字符串表现
 `toUtc() → DateTime`
 返回该实例在UTC时间表示下的实例

## 操作符

`operator ==(dynamic other) → bool`
 只有当传入值为DateTime实例并且表示的时间相同且在同一时区下时返回true

## 静态方法

`parse(String formattedString) → DateTime`
 通过标准格式的字符串来构造一个DateTime实例

```
  var now = DateTime.parse('2019-12-12');
  print(now);
复制代码
```

输出：
 2019-12-12 00:00:00.000

`tryParse(String formattedString) → DateTime`
 通过标准格式的字符串来构造一个DateTime实例

## 实例属性

`day → int`
 返回实例的日期，从取值范围从1到30
 `hashCode → int`
 返回实例对象的hash码
 `hour → int`
 返回在24小时计时法中的小时时间，取值范围从0到23
 `isUtc → bool`
 返回当前实例是否是UTC计时
 `microsecond`
 返回实例表示的时间距离1970年1月1日0时0分0秒的微秒数的最后三位数，取值范围0到999
 `microsecondsSinceEpoch → int`
 返回实例表示的时间距离1970年1月1日0时0分0秒的微秒数
 `millisecond → int`
 返回实例表示的时间距离1970年1月1日0时0分0秒的毫秒数的最后三位数，取值范围0到999
 `millisecondsSinceEpoch → int`
 返回实例表示的时间距离1970年1月1日0时0分0秒的毫秒数
 `minute → int`
 返回实例表示时间的分钟数，取值范围从0到59
 `month → int`
 返回实例表示时间的月份，取值范围从1到12 `second → int`
 返回实例表示时间的月份，取值范围从0到59
 `timeZoneName → String`
 返回实例的时区名 `timeZoneOffset → Duration`
 返回当前时区与UTC时间之间的时间间隔，用Duration对象来表示
 `weekday → int`
 返回实例表示时间是星期几，取值范围从1到7 `year → int` 返回实例表示时间的年份

## 使用UTC时区和本地时区

DateTime实例默认使用本地时区，除非在创建时显示声明使用UTC时区。

```
var dDay = new DateTime.utc(1944, 6, 6);
复制代码
```

可以使用`isUtc`来确定当前的时间是否是UTC时区。可以使用`toLocal`和`toUtc`来进行时区间的转换，使用`timeZoneName`来获取DateTime实例所属时区的简写，使用`timeZoneOffset`来获取不同时区之间的时间差值。

## DateTime实例之间的比较

DateTime类有几个简单的方法来实现DateTime实例之间的比较，比如`isAfter`、`isBefore`、`isAtSameMomentAs`:

```
assert(berlinWallFell.isAfter(moonLanding) == true);
assert(berlinWallFell.isBefore(moonLanding) == false);
复制代码
```

## DateTime与Duration类的配合使用

可以通过`add`和`subtract`方法加减一个Duration对象来对DateTime实例进行操作，并返回一个新的实例。例如，计算六天后的今天的时间：

```
var now = new DateTime.now();
var sixtyDaysFromNow = now.add(new Duration(days: 60));
复制代码
```

为了找出两个DateTime实例之间的差值，我们可以使用`difference`方法，它将返回一个Duration实例：

```
var difference = berlinWallFell.difference(moonLanding);
assert(difference.inDays == 7416);
复制代码
```

两个不同时区之间的时间差就是按照两地之间的时间纳秒差，这个结果并不会补偿日历日或时制之间的差别。这就意味着如果相邻两个午夜时间跨过了夏令时的变更日期，那么这两个午夜之间的时间差可能不满24个小时。上述代码中的时间差如果用澳大利亚本地时间计算，那么结果将为7415天又23小时，按照整天来计算只有7415天。

## 其他资源

关于时间区间(span of time)的表示方法，详见`Duration`，关于时间间隔的表示方法，详见`Stopwatch`,DateTime类并没有提供国际化的相关功能。为了进行国际化，可以使用`intl`包。

## 常量

`april → const int`
 4
 `august → const int`
 8
 `daysPerWeek → const int`
 7
 `december → const int`
 12
 `february → const int`
 2
 `friday → const int`
 5
 `january → const int`
 1
 `july → const int`
 7
 `june → const int`
 6
 `march → const int`
 3
 `may → const int`
 5
 `monday → const int`
 1
 `monthsPerYear → const int`
 12
 `november → const int`
 11
 `october → const int`
 10
 `saturday → const int`
 6
 `september → const int`
 9
 `sunday → const int`
 7
 `thursday → const int`
 4
 `tuesday → const int`
 2
 `wednesday → const int`
 3