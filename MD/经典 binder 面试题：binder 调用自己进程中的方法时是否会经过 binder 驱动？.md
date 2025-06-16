> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/UFV8Pi7UceA8Wj24lhFULw)

背景：

经常有学员会参加一些面试时候，会在群里带回一些经典 binder 面试题：binder 调用自己进程中的方法时是否会经过 binder 驱动？

针对这个题目其实还是属于比较有意思的，因为它并不是那种死记硬背背书党会 care 到的一个题目，也可以侧面深挖出 binder 对象的传递流程。题目其实主要考察以下两个部分：

1、对 binder 学习使用后的独立的深入思考能力

2、binder 对象传递流程方式的剖析

同时感谢学员朋友汽车人领袖的投稿关于剖析该经典面试题。

下面开始汽车人同学的投稿内容：

为了直接得出答案，我在马哥课程 (详见：binder 使用方式及常见组成及案例分析) 中的 demo 基础上加以修改，在 bindService 的 onServiceConnected 回调中的某个 binder 方法调用时加入 trace，方便查看，如图：![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2rtqGwnicn7Eg55xlV71qhiarYCvUBZTiaSI0KJqlECiaogRpviaKbcYLKGPIOmZAbdvgkQfb3CSOnqAzw/640?wx_fmt=png&from=appmsg)然后运行 app，抓取 perfetto，然后搜索 fyytestbinder, 得到结果如图：![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2rtqGwnicn7Eg55xlV71qhiarr14cVMfIxQGaCKXRvRfvrDEX7Is8f7Ty7yCicOjc3Td5moALhyiaHPoA/640?wx_fmt=png&from=appmsg)然后，在 AndroidManifest.xml 中，把 MyService 的配置修改，这样 MyService 就和 app 在同一个进程了：

```
        <service
            android:
            android:enabled="true"
            android:exported="true"></service>
<!--        <service-->
<!--            android:-->
<!--            android:enabled="true"-->
<!--            android:process=":myservice"-->
<!--            android:exported="true"></service>-->


```

然后，再次抓取 perfetto：![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2rtqGwnicn7Eg55xlV71qhiarHeqpccXKwBnoOtSN7BHcj5st1SBQvIviciaRemibJYvTLDNicGrxCHZ1LA/640?wx_fmt=png&from=appmsg)通过对比，很明显，当 service 和 app 在不同进程时，在 perfetto 中看到有 binder transaction; 当在同一个进程时，没有看到 binder transaction 。

**所以，binder 调用自己进程中的方法时是不会经过 binder 驱动的。**

这是为什么呢？

其实问题的关键就在这行：

```
@Override
public void onServiceConnected(ComponentName name, IBinder service) {
    IStudentInterface remoteInterface = IStudentInterface.Stub.asInterface(service);
    //省略
}


```

这是 bindService 的 onServiceConnected 回调中的第一行代码，把 IBinder 通过 asInterface() 这个方法转换成 IStudentInterface，而 asInterface() 详情如下：

```
public static com.example.myapplication.IStudentInterface asInterface(android.os.IBinder obj)
    {
      if ((obj==null)) {
        return null;
      }
      android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
      if (((iin!=null)&&(iin instanceof com.example.myapplication.IStudentInterface))) {
        return ((com.example.myapplication.IStudentInterface)iin);
      }
      return new com.example.myapplication.IStudentInterface.Stub.Proxy(obj);
    }


```

可以看到，这里通过 queryLocalInterface() 方法的返回值，来决定返回的是一个 IStudentInterface, 还是一个 IStudentInterface.Stub.Proxy, 如果是后者，则会调用到 mRemote.transact(), 这就肯定是会走 binder 驱动了。

来看 queryLocalInterface() 方法，它是 IBinder 的一个接口方法，只有两个地方对它进行了实现：

Binder.java 中：

```
public @Nullable IInterface queryLocalInterface(@NonNull String descriptor) {
        if (mDescriptor != null && mDescriptor.equals(descriptor)) {
            return mOwner;
        }
        return null;
    }


```

BinderProxy.java 中：

```
   public IInterface queryLocalInterface(String descriptor) {
        return null;
    }


```

那 asInterface() 方法中，obj.queryLocalInterface(DESCRIPTOR)；这里这个 obj 是 Binder 还是 BinderProxy 呢？ 往回看，这个 obj 就是 bindService 的 onServiceConnected 回调中给的，bindService 是个复杂的过程，马哥曾经有博客详细分析过，总之就是 client 端与 systemServer 进行 binder 通信，systemServer 又与 server 端进行 binder 通信，最终得到了 server 端的 binder 代理，通过 onServiceConnected 回调传给了 client 端。

而 binder 对象在进程间的传递过程又涉及到 binder 驱动的相关内容，其关键部分我觉得主要是： bind.c 中的两个方法：binder_translate_handle() 和 binder_translate_binder() Parcel.cpp 中的两个方法：flattenBinder() 和 unflattenBinder()

这几个方法共同保证了：

**不管 binder 对象在进程间怎么传递，传递了多少次，最终， 在 binder 对象所属的进程中，拿到的就是 BBinder 对象 (c++ 层)，对应 java 层的 Binder 对象； 在其他的进程中，拿到的就是 BpBinder 对象 (c++ 层)，对应 java 层的 BinderProxy 对象。**

回到上面的例子，app 和 service 在同一进程中时，当 bindService 时，其实是 service 中的 binder 传到了 systemServer 进程后，又传回了 app 进程。最后传回 app 进程时，在驱动层应该经历了如下代码： binder_translate_handle() 中：

```
if (node->proc == target_proc) {//目标进程就是binder node的原进程
  if (fp->hdr.type == BINDER_TYPE_HANDLE)
   fp->hdr.type = BINDER_TYPE_BINDER;//走这里
  else
   fp->hdr.type = BINDER_TYPE_WEAK_BINDER;
  fp->binder = node->ptr;
  fp->cookie = node->cookie;
  //省略
}


```

回到用户空间后，在 Parcel::unflattenBinder() 中

```
 switch (flat->hdr.type) {
            case BINDER_TYPE_BINDER: {
            //这里已经回到了原进程，根据cookie指针，直接获取到了IBinder对象，应该就是BBinder或BBinder的子类对象
                sp<IBinder> binder =
                        sp<IBinder>::fromExisting(reinterpret_cast<IBinder*>(flat->cookie));
                return finishUnflattenBinder(binder, out);
            }
            case BINDER_TYPE_HANDLE: {
            //这里会根据handle，生成一个BpBinder
                sp<IBinder> binder =
                    ProcessState::self()->getStrongProxyForHandle(flat->handle);
                return finishUnflattenBinder(binder, out);
            }
        }


```

再回到上文，所以 asInterface() 方法中，obj.queryLocalInterface(DESCRIPTOR)；这里这个 obj 是 Binder，而不是 BinderProxy。 而 Binder.java 中 queryLocalInterface() 返回的 mOwner，在这里，其实就是 IStudentInterface.Stub 对象，最终，asInterface() 方法，返回的其实就是 MyService 中的 mBinder：

```
public class MyService extends Service {
    public MyService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        return mBinder;
//        return  null;
    }
    IChangeCallback mCallBack = null;
    IStudentInterface.Stub mBinder = new IStudentInterface.Stub() {
  //省略
  }
}


```

而 MyService 和 app 属同一个进程，所以，此时 binder 调用方法其实就相当于调用本进程中另一个对象的方法，自然不会经过 binder 驱动了。

其他 framework 实战技术干货相关手把手课程资料：

[https://mp.weixin.qq.com/s/Qv8zjgQ0CkalKmvi8tMGaw](https://mp.weixin.qq.com/s?__biz=MzkzOTQ4NDUyNg==&mid=2247484186&idx=1&sn=328a6efaf16b78b1029b3595be03268b&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2osas0xlUuOGicHsjnhibC1f59PLibaT8XRca0vysZoleXmG6iaiaB6ppyBydjRIt28ibjj4y9t6Zg23JQA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

  

具体优惠购买和成为 vip 学员加入 vip 群可以私聊马哥微信号：

androidframework007

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2psicybK2UNpjjHiagw9kwTgju4GQKtkwYAl5pAE7X6CJVVXDpAyAkSMvmNuUczgLk4n4xnYXkHGwMw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)