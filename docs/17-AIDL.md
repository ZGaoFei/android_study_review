##### AIDL

[Binder](https://blog.csdn.net/universus/article/details/6211589)

```
1、管道pipe：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。

2、命名管道FIFO：有名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信。

3、消息队列MessageQueue：消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。

4、共享存储SharedMemory：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号量，配合使用，来实现进程间的同步和通信。

5、信号量Semaphore：信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。

6、套接字Socket：套解字也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同及其间的进程通信。

7、信号 ( sinal ) ： 信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。

8、Binder：使用Client-Server通信方式，Binder是一个实体位于Server中的对象，该对象提供了一套方法用以实现对服务的请求，就像类的成员函数。遍布于client中的入口可以看成指向这个binder对象的引用，一旦获得了这个引用就可以调用该对象的方法访问server。在Client看来，通过Binder引用调用其提供的方法和通过引用调用其它任何本地对象的方法并无区别，尽管前者的实体位于远端Server中，而后者实体位于本地内存中。
Binder的四个角色：
（1）Server：Server提供一套接口函数以便Client通过远程访问使用各种服务。通常采用Proxy设计模式：将接口函数定义在一个抽象类中，Server和Client都会以该抽象类为基类实现所有接口函数，所不同的是Server端是真正的功能实现，而Client端是对这些函数远程调用请求的包装。
（2）Client：
（3）ServiceManager：作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。Server创建了Binder实体，为其取一个字符形式，可读易记的名字，将这个Binder连同名字以数据包的形式通过Binder驱动发送给SMgr，通知SMgr注册一个名叫张三的Binder，它位于某个Server中。驱动为这个穿过进程边界的Binder创建位于内核中的实体节点以及SMgr对实体的引用，将名字及新建的引用打包传递给SMgr。SMgr收数据包后，从中取出名字和引用填入一张查找表中。
（4）Binder驱动：驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。
SMgr和其它进程同样采用Binder通信：SMgr是Server端，有自己的Binder对象（实体），其它进程都是Client，需要通过这个Binder的引用来实现Binder的注册，查询和获取。SMgr提供的Binder比较特殊，它没有名字也不需要注册，当一个进程使用BINDER_SET_CONTEXT_MGR命令将自己注册成SMgr时Binder驱动会自动为它创建Binder实体（这就是那只预先造好的鸡）。其次这个Binder的引用在所有Client中都固定为0而无须通过其它手段获得。也就是说，一个Server若要向SMgr注册自己Binder就必需通过0这个引用号和SMgr的Binder通信。


Socket：socket作为一款通用接口，其传输效率低，开销大，主要用在跨网络的进程间通信和本机上进程间的低速通信。

共享内存：无需拷贝，但控制复杂，难以使用。

管道：
消息队列：消息队列和管道采用存储-转发方式，即数据先从发送方缓存区拷贝到内核开辟的缓存区中，然后再从内核缓存区拷贝到接收方缓存区，至少有两次拷贝过程。

binder：Android需要建立一套新的IPC机制来满足系统对通信方式，传输性能和安全性的要求，这就是Binder。Binder基于Client-Server通信模式，传输过程只需一次拷贝，为发送方添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。

传统IPC没有任何安全措施，完全依赖上层协议来确保。首先传统IPC的接收方无法获得对方进程可靠的UID/PID（用户ID/进程ID），从而无法鉴别对方身份。Android为每个安装好的应用程序分配了自己的UID，故进程的UID是鉴别进程身份的重要标志。使用传统IPC只能由用户在数据包里填入UID/PID，但这样不可靠，容易被恶意程序利用。可靠的身份标记只有由IPC机制本身在内核中添加。其次传统IPC访问接入点是开放的，无法建立私有通道。比如命名管道的名称，system V的键值，socket的ip地址或文件名都是开放的，只要知道这些接入点的程序都可以和对端建立连接，不管怎样都无法阻止恶意程序通过猜测接收方地址获得连接。

主要从传输性能和安全性进行比较
传输性能：
socket作为一款通用接口，其传输效率低，开销大，主要用在跨网络的进程间通信和本机上进程间的低速通信。
共享内存无需拷贝，但控制复杂，难以使用。
消息队列和管道至少有两次拷贝过程。
Binder传输过程只需一次拷贝。

安全性：
传统IPC没有任何安全措施，完全依赖上层协议来确保。
Binder为发送方添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。

为什么Binder只需要一次复制？
因为服务端进程的用户空间通过系统调用mmap机制和内核空间进行了映射，因此客户端进程只需要将数据复制到内核空间与服务端的用户空间映射的那块内存上，服务端就可以直接获取数据了。

共享内存为什么不需要复制？
是因为服务端进程和客户端进程都通过系统调用mmap机制和内核空间进行了映射，而且是映射到了同一块内核空间中。因为是不同进程，所以读取同一块内存可能出现同步的问题，因此需要信号量来保证进程访问内存的同步性
```

| **IPC**              | **数据拷贝次数** |
| -------------------- | ---------------- |
| 共享内存             | 0                |
| Binder               | 1                |
| Socket/管道/消息队列 | 2                |


# IPC
IPC 即 Inter-Process Communication (进程间通信)。Android 基于 Linux，而 Linux 出于安全考虑，不同进程间不能之间操作对方的数据，这叫做“进程隔离”。
> 在 Linux 系统中，虚拟内存机制为每个进程分配了线性连续的内存空间，操作系统将这种虚拟内存空间映射到物理内存空间，每个进程有自己的虚拟内存空间，进而不能操作其他进程的内存空间，只有操作系统才有权限操作物理内存空间。 进程隔离保证了每个进程的内存安全。

## IPC方式

RPC（Remote Procedure Call 远程进程调用）

AIDL（Android Interface Definination Language)

| 名称 | 优点 | 缺点 | 适用场景 |
|----|-----|----|----|
| Bundle | 简单易用 | 只能传输 Bundle 支持的数据类型 | 四大组件间的进程间通信|
| 文件共享 | 简单易用 | 不适合高并发场景，并且无法做到进程间即时通信|无并发访问情形，交换简单的数据实时性不高的场景|
| AIDL |功能强大，支持一对多并发通信，支持实时通信 | 使用稍复杂，需要处理好线程同步 | 一对多通信且有 RPC 需求|
| Messenger | 功能一般，支持一对多串行通信，支持实时通信|不能处理高并发情况，不支持 RPC，数据通过 Message 进行传输，因此只能传输 Bundle 支持的数据类型|低并发的一对多即时通信，无RPC需求，或者无需返回结果的RPC需求|
| ContentProvider | 在数据源访问方面功能强大，支持一对多并发数据共享，可通过 Call 方法扩展其他操作|可以理解为受约束的 AIDL，主要提供数据源的 CRUD 操作 | 一对多的进程间数据共享|
| Socket | 可以通过网络传输字节流，支持一对多并发实时通信 | 实现细节稍微有点烦琐，不支持直接的RPC | 网络数据交换|

## Binder
Binder 是 Android 中的一个类，实现了 IBinder 接口。从 IPC 角度来说，Binder 是 Android 中的一种夸进程通信方式。从 Android 应用层来说，Binder 是客户端和服务器端进行通信的媒介，当 bindService 的时候，服务端会返回一个包含了服务端业务调用的 Binder 对象。

Binder 相较于传统 IPC 来说更适合于Android系统，具体原因的包括如下三点：
- Binder 本身是 C/S 架构的，这一点更符合 Android 系统的架构
- 性能上更有优势：管道，消息队列，Socket 的通讯都需要两次数据拷贝，而 Binder 只需要一次。要知道，对于系统底层的 IPC 形式，少一次数据拷贝，对整体性能的影响是非常之大的
- 安全性更好：传统 IPC 形式，无法得到对方的身份标识（UID/GID)，而在使用 Binder IPC 时，这些身份标示是跟随调用过程而自动传递的。Server 端很容易就可以知道 Client 端的身份，非常便于做安全检查

示例：

- **新建AIDL接口文件**
  

``RemoteService.aidl``
```java
package com.example.mystudyapplication3;

interface IRemoteService {

    int getUserId();

}
```

系统会自动生成 ``IRemoteService.java``:

```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.example.mystudyapplication3;
// Declare any non-default types here with import statements
//import com.example.mystudyapplication3.IUserBean;

public interface IRemoteService extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.example.mystudyapplication3.IRemoteService {
        private static final java.lang.String DESCRIPTOR = "com.example.mystudyapplication3.IRemoteService";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.example.mystudyapplication3.IRemoteService interface,
         * generating a proxy if needed.
         */
        public static com.example.mystudyapplication3.IRemoteService asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.example.mystudyapplication3.IRemoteService))) {
                return ((com.example.mystudyapplication3.IRemoteService) iin);
            }
            return new com.example.mystudyapplication3.IRemoteService.Stub.Proxy(obj);
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
                case TRANSACTION_getUserId: {
                    data.enforceInterface(descriptor);
                    int _result = this.getUserId();
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements com.example.mystudyapplication3.IRemoteService {
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
            public int getUserId() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getUserId, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }

        static final int TRANSACTION_getUserId = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    }

    public int getUserId() throws android.os.RemoteException;
}
```

| 方法 | 含义 |
|--|--|
| DESCRIPTOR | Binder 的唯一标识，一般用当前的 Binder 的类名表示|
| asInterface(IBinder obj) | 将服务端的 Binder 对象成客户端所需的 AIDL 接口类型对象，这种转换过程是区分进程的，如果位于同一进程，返回的就是 Stub 对象本身，否则返回的是系统封装后的 Stub.proxy 对象。|
| asBinder | 用于返回当前 Binder 对象|
| onTransact | 运行在服务端中的 Binder 线程池中，远程请求会通过系统底层封装后交由此方法来处理|

| 定向 tag | 含义|
|--|--|
| in | 数据只能由客户端流向服务端，服务端将会收到客户端对象的完整数据，客户端对象不会因为服务端对传参的修改而发生变动。|
| out | 数据只能由服务端流向客户端，服务端将会收到客户端对象，该对象不为空，但是它里面的字段为空，但是在服务端对该对象作任何修改之后客户端的传参对象都会同步改动。|
| inout | 服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。|

### 流程
![](http://gityuan.com/images/binder/binder_start_service/binder_ipc_arch.jpg)

## AIDL 通信
Android Interface Definition Language

使用示例：
- **新建AIDL接口文件**
```java
// RemoteService.aidl
package com.example.mystudyapplication3;

interface IRemoteService {

    int getUserId();

}
```
- **创建远程服务**
```java
public class RemoteService extends Service {

    private int mId = -1;

    private Binder binder = new IRemoteService.Stub() {

        @Override
        public int getUserId() throws RemoteException {
            return mId;
        }
    };

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        mId = 1256;
        return binder;
    }
}
```
- **声明远程服务**
```java
<service
    android:name=".RemoteService"
    android:process=":aidl" />
```
- **绑定远程服务**
```java
public class MainActivity extends AppCompatActivity {

    public static final String TAG = "wzq";

    IRemoteService iRemoteService;
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iRemoteService = IRemoteService.Stub.asInterface(service);
            try {
                Log.d(TAG, String.valueOf(iRemoteService.getUserId()));
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            iRemoteService = null;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bindService(new Intent(MainActivity.this, RemoteService.class), mConnection, Context.BIND_AUTO_CREATE);
    }
}
```