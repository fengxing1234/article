---
title: 使用AIDL实现一个多进程消息推送(3)
date: 2020-05-22 00:19:00
tags: 
	- Android
	- Binder
	- 多进程
categories: 
	- Android
	- Binder
---

像图片选择这样的多进程需求，可能并不需要我们额外编写进程通讯的代码，使用四大组件传输Bundle就行了，但是像推送服务这种需求，进程与进程之间需要高度的交互，此时就绕不过进程通讯这一步了。

下面我们就用即时聊天软件为例，手动去实现一个多进程的推送例子，具体需求如下：

1. UI和消息推送的Service分两个进程；
2. UI进程用于展示具体的消息数据，把用户发送的消息，传递到消息Service，然后发送到远程服务器；
3. Service负责收发消息，并和远程服务器保持长连接，UI进程可通过Service发送消息到远程服务器，Service收到远程服务器消息通知UI进程；
4. 即使UI进程退出了，Service仍需要保持运行，收取服务器消息。

**实现思路**

1. 创建UI进程（下文统称为客户端）；

2. 创建消息Service（下文统称为服务端）；

3. 把服务端配置到独立的进程(AndroidManifest.xml中指定process标签)；

4. 客户端和服务端进行绑定（bindService）；

5. 让客户端和服务端具备交互的能力。(AIDL使用)

# 例子具体实现

## AIDL调用流程概览

开始之前，我们先来概括一下使用AIDL进行多进程调用的整个流程：

1. 客户端使用bindService方法绑定服务端；
2. 服务端在onBind方法返回Binder对象；
3. 客户端拿到服务端返回的Binder对象进行跨进程方法调用；

![AIDL调用过程](http://jsh180.net/blog_aidl_img_flow_1.jpg)

**整个AIDL调用过程概括起来就以上3个步骤，下文中我们使用上面描述的例子，来逐步分解这些步骤，并讲述其中的细节。**

## 客户端使用bindService方法绑定服务端

### 创建客户端和服务端，把服务端配置到另外的进程

1. 创建客户端 -> MainActivity；
2. 创建服务端 -> MessageService;
3. 把服务端配置到另外的进程 -> android:process=”:remote”

上面描述的客户端、服务端、以及把服务端配置到另外进程，体现在AndroidManifest.xml中，如下所示：

```
<manifest ...>
    <application ...>
        <activity android:name=".test.test_aidl.TestAidlMessageActivity" />
        <service
            android:name=".test.test_aidl.MessageService"
            android:enabled="true"
            android:exported="true"
            android:process=":remote" />
    </application>
</manifest>
```

开启多进程的方法很简单，只需要给四大组件指定android:process标签。

### 绑定MessageService到MainActivity

**创建MessageService**

此时的MessageService就是刚创建的模样，onBind中返回了null，下一步中我们将返回一个可操作的对象给客户端。

```
package com.zhyen.android.test.test_aidl;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;

import androidx.annotation.Nullable;

public class MessageService extends Service {
    
    public MessageService() {
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

**客户端MainActivity调用bindService方法绑定MessageService**

这一步其实是属于Service组件相关的知识，在这里就比较简单地说一下，启动服务可以通过以下两种方式：

- 使用bindService方法 -> bindService(Intent service, ServiceConnection conn, int flags)；
- 使用startService方法 -> startService(Intent service);

**bindService & startService区别：**

使用bindService方式，多个Client可以同时bind一个Service，但是当所有Client unbind后，Service会退出，通常情况下，如果希望和Service交互，一般使用bindService方法，使用onServiceConnected中的IBinder对象可以和Service进行交互，不需要和Service交互的情况下，使用startService方法即可。

正如上面所说，我们是要和Service交互的，所以我们需要使用bindService方法，**但是我们希望unbind后Service仍保持运行，这样的情况下，可以同时调用bindService和startService**（比如像本例子中的消息服务，退出UI进程，Service仍需要接收到消息），代码如下：

```
package com.zhyen.android.test.test_aidl;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.util.Log;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.zhyen.android.R;

public class TestAidlMessageActivity extends AppCompatActivity {

    private static final String TAG = "TestAidlMessageActivity";

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.test_aidl_message_activity);
        setupService();
    }

    private void setupService() {
        Intent intent = new Intent(this, MessageService.class);
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
        startService(intent);
    }

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.d(TAG, "onServiceConnected: name = " + name);
            Log.d(TAG, "onServiceConnected: service = " + service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.d(TAG, "onServiceDisconnected: name = " + name);
        }
    };

    @Override
    protected void onDestroy() {
        unbindService(connection);
        super.onDestroy();
    }
}

```

## 服务端在onBind方法返回Binder对象

### 首先，什么是Binder?

要说Binder，首先要说一下IBinder这个接口，IBinder是远程对象的基础接口，轻量级的远程过程调用机制的核心部分，该接口描述了与远程对象交互的抽象协议，而Binder实现了IBinder接口，简单说，Binder就是Android SDK中内置的一个多进程通讯实现类，在使用的时候，我们不用也不要去实现IBinder，而是继承Binder这个类即可实现多进程通讯。

### 其次，这个需要在onBind方法返回的Binder对象从何而来？

**在这里就要引出本文中的主题了——AIDL**

多进程中使用的Binder对象，一般通过我们定义好的 .adil 接口文件自动生成，当然你可以走野路子，直接手动编写这个跨进程通讯所需的Binder类，其本质无非就是一个继承了Binder的类，鉴于野路子走起来麻烦，而且都是重复步骤的工作，Google提供了 AIDL 接口来帮我们自动生成Binder这条正路，下文中我们围绕 AIDL 这条正路继续展开讨论

### 定义AIDL接口

很明显，接下来我们需要搞一波上面说的Binder，让客户端可以调用到服务端的方法，而这个Binder又是通过AIDL接口自动生成，那我们就先从AIDL搞起，搞之前先看看注意事项，以免出事故：

AIDL支持的数据类型：

- Java 编程语言中的所有基本数据类型（如 int、long、char、boolean 等等）
- String和CharSequence
- Parcelable：实现了Parcelable接口的对象
- List：其中的元素需要被AIDL支持，另一端实际接收的具体类始终是 ArrayList，但生成的方法使用的是 List 接口
- Map：其中的元素需要被AIDL支持，包括 key 和 value，另一端实际接收的具体类始终是 HashMap，但生成的方法使用的是 Map 接口

其他注意事项：

- 在AIDL中传递的对象，必须实现Parcelable序列化接口；
- 在AIDL中传递的对象，需要在类文件相同路径下，创建同名、但是后缀为.aidl的文件，并在文件中使用parcelable关键字声明这个类；
- 跟普通接口的区别：只能声明方法，不能声明变量；
- 所有非基础数据类型参数都需要标出数据走向的方向标记。可以是 in、out 或 inout，基础数据类型默认只能是 in，不能是其他方向。

### 创建一个AIDL接口

接口中提供发送消息的方法（Android Studio创建AIDL：项目右键 -> New -> AIDL -> AIDL File），代码如下：

```
// MessageSender.aidl
package com.zhyen.android;
import com.zhyen.android.model.MessageModel;
// Declare any non-default types here with import statements

interface MessageSender {
    void sendMessage(in MessageModel messageModel);
}
```

> 被“in”标记的参数，就是接收实际数据的参数，这个跟我们普通参数传递一样的含义。在AIDL中，“out” 指定了一个仅用于输出的参数，换而言之，这个参数不关心调用方传递了什么数据过来，但是这个参数的值可以在方法被调用后填充（无论调用方传递了什么值过来，在方法执行的时候，这个参数的初始值总是空的），这就是“out”的含义，仅用于输出。而“inout”显然就是“in”和“out”的合体了，输入和输出的参数。区分“in”、“out”有什么用？这是非常重要的，因为每个参数的内容必须编组（序列化，传输，接收和反序列化）。in/out标签允许Binder跳过编组步骤以获得更好的性能。

上述的MessageModel为消息的实体类，该类在AIDL中传递，实现了Parcelable序列化接口，代码如下：

```
package com.zhyen.android.model;

import android.os.Parcel;
import android.os.Parcelable;

public class MessageModel implements Parcelable {

    private String from;
    private String to;
    private String content;

    public MessageModel() {

    }

    public MessageModel(String from, String to, String content) {
        this.from = from;
        this.to = to;
        this.content = content;
    }

    protected MessageModel(Parcel in) {
        from = in.readString();
        to = in.readString();
        content = in.readString();
    }

    public static final Creator<MessageModel> CREATOR = new Creator<MessageModel>() {
        @Override
        public MessageModel createFromParcel(Parcel in) {
            return new MessageModel(in);
        }

        @Override
        public MessageModel[] newArray(int size) {
            return new MessageModel[size];
        }
    };

    public String getFrom() {
        return from;
    }

    public void setFrom(String from) {
        this.from = from;
    }

    public String getTo() {
        return to;
    }

    public void setTo(String to) {
        this.to = to;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "MessageModel{" +
                "from='" + from + '\'' +
                ", to='" + to + '\'' +
                ", content='" + content + '\'' +
                '}';
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(from);
        dest.writeString(to);
        dest.writeString(content);
    }
}
```

创建完MessageModel这个实体类，别忘了还有一件事要做：”在AIDL中传递的对象，需要在类文件相同路径下，创建同名、但是后缀为.aidl的文件，并在文件中使用parcelable关键字声明这个类“。代码如下：

```
// MessageModel.aidl
package com.zhyen.android.model;

// Declare any non-default types here with import statements

parcelable MessageModel;

```

说明：

- MessageSender.aidl -> 定义了发送消息的方法，会自动生成名为MessageSender.Stub的Binder类，在服务端实现，返回给客户端调用
- MessageModel.java -> 消息实体类，由客户端传递到服务端，实现了Parcelable序列化
- MessageModel.aidl -> 声明了MessageModel可在AIDL中传递，放在跟MessageModel.java相同的包路径下

### 服务端MessageService

在服务端创建MessageSender.aidl这个AIDL接口自动生成的Binder对象，并返回给客户端调用，服务端MessageService代码如下：

```
package com.zhyen.android.test.test_aidl;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.RemoteException;
import android.util.Log;

import androidx.annotation.Nullable;

import com.zhyen.android.MessageSender;
import com.zhyen.android.model.MessageModel;

public class MessageService extends Service {

    private static final String TAG = "MessageService";

    public MessageService() {
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return stub;
    }

    private MessageSender.Stub stub = new MessageSender.Stub() {
        @Override
        public void sendMessage(MessageModel messageModel) throws RemoteException {
            Log.d(TAG, "sendMessage: " + messageModel);
            if (messageModel == null) return;
            Log.d(TAG, "sendMessage: " + messageModel.toString());
        }
    };
}
```

MessageSender.Stub是Android Studio根据我们MessageSender.aidl文件自动生成的Binder对象（至于是怎样生成的，下文会有答案），我们需要把这个Binder对象返回给客户端。

## 客户端拿到Binder对象后调用远程方法

调用步骤如下：

1. 在客户端的onServiceConnected方法中，拿到服务端返回的Binder对象；
2. 使用MessageSender.Stub.asInterface方法，取得MessageSender.aidl对应的操作接口；
3. 取得MessageSender对象后，像普通接口一样调用方法即可。

此时客户端代码如下：

```
package com.zhyen.android.test.test_aidl;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.os.RemoteException;
import android.util.Log;
import android.view.View;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.zhyen.android.MessageSender;
import com.zhyen.android.R;
import com.zhyen.android.model.MessageModel;

public class TestAidlMessageActivity extends AppCompatActivity {

    private static final String TAG = "TestAidlMessageActivity";
    private MessageSender messageSender;
    private boolean connected;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.test_aidl_message_activity);
        findViewById(R.id.btn_send_msg).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (!connected) return;
                //使用asInterface方法取得AIDL对应的操作接口
                MessageModel messageModel = new MessageModel();
                messageModel.setFrom("client user id");
                messageModel.setTo("receiver user id");
                messageModel.setContent("This is message content");
                //调用远程Service的sendMessage方法，并传递消息实体对象
                try {
                    messageSender.sendMessage(messageModel);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
        setupService();
    }

    private void setupService() {
        Intent intent = new Intent(this, MessageService.class);
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
        startService(intent);
    }

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            connected = true;
            Log.d(TAG, "onServiceConnected: name = " + name);
            Log.d(TAG, "onServiceConnected: service = " + service);
            messageSender = MessageSender.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            connected = false;
            Log.d(TAG, "onServiceDisconnected: name = " + name);
        }
    };

    @Override
    protected void onDestroy() {
        unbindService(connection);
        super.onDestroy();
    }
}
```

在客户端中我们调用了MessageSender的sendMessage方法，向服务端发送了一条消息，并把生成的MessageModel对象作为参数传递到了服务端，最终服务端打印的结果如下：

```
2020-05-22 10:53:25.747 22509-22545/com.zhyen.android:remote D/MessageService: sendMessage: MessageModel{from='client user id', to='receiver user id', content='This is message content'}

```

这里有两点要说：

- 服务端已经接收到客户端发送过来的消息，并正确打印；
- 服务端和客户端区分两个进程，PID不一样，进程名也不一样；

# 分析Binder上层机制

我们通过上述的调用流程，看看从客户端到服务端，都经历了些什么事，看看Binder的上层是如何工作的，至于Binder的底层，这是一个非常复杂的话题，本文不深究。（如果看到这里你又想问什么是Binder的话，请手动倒带往上看…）

我们先来回顾一下从客户端发起的调用流程：

- MessageSender messageSender = MessageSender.Stub.asInterface(service);
- messageSender.sendMessage(messageModel);

抛开其它无关代码，客户端调跨进程方法就这两个步骤，而这两个步骤都封装在 MessageSender.aidl 最终生成的 MessageSender.java 源码。

```
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.zhyen.android;
// Declare any non-default types here with import statements

public interface MessageSender extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.zhyen.android.MessageSender {
		    //描述符，该值为全类名
        private static final java.lang.String DESCRIPTOR = "com.zhyen.android.MessageSender";

        /**
         * Construct the stub at attach it to the interface.
         根据AIDL的使用流程，Server会在onBind的时候返回一个Stub实例，
         调用了Stub的构造器内部调用Binder的attachInterface方法将当前实例以及描述符存到Binder实例中
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * 把IBinder对象转换为MessageSender 接口
         * 判断IBinder是否处于相同进程，相同进程返回Stub实现的MessageSender接口
         * 不同进程，则返回Stub.Proxy实现的MessageSender接口
         */
        public static com.zhyen.android.MessageSender asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.zhyen.android.MessageSender))) {
                return ((com.zhyen.android.MessageSender) iin);
            }
            return new com.zhyen.android.MessageSender.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        /**
         * 同一进程时，不会触发
         *
         * 不同进程时，asInterface会返回Stub.Proxy，客户端调用 messageSender.sendMessage(messageModel)
         * 实质是调用了 Stub.Proxy 的 sendMessage 方法，从而触发跨进程数据传递，
         * 最终Binder底层将处理好的数据回调到此方法，并调用我们真正的sendMessage方法
         */
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_sendMessage: {
                    data.enforceInterface(descriptor);
                    com.zhyen.android.model.MessageModel _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = com.zhyen.android.model.MessageModel.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    this.sendMessage(_arg0);
                    reply.writeNoException();
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements com.zhyen.android.MessageSender {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            /**
             * Proxy中的sendMessage方法，并不是直接调用我们定义的sendMessage方法，而是经过一顿的Parcel读写，
             * 然后调用mRemote.transact方法，把数据交给Binder处理，transact处理完毕后会调用上方的onTransact方法，
             * onTransact拿到最终得到的参数数据，调用由我们真正的sendMessage方法
             */
            @Override
            public void sendMessage(com.zhyen.android.model.MessageModel messageModel) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((messageModel != null)) {
                        _data.writeInt(1);
                        messageModel.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    //调用Binder的transact方法进行多进程数据传输，处理完毕后调用上方的onTransact方法
                    mRemote.transact(Stub.TRANSACTION_sendMessage, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }

        static final int TRANSACTION_sendMessage = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    }

    public void sendMessage(com.zhyen.android.model.MessageModel messageModel) throws android.os.RemoteException;
}

```

只看代码的话，可能会有点懵逼，相信结合代码再看下方的流程图会更好理解：

![flow chart](http://jsh180.net/image/jpg/blog_aidl_flowchart.jpg)

从客户端的sendMessage开始，整个AIDL的调用过程如上图所示，asInterface方法，将会判断onBind方法返回的Binder是否存处于同一进程，在同一进程中，则进行常规的方法调用，若处于不同进程，整个数据传递的过程则需要通过Binder底层去进行编组（序列化，传输，接收和反序列化），得到最终的数据后再进行常规的方法调用。

**对象跨进程传输的本质就是 序列化，传输，接收和反序列化 这样一个过程，这也是为什么跨进程传输的对象必须实现Parcelable接口**

# 跨进程的回调接口

在上面我们已经实现了从客户端发送消息到跨进程服务端的功能，接下来我们还需要将服务端接收到的远程服务器消息，传递到客户端。有同学估计会说：“这不就是一个回调接口的事情嘛”，设置回调接口思路是对的，但是在这里使用的回调接口有点不一样，在AIDL中传递的接口，不能是普通的接口，只能是AIDL接口，所以我们需要新建一个AIDL接口传到服务端，作为回调接口。

新建消息收取的AIDL接口MessageReceiver.aidl：

```
// MessageReceiver.aidl
package com.zhyen.android;
import com.zhyen.android.model.MessageModel;
// Declare any non-default types here with import statements
//消息回调接口
interface MessageReceiver {
    void onMessageReceived(in MessageModel receivedMessage);
}

```

接下来我们把回调接口注册到服务端去，修改我们的MessageSender.aidl:

```
// MessageSender.aidl
package com.zhyen.android;
import com.zhyen.android.model.MessageModel;
import com.zhyen.android.MessageReceiver;
// Declare any non-default types here with import statements

interface MessageSender {
    void sendMessage(in MessageModel messageModel);

    void registerReceiveListener(MessageReceiver messageReceiver);

    void unregisterReceiveListener(MessageReceiver messageReceiver);
}
```

以上就是我们最终修改好的aidl接口，接下来我们需要做出对应的变更：

- 在服务端中增加MessageSender的注册和反注册接口的方法；

- 在客户端中实现MessageReceiver接口，并通过MessageSender注册到服务端。

客户端变更，修改MainActivity：

```
package com.zhyen.android.test.test_aidl;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.os.RemoteException;
import android.util.Log;
import android.view.View;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.zhyen.android.MessageReceiver;
import com.zhyen.android.MessageSender;
import com.zhyen.android.R;
import com.zhyen.android.model.MessageModel;

public class TestAidlMessageActivity extends AppCompatActivity {

    private static final String TAG = "TestAidlMessageActivity";
    private MessageSender messageSender;
    private boolean connected;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.test_aidl_message_activity);
        findViewById(R.id.btn_send_msg).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (!connected) return;
                //使用asInterface方法取得AIDL对应的操作接口
                MessageModel messageModel = new MessageModel();
                messageModel.setFrom("client user id");
                messageModel.setTo("receiver user id");
                messageModel.setContent("This is message content");
                //调用远程Service的sendMessage方法，并传递消息实体对象
                try {
                    messageSender.sendMessage(messageModel);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
        findViewById(R.id.btn_register).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (connected) return;
                try {
                    messageSender.registerReceiveListener(messageReceiver);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
        setupService();
    }

    private void setupService() {
        Intent intent = new Intent(this, MessageService.class);
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
        startService(intent);
    }

    //消息监听回调接口
    private MessageReceiver.Stub messageReceiver = new MessageReceiver.Stub() {
        @Override
        public void onMessageReceived(MessageModel receivedMessage) throws RemoteException {
            Log.d(TAG, "onMessageReceived: " + receivedMessage.toString());
        }
    };

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            connected = true;
            Log.d(TAG, "onServiceConnected: name = " + name);
            Log.d(TAG, "onServiceConnected: service = " + service);
            messageSender = MessageSender.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            connected = false;
            Log.d(TAG, "onServiceDisconnected: name = " + name);
        }
    };

    @Override
    protected void onDestroy() {
        //解除消息监听接口
        if (messageSender != null && messageSender.asBinder().isBinderAlive()) {
            try {
                messageSender.unregisterReceiveListener(messageReceiver);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        unbindService(connection);
        super.onDestroy();
    }
}

```

客户端主要有3个变更：

1. 增加了messageReceiver对象，用于监听服务端的消息通知；
2. onServiceConnected方法中，把messageReceiver注册到Service中去；
3. onDestroy时候解除messageReceiver的注册。

**下面对服务端MessageServie进行变更：**

```
package com.zhyen.android.test.test_aidl;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.RemoteCallbackList;
import android.os.RemoteException;
import android.util.Log;

import androidx.annotation.Nullable;

import com.zhyen.android.MessageReceiver;
import com.zhyen.android.MessageSender;
import com.zhyen.android.model.MessageModel;

import java.util.concurrent.atomic.AtomicBoolean;

public class MessageService extends Service {

    private static final String TAG = "MessageService";
    private AtomicBoolean serviceStop = new AtomicBoolean();
    //RemoteCallbackList专门用来管理多进程回调接口
    private RemoteCallbackList<MessageReceiver> listenerList = new RemoteCallbackList<>();

    public MessageService() {
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return stub;
    }

    private MessageSender.Stub stub = new MessageSender.Stub() {
        @Override
        public void sendMessage(MessageModel messageModel) throws RemoteException {
            Log.d(TAG, "sendMessage: " + messageModel);
            if (messageModel == null) return;
            Log.d(TAG, "sendMessage: " + messageModel.toString());
        }

        @Override
        public void registerReceiveListener(MessageReceiver messageReceiver) throws RemoteException {
            listenerList.register(messageReceiver);
        }

        @Override
        public void unregisterReceiveListener(MessageReceiver messageReceiver) throws RemoteException {
            listenerList.unregister(messageReceiver);
        }
    };

    @Override
    public void onCreate() {
        super.onCreate();
        new Thread(new FakeTCPTask()).start();
    }

    @Override
    public void onDestroy() {
        serviceStop.set(true);
        super.onDestroy();
    }

    //模拟长连接，通知客户端有新消息到达
    private class FakeTCPTask implements Runnable {
        @Override
        public void run() {
            while (!serviceStop.get()) {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                MessageModel messageModel = new MessageModel();
                messageModel.setFrom("Service");
                messageModel.setTo("Client");
                messageModel.setContent(String.valueOf(System.currentTimeMillis()));
                /**
                 * RemoteCallbackList的遍历方式
                 * beginBroadcast和finishBroadcast一定要配对使用
                 */
                final int listenerCount = listenerList.beginBroadcast();
                Log.d(TAG, "listenerCount == " + listenerCount);
                for (int i = 0; i < listenerCount; i++) {
                    MessageReceiver messageReceiver = listenerList.getBroadcastItem(i);
                    if (messageReceiver != null) {
                        try {
                            messageReceiver.onMessageReceived(messageModel);
                        } catch (RemoteException e) {
                            e.printStackTrace();
                        }
                    }
                }
                listenerList.finishBroadcast();
            }
        }
    }
}

```

服务端主要变更：

1. MessageSender.Stub实现了注册和反注册回调接口的方法；
2. 增加了RemoteCallbackList来管理AIDL远程接口；
3. FakeTCPTask模拟了长连接通知客户端有新消息到达。

这里还有一个需要讲一下的，就是RemoteCallbackList，为什么要用RemoteCallbackList，普通ArrayList不行吗？当然不行，不然干嘛又整一个RemoteCallbackList 🙃，registerReceiveListener 和 unregisterReceiveListener在客户端传输过来的对象，经过Binder处理，在服务端接收到的时候其实是一个新的对象，这样导致在 unregisterReceiveListener 的时候，普通的ArrayList是无法找到在 registerReceiveListener 时候添加到List的那个对象的，但是它们底层使用的Binder对象是同一个，RemoteCallbackList利用这个特性做到了可以找到同一个对象，这样我们就可以顺利反注册客户端传递过来的接口对象了。RemoteCallbackList在客户端进程终止后，它能自动移除客户端所注册的listener，它内部还实现了线程同步，所以我们在注册和反注册都不需要考虑线程同步，的确是个666的类。（至于使用ArrayList的幺蛾子现象，大家可以自己试试，篇幅问题，这里就不演示了）

## 结果

客户端结果

```
2020-05-22 12:15:07.405 31763-31868/com.zhyen.android D/TestAidlMessageActivity: onMessageReceived: MessageModel{from='Service', to='Client', content='1590120907403'}
2020-05-22 12:15:10.408 31763-31868/com.zhyen.android D/TestAidlMessageActivity: onMessageReceived: MessageModel{from='Service', to='Client', content='1590120910406'}
2020-05-22 12:15:13.409 31763-31868/com.zhyen.android D/TestAidlMessageActivity: onMessageReceived: MessageModel{from='Service', to='Client', content='1590120913408'}
2020-05-22 12:15:16.411 31763-31868/com.zhyen.android D/TestAidlMessageActivity: onMessageReceived: MessageModel{from='Service', to='Client', content='1590120916410'}
2020-05-22 12:15:19.413 31763-31868/com.zhyen.android D/TestAidlMessageActivity: onMessageReceived: MessageModel{from='Service', to='Client', content='1590120919412'}
2020-05-22 12:15:22.415 31763-32268/com.zhyen.android D/TestAidlMessageActivity: onMessageReceived: MessageModel{from='Service', to='Client', content='1590120922414'}
2020-05-22 12:15:25.419 31763-32268/com.zhyen.android D/TestAidlMessageActivity: onMessageReceived: MessageModel{from='Service', to='Client', content='1590120925417'}
2020-05-22 12:15:28.422 31763-32268/com.zhyen.android D/TestAidlMessageActivity: onMessageReceived: MessageModel{from='Service', to='Client', content='1590120928420'}

```

服务端结果

```
2020-05-22 12:14:37.395 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:14:39.397 32152-32152/com.zhyen.android:remote D/AwareBitmapCacher: handleInit disable com.zhyen.android:remote
2020-05-22 12:14:40.395 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:14:43.397 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:14:46.398 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:14:49.399 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:14:52.400 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:14:55.400 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:14:56.750 32152-32191/com.zhyen.android:remote D/MessageService: sendMessage: MessageModel{from='client user id', to='receiver user id', content='This is message content'}
2020-05-22 12:14:56.751 32152-32191/com.zhyen.android:remote D/MessageService: sendMessage: MessageModel{from='client user id', to='receiver user id', content='This is message content'}
2020-05-22 12:14:58.401 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:15:01.402 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:15:04.402 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 0
2020-05-22 12:15:07.403 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 1
2020-05-22 12:15:10.407 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 1
2020-05-22 12:15:13.408 32152-32192/com.zhyen.android:remote D/MessageService: listenerCount == 1
```

# DeathRecipient

不知道你有没有感觉到，两个进程交互总是觉得缺乏那么一点安全感…比如说服务端进程Crash了，而客户端进程想要调用服务端方法，这样就调用不到了。此时我们可以给Binder设置一个DeathRecipient对象，当Binder意外挂了的时候，我们可以在DeathRecipient接口的回调方法中收到通知，并作出相应的操作，比如重连服务等等。

DeathRecipient的使用如下：

- 声明DeathRecipient对象，实现其binderDied方法，当binder死亡时，会回调binderDied方法；
- 给Binder对象设置DeathRecipient对象。

在客户端MainActivity声明DeathRecipient：

```
package com.zhyen.android.test.test_aidl;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Binder;
import android.os.Bundle;
import android.os.IBinder;
import android.os.RemoteException;
import android.util.Log;
import android.view.View;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.zhyen.android.MessageReceiver;
import com.zhyen.android.MessageSender;
import com.zhyen.android.R;
import com.zhyen.android.model.MessageModel;

public class TestAidlMessageActivity extends AppCompatActivity {

    private static final String TAG = "TestAidlMessageActivity";
    private MessageSender messageSender;
    private boolean connected;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.test_aidl_message_activity);
        findViewById(R.id.btn_send_msg).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (!connected) return;
                //使用asInterface方法取得AIDL对应的操作接口
                MessageModel messageModel = new MessageModel();
                messageModel.setFrom("client user id");
                messageModel.setTo("receiver user id");
                messageModel.setContent("This is message content");
                //调用远程Service的sendMessage方法，并传递消息实体对象
                try {
                    messageSender.sendMessage(messageModel);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
        findViewById(R.id.btn_register).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (!connected) return;
                try {
                    messageSender.registerReceiveListener(messageReceiver);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
        setupService();
    }

    private void setupService() {
        Intent intent = new Intent(this, MessageService.class);
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
        startService(intent);
    }

    /**
     * Binder可能会意外死忙（比如Service Crash），Client监听到Binder死忙后可以进行重连服务等操作
     */
    private Binder.DeathRecipient deathRecipient = new IBinder.DeathRecipient() {
        @Override
        public void binderDied() {
            Log.d(TAG, "binderDied: ");
            if (messageSender != null) {
                messageSender.asBinder().unlinkToDeath(this, 0);
                messageSender = null;
            }
            //重连服务或其他操作
            setupService();
        }
    };

    //消息监听回调接口
    private MessageReceiver.Stub messageReceiver = new MessageReceiver.Stub() {
        @Override
        public void onMessageReceived(MessageModel receivedMessage) throws RemoteException {
            Log.d(TAG, "onMessageReceived: " + receivedMessage.toString());
        }
    };

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            connected = true;
            Log.d(TAG, "onServiceConnected: name = " + name);
            Log.d(TAG, "onServiceConnected: service = " + service);
            messageSender = MessageSender.Stub.asInterface(service);
            //设置Binder死亡监听
            try {
                messageReceiver.asBinder().linkToDeath(deathRecipient, 0);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            connected = false;
            Log.d(TAG, "onServiceDisconnected: name = " + name);
        }
    };

    @Override
    protected void onDestroy() {
        //解除消息监听接口
        if (messageSender != null && messageSender.asBinder().isBinderAlive()) {
            try {
                messageSender.unregisterReceiveListener(messageReceiver);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        unbindService(connection);
        super.onDestroy();
    }
}
```

Binder中两个重要方法：

1. linkToDeath -> 设置死亡代理 DeathRecipient 对象；
2. unlinkToDeath -> Binder死亡的情况下，解除该代理。

此外，Binder中的isBinderAlive也可以判断Binder是否死亡。

# 权限验证

就算是公交车，上车也得嘀卡对不，如果希望我们的服务进程不想像公交车一样谁想上就上，那么我们可以加入权限验证。

介绍两种常用验证方法：

- 在服务端的onBind中校验自定义permission，如果通过了我们的校验，正常返回Binder对象，校验不通过返回null，返回null的情况下客户端无法绑定到我们的服务；
- 在服务端的onTransact方法校验客户端包名，不通过校验直接return false，校验通过执行正常的流程。

自定义permission，在Androidmanifest.xml中增加自定义的权限：

```
<permission
        android:name="com.zhyen.android.permission.REMOTE_SERVICE_PERMISSION"
        android:protectionLevel="normal" />

    <uses-permission android:name="com.zhyen.android.permission.REMOTE_SERVICE_PERMISSION" />
```

服务端检查权限的方法：

- 校验权限的方式

```
@Nullable
    @Override
    public IBinder onBind(Intent intent) {
        //自定义permission方式检查权限
        if (checkCallingOrSelfPermission("com.zhyen.android.permission.REMOTE_SERVICE_PERMISSION") == PackageManager.PERMISSION_DENIED) {
            Log.d(TAG, "onBind: 没有设置权限");
            return null;
        }
        return stub;
    }
```

- 判断包名

```
private MessageSender.Stub stub = new MessageSender.Stub() {
        @Override
        public void sendMessage(MessageModel messageModel) throws RemoteException {
            Log.d(TAG, "sendMessage: " + messageModel);
            if (messageModel == null) return;
            Log.d(TAG, "sendMessage: " + messageModel.toString());
        }

        @Override
        public void registerReceiveListener(MessageReceiver messageReceiver) throws RemoteException {
            listenerList.register(messageReceiver);
        }

        @Override
        public void unregisterReceiveListener(MessageReceiver messageReceiver) throws RemoteException {
            listenerList.unregister(messageReceiver);
        }

        @Override
        public boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
            /**
             * 包名验证方式
             */
            String packageName = null;
            String[] packages = getPackageManager().getPackagesForUid(getCallingUid());
            if (packages != null && packages.length > 0) {
                packageName = packages[0];
            }
            if (packageName == null || !packageName.startsWith("com.zhyen.android")) {
                Log.d("onTransact", "拒绝调用：" + packageName);
                return false;
            }
            return super.onTransact(code, data, reply, flags);
        }
    };
```

# 根据不同进程，做不同的初始化工作

相信前一两年很多朋友还在使用Android-Universal-Image-Loader来加载图片，它是需要在Application类进行初始化的。打个比如，我们用它来加载图片，而且还有一个图片选择进程，那么我们希望分配更多的缓存给图片选择进程，又或者是一些其他的初始化工作，不需要在图片选择进程初始化怎么办？

这里提供一个简单粗暴的方法，博主也是这么干的…直接拿到进程名判断，作出相应操作即可：

```
package com.zhyen.android;

import android.app.ActivityManager;
import android.app.Application;
import android.content.Context;
import android.nfc.Tag;
import android.os.Process;
import android.util.Log;

import java.util.List;

public class ZhyenApplication extends Application {
    private static final String TAG = "ZhyenApplication";

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, getProcessName(getApplicationContext()));
    }

    //取得进程名
    public static String getProcessName(Context context) {
        ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
        List<ActivityManager.RunningAppProcessInfo> runningApps = am.getRunningAppProcesses();
        if (runningApps == null) {
            return null;
        }
        for (ActivityManager.RunningAppProcessInfo procInfo : runningApps) {
            if (procInfo.pid == Process.myPid()) {
                return procInfo.processName;
            }
        }
        return null;
    }
}
```

**每个进程创建，都会调用Application的onCreate方法，这是一个需要注意的地方，我们也可以根据当前进程的pid，拿到当前进程的名字去做判断，然后做一些我们需要的逻辑，我们这个例子，拿到的两个进程名分别是：**

 ```
2020-05-22 13:54:17.090 16347-16347/com.zhyen.android:remote D/ZhyenApplication: com.zhyen.android:remote

2020-05-22 13:54:08.698 16280-16280/? D/ZhyenApplication: com.zhyen.android
 ```