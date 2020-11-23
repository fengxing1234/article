---
title: 如何使用Aidl
date: 2020-05-21 11:58:35
tags: 
	- Android
	- Binder
	- AIDL
categories: 
	- Android
	- Binder
---

# Aidl是什么

它全称是Android Interface Definition Language，即Android接口定义语言，为了使其他的进程也可以访问本进程提供的服务，Android使用AIDL来公开服务的接口，它里面定义了本进程可以为其他进程提供什么服务，即定义了一些方法，其他进程就可以通过RPC（远程调用）来调用这些方法，从而获得服务，其中提供服务的进程称为服务端，获取服务的进程称为客户端。

![binder_interprocess_communication](http://gityuan.com/images/binder/prepare/binder_interprocess_communication.png)

# 语法

AIDL的语法十分简单，与Java语言基本保持一致，需要记住的规则有以下几点：

1. AIDL文件以 **.aidl** 为后缀名
2. AIDL支持的数据类型分为如下几种： 
   - 八种基本数据类型：byte、char、short、int、long、float、double、boolean
   - String，CharSequence
   - 实现了Parcelable接口的数据类型
   - List 类型。List承载的数据必须是AIDL支持的类型，或者是其它声明的AIDL对象
   - Map类型。Map承载的数据必须是AIDL支持的类型，或者是其它声明的AIDL对象
3. AIDL文件可以分为两类。一类用来声明实现了Parcelable接口的数据类型，以供其他AIDL文件使用那些非默认支持的数据类型。还有一类是用来定义接口方法，声明要暴露哪些接口给客户端调用，定向Tag就是用来标注这些方法的参数值
4. 定向Tag。定向Tag表示在跨进程通信中数据的流向，用于标注方法的参数值，分为 in、out、inout 三种。其中 in 表示数据只能由客户端流向服务端， out 表示数据只能由服务端流向客户端，而 inout 则表示数据可在服务端与客户端之间双向流通。此外，如果AIDL方法接口的参数值类型是：基本数据类型、String、CharSequence或者其他AIDL文件定义的方法接口，那么这些参数值的定向 Tag 默认是且只能是 in，所以除了这些类型外，其他参数值都需要明确标注使用哪种定向Tag。
5. 明确导包。在AIDL文件中需要明确标明引用到的数据类型所在的包名，即使两个文件处在同个包名下。

# 案例

客户端绑定服务端的service调用服务端的方法，实现应用数据共享

## 服务端

创建aidl接口定义外暴露方法，以及需要的传递的实体。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-5aeb537d22344755.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对外接口

```
// TestAidl.aidl
package com.zhyen.test;
import com.zhyen.test.Book;
// Declare any non-default types here with import statements

interface TestAidl {
       List<Book> getBookList();

       void addBookInOut(inout Book book);
}
```

将Book改为声明Parcelable数据类型的AIDL文件

```
// Book.aidl
package com.zhyen.test;

// Declare any non-default types here with import statements

parcelable Book;
```

java类型实体类，注意包的结构相同。此处都是`com.zhyen.test`

自定义数据类型，用于进程间通信的话，必须实现Parcelable接口，Parcelable是类似于Java中的Serializable，Android中定义了Parcelable，用于进程间数据传递，对传输数据进行分解，编组的工作，相对于Serializable，他对于进程间通信更加高效。

```
package com.zhyen.test;

import android.os.Parcel;
import android.os.Parcelable;

public class Book implements Parcelable {
    private String name;

    public Book(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "book name：" + name;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(this.name);
    }

    public void readFromParcel(Parcel dest) {
        name = dest.readString();
    }

    protected Book(Parcel in) {
        this.name = in.readString();
    }

    public static final Creator<Book> CREATOR = new Creator<Book>() {
        @Override
        public Book createFromParcel(Parcel source) {
            return new Book(source);
        }

        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };
}
```

编写服务端service

```
package com.zhyen.test;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.RemoteException;
import android.util.Log;

import androidx.annotation.Nullable;

import java.util.ArrayList;
import java.util.List;

public class AidlBookService extends Service {

    private static final String TAG = "AidlService";
    private List<Book> books;

    @Override
    public void onCreate() {
        super.onCreate();
        books = new ArrayList<>();
        Book book1 = new Book("活着");
        Book book2 = new Book("或者");
        Book book3 = new Book("叶应是叶");
        Book book4 = new Book("https://github.com/leavesC");
        Book book5 = new Book("http://www.jianshu.com/u/9df45b87cfdf");
        Book book6 = new Book("http://blog.csdn.net/new_one_object");
        books.add(book1);
        books.add(book2);
        books.add(book3);
        books.add(book4);
        books.add(book5);
        books.add(book6);
    }

    private TestAidl.Stub stub = new TestAidl.Stub() {
        @Override
        public List<Book> getBookList() throws RemoteException {
            Log.d(TAG, "getBookList: ");
            return books;
        }

        @Override
        public void addBookInOut(Book book) throws RemoteException {
            Log.d(TAG, "addBookInOut: " + book);
            books.add(book);
        }
    };

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return stub;
    }
}

```

service配置文件

```
<service
            android:name=".AidlBookService"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.zhyen.test.action" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </service>
```

服务端已经做好了

## 客户端

![image.png](https://upload-images.jianshu.io/upload_images/4118241-f2595f3330c66dd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



客户端连带着aidl目录全都复制到main目录下，负责java类Book，注意包的结构。

编写客户端代码

```
package com.zhyen.android.test;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.os.RemoteException;
import android.util.Log;
import android.view.View;
import android.widget.Button;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.zhyen.android.R;
import com.zhyen.base.IMyService;
import com.zhyen.base.Student;
import com.zhyen.test.Book;
import com.zhyen.test.TestAidl;

import java.util.List;

public class TestAIDLActivity extends AppCompatActivity {


    private static final String TAG = "TestAIDLActivity";
    private boolean bookConnected;

  
    private ServiceConnection connectionBook = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.d(TAG, "onServiceConnected: " + name);
            bookConnected = true;
            testAidl = TestAidl.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            bookConnected = false;
            Log.d(TAG, "onServiceDisconnected: " + name);
        }
    };
    private TestAidl testAidl;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.test_aidl_activity);
        findViewById(R.id.btn_add_book).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                try {
                    if (!bookConnected) return;
                    testAidl.addBookInOut(new Book("呵呵哒"));
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
        findViewById(R.id.btn_book_list).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (!bookConnected) return;
                try {
                    List<Book> bookList = testAidl.getBookList();
                    for (Book book : bookList) {
                        Log.d(TAG, "onClick: " + book.getName());
                    }
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
        bindServiceBook();
    }

    private void bindServiceBook() {
        Intent intent = new Intent();
        intent.setPackage("com.zhyen.test");
        intent.setAction("com.zhyen.test.action");
        bindService(intent, connectionBook, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(connectionBook);
    }
}

```

结果

```
2020-05-21 18:33:13.678 31931-31931/com.zhyen.android D/TestAIDLActivity: onClick: 活着
2020-05-21 18:33:13.678 31931-31931/com.zhyen.android D/TestAIDLActivity: onClick: 或者
2020-05-21 18:33:13.678 31931-31931/com.zhyen.android D/TestAIDLActivity: onClick: 叶应是叶
2020-05-21 18:33:13.678 31931-31931/com.zhyen.android D/TestAIDLActivity: onClick: https://github.com/leavesC
2020-05-21 18:33:13.678 31931-31931/com.zhyen.android D/TestAIDLActivity: onClick: http://www.jianshu.com/u/9df45b87cfdf
2020-05-21 18:33:13.678 31931-31931/com.zhyen.android D/TestAIDLActivity: onClick: http://blog.csdn.net/new_one_object
2020-05-21 18:33:13.678 31931-31931/com.zhyen.android D/TestAIDLActivity: onClick: 呵呵哒
2020-05-21 18:33:13.679 31931-31931/com.zhyen.android I/chatty: uid=10206(com.zhyen.android) identical 6 lines
2020-05-21 18:33:13.679 31931-31931/com.zhyen.android D/TestAIDLActivity: onClick: 呵呵哒

```

## aidl编译后的样子

```
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.zhyen.test;
// Declare any non-default types here with import statements

public interface TestAidl extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.zhyen.test.TestAidl {
        private static final java.lang.String DESCRIPTOR = "com.zhyen.test.TestAidl";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.zhyen.test.TestAidl interface,
         * generating a proxy if needed.
         */
        public static com.zhyen.test.TestAidl asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.zhyen.test.TestAidl))) {
                return ((com.zhyen.test.TestAidl) iin);
            }
            return new com.zhyen.test.TestAidl.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_getBookList: {
                    data.enforceInterface(descriptor);
                    java.util.List<com.zhyen.test.Book> _result = this.getBookList();
                    reply.writeNoException();
                    reply.writeTypedList(_result);
                    return true;
                }
                case TRANSACTION_addBookInOut: {
                    data.enforceInterface(descriptor);
                    com.zhyen.test.Book _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = com.zhyen.test.Book.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    this.addBookInOut(_arg0);
                    reply.writeNoException();
                    if ((_arg0 != null)) {
                        reply.writeInt(1);
                        _arg0.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements com.zhyen.test.TestAidl {
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

            @Override
            public java.util.List<com.zhyen.test.Book> getBookList() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.util.List<com.zhyen.test.Book> _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.createTypedArrayList(com.zhyen.test.Book.CREATOR);
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            @Override
            public void addBookInOut(com.zhyen.test.Book book) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((book != null)) {
                        _data.writeInt(1);
                        book.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_addBookInOut, _data, _reply, 0);
                    _reply.readException();
                    if ((0 != _reply.readInt())) {
                        book.readFromParcel(_reply);
                    }
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }

        static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_addBookInOut = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }

    public java.util.List<com.zhyen.test.Book> getBookList() throws android.os.RemoteException;

    public void addBookInOut(com.zhyen.test.Book book) throws android.os.RemoteException;
}
```

![aidl image](http://gityuan.com/images/binder/AIDL/MyServer_java_binder.jpg)