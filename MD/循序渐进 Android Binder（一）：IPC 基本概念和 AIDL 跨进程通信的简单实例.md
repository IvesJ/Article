> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6d6dd82j07u2eDDc9Xp-Wg)

Binder  给人的第一印象是” 捆绑者 “，即将两个需要建立关系的事物用某些工具束缚在一起。在 Android 中，Binder  是一种高效的跨进程通信（IPC）机制，它将可以将运行在不同进程中的组件进行绑定，以实现彼此通信和数据共享。Binder 机制在 Android  系统中扮演这至关重要的角色，特别是在管理应用程序和系统服务之间的交互方面。

在  Android 系统中，Binder 占有举足轻重的地位。而 Binder 机制涉及的东西即多且复杂，对于学习者来说，想要全面理解  Binder  仅靠一两篇文章就融会贯通是不可能的。越复杂的东西，越需要循序渐进，逐步理解，以免迷失了方向。因此，我将通过一系列文章，逐步带领大家进入  Binder 的内容，最终实现对于 Binder 的理解和运用。

![](https://mmbiz.qpic.cn/mmbiz_png/U6YYnax9cCMuXxw3hsF6lIfuwoLqia44eEMJia4KLsbBD9PBzjgCwYdp8n6uicKNXkbHiavPOCIwGqQYKM77Dq8f7g/640?wx_fmt=png&from=appmsg)

我们知道 Binder 是一个 IPC 工具，那什么是 IPC 呢？我们为什么需要这玩意呢？接下来，我们就从此问题开始，逐步进入 Binder 的世界。

什么是 IPC？为什么我们需要它？
-----------------

在操作系统中，每个进程都是在一个独立的沙箱环境中的，它拥有自己的内存空间、权限边界、资源管理。为了保证系统稳定性和安全性，进程之间是无法直接访问彼此的数据或代码的。

但是在实际开发中，我们经常需要不同进程之间进行协作。例如：

•APP 通过 DisplayManager 获取屏幕的物理像素；• 从一个 Activity 跳转到另一个 Activity；

这些场景都离不开一个关键词：IPC（Inter-Process Communication），进程间通信。这是一种技术或方法，用于在至少两个进程或线程之间传输数据或信号。

这时候，有人会问了，为什么操作系统要将应用程序放到独立的沙箱环境中运行呢？都放到系统空间中，效率岂不是更高，而且避免了进程间通信的这些麻烦事。

这是个好问题，也确实有些系统不强制应用程序运行在独立空间中，而允许其直接运行在内核态。例如，在早期的 DOS 系统中，没有用户态 / 内核态之分，程序可以直接操作硬件；而现在的很拖 RTOS 也是不区分用户态、内核态的，其目的就是为了极致的性能表现。

但这些操作系统基本都是面向特定领域，不做应用程序隔离的设计使其无法成为面向大众的通用操作系统。毕竟，谁都不喜欢一个 BUG 就蓝屏的系统吧。

![](https://mmbiz.qpic.cn/mmbiz_png/U6YYnax9cCMuXxw3hsF6lIfuwoLqia44e6VIdlzOJ2mab9ktfpmwD52q8NSNpEOo1YTddBdIsToo0rJ04En247g/640?wx_fmt=png&from=appmsg)

因此，现代主流通用操作系统（Linux、Windows、macOS、Android、iOS）都强制将应用运行在用户态中。这里我们解释下用户态和内核态这两个概念：

• 用户态（User Mode）：应用程序运行的环境，不能直接访问硬件和内核资源，必须通过系统调用来请求服务。• 内核态（Kernel Mode）：操作系统内核拥有最高权限，负责调度、资源管理、设备控制等。

这么设计的核心原因是：安全性 + 稳定性 + 可控性。如果允许一个普通程序在内核态运行，那么它一旦崩溃，就可能导致整个系统内核崩溃，影响所有进程。所以这是一种 “以安全和控制为核心” 的设计哲学。

Android 系统中支持的 IPC 机制
---------------------

上面已经介绍了 IPC 的基本概念，那 Android 系统支持哪些 IPC 机制呢？

首先，Android 基于 Linux 内核，因此天然继承了 Linux 提供的各种经典 IPC 机制，这这些机制主要包括：

• 管道（Pipe）：单向通信、半双工，只适用于亲缘进程间通信（如父子进程）• 命名管道（FIFO）：不支持双向通信，只适用于简单的数据流 • 消息队列（Message Queue）：内核对象生命周期管理复杂，数据大小有限，不适合大数据结构 • 共享内存（Shared Memory）：高效但非常依赖开发者手动控制同步，容易出错 • 信号量（Semaphore）：仅限控制用途，无法传递数据 • 套接字（Socket）：支持远程通信但性能较低，安全性不够强，需要开发者自己维护协议等

Android  虽然基于 Linux，但并没有直接采用传统的 Linux IPC  机制来构建系统服务框架。原因很简单：这些机制（如管道、消息队列、信号）要么功能太弱，要么难以管理、缺乏安全保障。为了构建一个高性能、可控、安全的跨进程通信模型，Google  选择在内核层扩展了一套独特的机制 —— Binder。

Binder 的优势
----------

Binder 是 Android 独有的一套进程通信机制，底层由内核模块 binder 实现，用户态通过 Binder 驱动访问。使用它能够像调用本地方法一样，调用另一个进程中的方法。 

*   高性能：基于内核实现，采用零拷贝（zero-copy）技术，远高于 Socket 传输效率
    
*   统一接口调用语义：支持同步、异步、回调，封装成面向对象接口，通过 Stub/Proxy 自动生成代码
    
*   安全性强：每个 Binder 调用都带有 UID/PID，可用于权限验证和访问控制
    
*   支持大数据传输：配合 Ashmem 可高效传输图像、音频等大对象
    
*   服务注册 / 发现机制：由 centralized 的 ServiceManager 管理所有系统服务
    
*   内存安全 / 生命周期管理：每个 Binder 对象都有引用计数，不会悬空引用
    

Binder  的出现，是 Android 构建自身 IPC 世界的重要基石。它不仅替代了 Linux 的传统 IPC  机制，还通过提供高效、安全、可扩展的通信模型，成功支撑起整个 Android 系统服务架构 ——  从最底层的启动服务到最顶层的应用组件通信，Binder 是 Android 系统的 “神经网络”。

上面我们介绍了 IPC 和 Binder 的一些基本概念，接下来，我们就用一个 AIDL 的简单实例，来演示一下 Binder 的基本调用。

基于 AIDL 的跨进程通信的简单实例
-------------------

Binder   通信采用客户端 - 服务器（Client/Server）模型，因此即使是一个简单的跨进程通信实例，我们也要写服务端和客户端两个进程代码。在继续写代码之前，我们需要接受一个新的词：AIDL（Android  Interface Definition Language）。

![](https://mmbiz.qpic.cn/mmbiz_png/U6YYnax9cCMuXxw3hsF6lIfuwoLqia44e31ncaJGqibP39ro7GlecPPAarPamheGfwOibA1qotsmGpEIepHCF3ohw/640?wx_fmt=png&from=appmsg)

什么是 AIDL？
---------

虽然  Android Binder  提供了强大的进程通信机制，但直接使用它代价极高，开发者需要手动处理方法编号、参数序列化等细节，既麻烦又容易出错。因此 Android 引入了  AIDL 接口描述语言，其目的就是自动生成一套可靠的、类型安全的、跨进程可用的接口通信代码。

其实，使用 Binder 调用并不是必须要使用 AIDL，但是，当你需要进行 IPC，调用时，AIDL 是最清晰、高效、安全的解决方案。

一个简单的 AIDL 接口定义就如下：

```
interface IComputer {
    int add(int a, int b);
}

```

当你将其添加到项目并编译后，Android Studio 会自动生成一整套 Java 类，其主要包括：

•IComputer.Stub：服务端实现 •IComputer.Sutb.Proxy：客户端调用

你只需要在服务端继承 Stub，并在客户端调用 Proxy，就可以像调用本地方法一样进行 IPC 调用。 下面我们就以这个 AIDL 为例，来完成一个简单的 IPC 实例。

服务端
---

首先我们创建一个包名为 `lic.swift.binder.server` 的程序用于服务端。

然后添加上面的 IComputer.aidl 接口到代码中，先编译下，以便生成对应的 Java 文件。

接着创建一个全类名 `lic.swift.binder.server.ComputeService` 的 Service 作为服务端实现，如下：

```
public class ComputeService extends Service {
    private final static IComputer.Stub computer = new IComputer.Stub() { 
        @Override
        public int add(int a, int b) {
            Log.d("avril", "server, stub, add(a = " + a + ", b = " + b + "), " + getProcessLog());
            return a + b;
        }
    };
    @Override
    public IBinder onBind(Intent intent) {
        Log.d("avril", "onBind(intent = " + intent + "), return " + computer + ", " + getProcessLog());
        return computer;
    }
}

```

这个 ComputerService 里面实现了一个 IComputer.Stub 这个就是作为服务端代码的实现，也就是真正执行 `add(int, int)` 的地方。在 onBind 中返回这个实例对象。

同时我们需要在 AndroidManifest.xml 中配置一下这个 ComputerService：

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <permission android: />
    <application
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round" >
        <service android:
            android:permission="lic.swift.binder.server.CALL_SERVER"
            android:exported="true" >
            <intent-filter>
                <action android: />
                <category android:/>
            </intent-filter>
        </service>
    </application>
</manifest>

```

为了能够让其他进程的应用访问到这个  Service，需要将其 exported 设置为 true。另外在 XML 中还为这个 Service 配置了 permission。其实在  Android 中，Service  本身并不强制要求声明访问权限，但如果你希望控制哪些应用可以访问你的服务，特别是涉及跨进程调用或敏感功能的服务，那么通过 `android:permission` 是一种标准且推荐的方式。

客户端
---

上面说的 IComputer.aidl 文件，它类似于 HTTP 协议，这个协议在服务端有一个，在客户端当然也需要有一个。因此在客户端的项目中，我们需要先把这个文件复制过来，注意，这里复制时必须要保证 .aidl 文件的包名、类名都相同。

接下来我们在 Activity 中添加一个按钮，每次点击这个按钮，就调用服务端的 add 方法计算一次加法并返回。

但为了能够进行远程调用，我们首先要连接到远程服务，以下是在 Activity 中用于连接到远程服务的代码：

```
private IComputer computer = null;
private final ServiceConnection connection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        Log.d("avril", "client onServiceConnected(name = " + name + ", service = " + service + ") " + getProcessLog());
        computer = IComputer.Stub.asInterface(service);
    }
    @Override
    public void onServiceDisconnected(ComponentName name) {
        Log.d("avril", "client onServiceConnected(name = " + name + ") " + getProcessLog());
        computer = null;
    }
};
final Intent intent = new Intent();
intent.setComponent(new ComponentName("lic.swift.binder.server", "lic.swift.binder.server.ComputeService"));
boolean bindResult = bindService(intent, connection, Context.BIND_AUTO_CREATE);
Log.d("avril", "client bindService 返回 " + bindResult);

```

为了连接到远程服务器，需要在清单文件中配置远程服务器需要的权限：

```
<uses-permission android: />
<queries>
    <package android:/>
</queries>

```

在能够连接到远程服务器之后，我们就可以设置一个点击事件用于 IPC 调用了：

```
findViewById(R.id.button_1).setOnClickListener(v -> {
    if (computer == null)
        return;
    try {
        final int p0 = new Random().nextInt(1000);
        final int p1 = new Random().nextInt(1000);
        Log.d("avril", "client 远程调用 computer.add(" + p0 + ", " + p1 + ")，等待返回...");
        final long time = SystemClock.uptimeMillis();
        int r = computer.add(p0, p1);
        Log.d("avril", "client 远程调用返回结果：" + r + ". 耗时：" + (SystemClock.uptimeMillis() - time) + "ms. " + getProcessLog());
    } catch (RemoteException e) {
        Log.d("avril", "client 远程调用发生异常 RemoteException:\n" + e.getLocalizedMessage());
    }
});

```

这就是一个很简单的 AIDL 实例了。服务端和客户端的代码结构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/U6YYnax9cCMuXxw3hsF6lIfuwoLqia44eT125mUk6r4dr0aw9Escfibvm3ibTIJtCENIK31M0rUeGCkZzbibMUgcPQ/640?wx_fmt=png&from=appmsg)在运行客户端程序并点击按钮后，我们能得到如下的 log 信息：

```
lic.swift.binder.client  D  client bindService 返回 true
lic.swift.binder.server  D  onBind(intent = Intent { cmp=lic.swift.binder.server/.ComputeService }), return lic.swift.binder.server.ComputeService$1@a97ebb8, Process.myPid() = 2139, Process.myUid() = 10149
lic.swift.binder.client  D  client onServiceConnected(name = ComponentInfo{lic.swift.binder.server/lic.swift.binder.server.ComputeService}, service = android.os.BinderProxy@7c18e1b) Process.myPid() = 2219, Process.myUid() = 10150
lic.swift.binder.client  D  client 远程调用 computer.add(136, 333)，等待返回...
lic.swift.binder.server  D  server, stub, add(a = 136, b = 333), Process.myPid() = 2139, Process.myUid() = 10149
lic.swift.binder.client  D  client 远程调用返回结果：469. 耗时：26ms. Process.myPid() = 2219, Process.myUid() = 10150

```

可以看到，代码已经完成了远程调用，服务端进程号为 2139，而客户端是 2219，虽然这个耗时有点高，但正确性是没问题的。

到此为止，这个基于 AIDL 的跨进程通信的简单实例就算是介绍完了。后续我们将逐步深入到 Binder 的更深层的内容当中。

![](https://mmbiz.qpic.cn/mmbiz_png/U6YYnax9cCMuXxw3hsF6lIfuwoLqia44eHBBEODic5Yfz2M9BLE6n6hAgm9ugDHWD81IsZZJ0pibwDiaGrIPGJG1sw/640?wx_fmt=png&from=appmsg)