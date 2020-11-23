---
title: AndroidManifest--Application详解
date: 2020-10-19 13:42:46
tags:
---

#### application 语法（SYNATX）：

```
<application 
    android:allowTaskReparenting=["true" | "false"]            
    android:allowBackup=["true" | "false"]             
    android:backupAgent="*string*"             
    android:banner="*drawable resource*"     
    android:extractNativeLibs=["true" | "false"]    
    android:fullBackupContent="string"  
    android:fullBackupOnly=["true" | "false"]  
    android:debuggable=["true" | "false"]             
    android:description="*string resource*"            
    android:enabled=["true" | "false"]             
    android:hasCode=["true" | "false"]             
    android:hardwareAccelerated=["true" | "false"]             
    android:icon="*drawable resource*"             
    android:isGame=["true" | "false"]             
    android:killAfterRestore=["true" | "false"]             
    android:largeHeap=["true" | "false"]             
    android:label="*string resource*"             
    android:logo="*drawable resource*"             
    android:manageSpaceActivity="*string*"             
    android:name="*string*"             
    android:permission="*string*"             
    android:persistent=["true" | "false"]             
    android:process="*string*"             
    android:restoreAnyVersion=["true" | "false"]             
    android:requiredAccountType="*string*"             
    android:restrictedAccountType="*string*"             
    android:supportsRtl=["true" | "false"]             
    android:taskAffinity="*string*"             
    android:testOnly=["true" | "false"]             
    android:theme="*resource or theme*"             
    android:uiOptions=["none" | "splitActionBarWhenNarrow"]    
    android:usesCleartextTraffic=["true" | "false"]             
    android:vmSafeMode=["true" | "false"] >    
   . . .
</application>
```

#### 能够包含的元素（CAN CONTAIN）：

```
<activity>

<activity-alias>

<service>

<receiver>

<provider>

<uses-library>
```

#### 说明（DESCRIPTION）：

这个元素用于应用程序的声明。它包含了每个应用程序组件所声明的子元素，并且还有能够影响所有组件的属性。其中的很多属性（如icon、label、permission、process、taskAffinity和allowTaskReparenting）会给组件元素中对应的属性设置默认值。其他的给是应用程序整体设置的值（如debuggable、enabled、description、allowClearUserData），并且这些属性值不能被组件的属性所覆盖。

#### 属性（ATTRIBUTES）：

###### Android:allowTaskReparenting

当一个与当前任务有亲缘关系的任务被带到前台时，用这个属性来指定应用程序中定义的Activity能否从他们当前的任务中转移到这个有亲缘关系的任务中。如果设置为true，则能够转移，如果设置为`false`，则应用程序中的Activity必须保留在它们所在的任务中。默认值是`false`。

`<activity>`元素有它们自己的`allowTaskReparenting`属性，它能够覆盖`<application>`元素中的设置。

###### android:allowBackup

这个标识用来表示是否允许应用备份相关的数据并且在必要时候恢复还原这些数据，如果该标识设为`false`，则代表不备份和恢复任何的应用数据，默认的该标识属性为`true`。当 `allowBackup` 标识设置为 `true` 时，用户即可以通过 `adb backup`和 `adb restore`来进行对应数据的备份和恢复，这个在很多时候会带来一定的安全风险。　　
 `adb backup` 命令容许任何一个打开 USB 调试开关的人从 Android 手机中复制应用数据到外设，一旦应用数据被备份之后，所有应用数据都可被用户读取；`adb restore`命令允许用户指定一个恢复的数据来源(即备份的应用数据)来恢复应用程序数据的创建。因此，当一个应用数据被备份之后，用户即可在其他Android手机或模拟器上安装同一个应用，以及通过恢复该备份的应用数据到该设备上，在该设备上打开该应用即可恢复到被备份的应用程序的状态。尤其是通讯录应用，一旦应用程序支持备份和恢复功能，攻击者即可通过`adb backup`和`adb restore`进行恢复新安装的同一个应用来查看聊天记录等信息；对于支付金融类应用，攻击者可通过此来进行恶意支付、盗取存款等；因此为了安全起见，开发者务必将`allowBackup`标志值设置为`false`来关闭应用程序的备份和恢复功能，以免造成信息泄露和财产损失。　　
 网上也可以看到很多将 `allowBackup` 设置为`true`带来的许多风险。
 可以看看这篇博客：[详解Android App AllowBackup配置带来的风险](https://link.jianshu.com?t=http%3A%2F%2Fwww.91ri.org%2F12500.html)。

###### android:backupAgent

这个属性用于定义应用程序备份代理的实现类的名称，这个类是`BackupAgent`类的一个子类。它的属性值应该是完整的Java类的名称（如，`com.example.project.MyBackupAgent`）。但是，也可以使用用”.”符号开头的简称（如，`.MyBackupAgent`），系统会把`<manifest>`元素中指定的包名追加到”.”符号的前面。

###### android:banner

这个标识是用在 Android TV 电视上用轮播图来代表一个应用，由于轮播图只是在 HOME 界面上显示的，所以它仅仅只能被一个带有能够处理 [CATEGORY_LEANBACK_LAUNCHER](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fcontent%2FIntent.html%23CATEGORY_LEANBACK_LAUNCHER) intent Activity 的应用声明。由于这个标识是 Android TV 开发使用到的，在这里就不详细介绍了，具体的可以看 Google 的 API 文档。

###### android:extractNativeLibs

这个标识为`android 6.0`引入，该属性如果设置为`false`,则系统在安装应用的时候不会把 so 文件从 apk 中解压出来了，同时修改 `System.loadLibrary`直接打开调用 apk 中的 so 文件。但是，目前要让该技巧生效还需要额外2个条件：一个是apk 中的 .so 文件不能被压缩；二个是 .so 必须用`zipalign -p 4`来对齐。该标识的默认值为`true`。

###### android:fullBackupContent

这个标识用来指明备份的数据的规则，该标识当然是配合[Auto Backup for Apps](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fdata%2Fautobackup.html)来使用的，它也是在 Android 6.0 中引入的，使用的方式如下所示：



```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"  
        xmlns:tools="http://schemas.android.com/tools"  
        package="com.my.appexample">  
    ...
    <app ...  
        android:fullBackupContent="@xml/mybackupscheme">  
    </app>  
    ...  
</manifest>
```

在此示例代码中，android:fullBackupContent 属性指定了一个 XML 文件。该文件名为mybackupscheme.xml，位于应用开发项目的 res/xml/ 目录中。 此配置文件包括关于要备份哪些文件的规则。 下列示例代码显示了将某一特定文件排除在备份之外的配置文件：



```xml
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <exclude domain="database" path="device_info.db"/>
<full-backup-content>
```

此示例备份配置仅将一个特定数据库文件排除在备份之外，所有其他文件均予以备份。

###### android:fullBackupOnly

这个标识用来指明当[Auto Backup for Apps](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fdata%2Fautobackup.html)功能可以使用的时候是否开启它。如果该标识设置为 true，在 Android 6.0 以及之上的手机上，应用将会执行[Auto Backup for Apps](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fdata%2Fautobackup.html)功能，在之前的 Android 版本中，你的应用将会自动忽略该标识，并且切换成 [Key/Value Backups](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fdata%2Fkeyvaluebackup.html)。

###### android:debuggable

这个属性用于指定应用程序是否能够被调试，即使是以用户模式运行在设备上的时候。如果设置为`true`，则能够被调试，否则不能调试，默认值是`false`。

###### android:description

这个属性用于定义应用程序相关的用户可读文本，它要比应用程序标签更长、更详细。它的的值必须被设置成一个字符串资源的引用。跟label属性不一样，label属性可以使用原生的字符串。这个属性没有默认值。

###### android:enabled

这个属性用于指定Android系统能否实例化应用程序组件。如果设置为true，这个可以实例化其组件，否则不能够实例化。如果这个属性被设置为true，那么就会使用每个组件自己enabled属性的设置来判断其是否能够被实例化。如果这个属性被设置为false，它会覆盖其所有组件自己指定的值，应用程序中的所有组件都会被禁用。

默认值是`true`。

###### android:hasCode

这个属性用于设置应用程序是否包含了代码，如果设置为true，则包含代码，否则不包含任何代码。在这个属性被设置为false的时候，系统在加载组件的时候不会试图加载任何应用程序的代码。默认值是`true`。

如果应用程序没有使用任何应用内置组件类以外的组件，那么这个应用程序就不会有任何自己的代码，像使用AliasActivity类的Activity，是很少发生的。

###### android:hardwareAccelerated

是否为应用程序中所有的 Activity 和 View 启用硬件加速渲染功能`true`表示开启，`“false”`表示关闭。 如果 minSdkVersion 或 targetSdkVersion 的值大于等于`14`，则本属性默认值是`true`,否则，默认值为`false`。　　
 自 Android 3.0 （API 级别 11）开始，应用程序可以使用硬件加速的 OpenGL 渲染功能来提高很多常用 2D 图形操作的性能。 当开启硬件加速渲染功能时，大部分 Canvas、Paint、Xfermode、ColorFilter、Shader 和 Camera 中的操作都会被加速。 即便应用程序没有显式地调用系统的 OpenGL 库，这仍能使动画更加平滑、屏幕滚动也更加流畅、整体响应速度获得改善。　　
 请注意，并非所有的 OpenGL 2D 操作都会被加速。 如果开启了硬件加速渲染功能，请对应用程序进行测试以确保使用渲染时不会出错。更多详细的操作可以去查看 google 文档[硬件加速指南](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fgraphics%2Fhardware-accel.html)。

###### android:isGame

这个标识用来指明该应用是否是游戏，这样就能够将该应用和其他应用分离开来，默认的改值为 false。

###### android:icon

这个属性用于设置应用程序的整个图标，以及每个应用组件的默认图标。对于<activity>、<activity-alias>、<service>、<service>、<receiver>和<provider>元素，请看它们各自的icon属性。

设置这个属性时，必须要引用一个包含图片的可绘制资源（例如，“@drawable/icon”）。没有默认的图标。

###### android:killAfterRestore

这个属性用于指定在全系统的恢复操作期间，应用的设置被恢复以后，对应的问题程序是否应该被终止。单包恢复操作不会导致应用程序被关掉。全系统的复原操作通常只会发生一次，就是在电话被首次建立的时候。第三方应用程序通常不需要使用这个属性。

默认值是true，这意味着在全系统复原期间，应用程序完成数据处理之后，会被终止。

###### android:largeHeap

这个标识用来表明这个应用的进程是否需要更大的运行内存空间，这个标识对该应用的所有进程都有效，但是需要注意的一点是，这仅仅对第一个加载进这个进程的应用起作用，如果用户通过 `sharedUserId` 将多个应用置于同一个进程（SharedUserId 的具体用法可以参考：[android IPC通信（上）－sharedUserId&&Messenger](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Fself_study%2Farticle%2Fdetails%2F50164199%23t0)），那么两个应用都必须要指定该标识并且设置为同一个值，要不然就会产生意想不到的结果。　　
 大部分应用程序不需要用到本属性，而是应该关注如何减少内存消耗以提高性能。 使用本属性并不能确保一定会增加可用的内存，因为某些设备可用的内存本来就很有限。要在运行时查询可用的内存大小，请使用 [getMemoryClass()](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fapp%2FActivityManager.html%23getMemoryClass()) 或[getLargeMemoryClass()](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fapp%2FActivityManager.html%23getLargeMemoryClass()) 方法，后者的方法可以获取到应用开启 largeHeap 之后可以获得的内存大小。　　
 这里说到一点是，一个应用不应该通过这个属性来解决 OOM 问题，而是应该通过检测内存泄漏来彻底根治 OOM，而且当内存很大的时候，每次gc的时间也会长一些，性能也会随之下降。

###### android:label

这个属性用于设置应用程序整体的用户可读的标签，并也是每个应用程序组件的默认标签。对于<activity>、<activity-alias>、<service>、<receiver>和<provider>元素，请看它们各自的label属性。

设置这个属性值时，应该引用一个字符串资源。以便它能够跟用户界面中的其他字符串一样能够被本地化。但是为了应用程序开发的便利，也能够用原生的字符串来设置。

###### android:logo

这个属性用于给整个应用程序设置一个Logo，而且它也是所有Activity的默认Logo。

设置这个属性时，必须要引用一个包含图片的可绘制资源（如：“@drawable/logo”）。没有默认的Logo。

###### android:manageSpaceActivity

这个标识用来指定一个 Activity 的名字，当用户在设置页面中手动点击清除数据按钮时，不会像以前一样把应用的私有目录/data/data/包名下的数据完全清除，而是跳转到那个声明的 activity 中，让用户遵照 activity 中提供的功能清除指定的数据。　　
 感兴趣的可以看看该链接：[android:manageSpaceActivity让应用手动管理应用的数据目录](https://link.jianshu.com?t=http%3A%2F%2Ftangke.iteye.com%2Fblog%2F1817857)

###### android:name

这整个属性用完整的Java类名，给应用程序定义了一个Application子类的实现。当应用程序进程被启动时，这个类在其他任何应用程序组件被实例化之前实例化。

这个子类实现是可选的，大多数应用程序不需要一个子类的实现。如果没有实现自己的子类，Android系统会使用基本的Application类的一个实例。

###### android:permission

这个属性定义了一个权限，为了跟应用程序进行交互，客户端必须要有这个权限。这个属性是为给所有的应用程序组件设置权限提供了便利的方法。它能够被独立组件所设置的`permission`属性所覆盖。

###### android:persistent

这个属性用户设置应用程序是否应该时刻保持运行状态，如果设置为true，那么就保持，否则不保持。默认值是false。普通的应用程序不应该设置这个属性，持久运行模式仅用于某些系统级的应用程序。

###### android:process

这个属性用于定义一个进程名称，应用程序的所有组件都应该运行在这个进程中。每个组件都能够用它自己process属性的设置来覆盖这个<application>元素中的设置。

默认情况下，当应用程序的第一个组件需要运行时，Android系统就会给这个应用程序创建一个进程。然后，应用中的所有组件都运行在这个进程中。默认的进程名是跟<manifest>元素中设置的包名进行匹配的。

通过设置这个属性，能够跟另外一个应用程序共享一个进程名，能够把这两个应用程序中的组件都安排到同一个进程中运行---但是仅限于这两个应用程序共享一个用户ID，并且带有相同的数字证书。

如果这个进程名称用“：”开头，那么在需要的时候，就会给应用程序创建一个新的、私有的进程。如果进程名用小写字符开头，就会用这个名字创建一个全局的进程，这个全局的进程能够被其他应用程序共享，从而减少资源的使用。

###### android:restoreAnyVersion

设置这个属性表示应用程序准备尝试恢复任何备份的数据集，即使备份比设备上当前安装的应用程序的版本要新。这个属性设置为true，即使是在版本不匹配而产生数据兼容性提示的时候，也会允许备份管理来恢复备份的数据，所以要谨慎使用。

这个属性的默认值是`false`。

###### android:taskAffinity

这个属性给应用的所有的Activity设置了一个亲缘关系名，除了那些用它们自己的taskAffinity属性设置不同亲缘关系的组件。

默认情况下，应用程序中的所有Activity都会共享相同的亲缘关系，亲缘关系的名称跟由<manifest>元素设置的包名相同。

###### android:theme

这个属性给应用程序中所有的Activity设置默认的主题，属性值要引用一个样式资源。每个独立的Activity的主题会被它们自己的theme属性所覆盖。

###### android:uiOptions

这个属性设置了Activity的UI的额外选项。它必须是下表中的一个值：

- none
   默认设置，没有额外的UI选项。
- splitActionBarWhenNarrow
   在水平空间受到限制的时候，会在屏幕的底部添加一个用于显示ActionBar中操作项的栏，例如：在纵向的手持设备上。而不是在屏幕顶部的操作栏中显示少量的操作项。它会把操作栏分成上下两部分，顶部用于导航选择，底部用于操作项目。这样就会确保可用的合理空间不仅只是针对操作项目，而且还会在顶部给导航和标题留有空间。菜单项目不能被分开到两个栏中，它们要显示在一起。

------

##### 二、Android uses-feature 和 uses-permission的作用 关系和区别

关于手机权限的获取我们在mainfest中有两个常用的标签，users-feature 和 uses-permission。

###### 1、uses-feature

定义一个该app会用到的硬件或者软件功能。(android 系统提供的可以选择的功能列表参考：[Features Reference](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fmanifest%2Fuses-feature-element.html%23features-reference))。标签的目的是用来描述该app所依赖的硬件和软件的功能有哪些，并不负责向系统去请求权限。



```swift
<uses-feature
  android:name="string"
  android:required=["true" | "false"]
  android:glEsVersion="integer" />

  //属性含义
  android:name | app需要定义的功能的名称
  android:required | 为ture时表示该功能对于app来说是必须有的，如果某一设备不具备该功能，google play 商店将会对该设备隐藏该app；为false时表示该功能对于      app来说时非必需的，即使某一设备不具备该功能，google play商店仍然会对该设备显示该app
  android:glEsVersion | 指定openGL ES的版本号，只针对open GL功能
```

例子如下：



```xml
<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
<uses-feature android:name="android.hardware.camera.flash" />
```

uses-feature的作用更像是一个过滤器，google play 商店会根据该标签来过滤设备，比如用户在uses-feature中声明了要使用相机，这时候在google play商店中该app就不再对没有照相机的设备显示。但是，如果用户同时也设置了uses-feature的属性android:required 为false的话，google play商店仍然会对没有照相机的设备显示该app。

###### 主题 android:theme="@android:style/Theme.NoDisplay"





作者：黄海佳
链接：https://www.jianshu.com/p/f535c0f6f65f
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。