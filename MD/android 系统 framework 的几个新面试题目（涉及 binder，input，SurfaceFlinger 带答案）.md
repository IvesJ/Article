> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5XtuKElaCcmIaj1Svolqbw?poc_token=HD8bUGijOyUx0Rk5qtLOpkIONhzd_mTNlwMGCg3c)

背景：
---

今天给大家分享几个学员朋友面试过程中带回来的几个新面试题，这些面试题目属于比较独特一些，一些不属于 1+1=2 直接有标准答案，但是需要对模块熟悉后有一些自己的理解和思考答出的开放性题目，比如问题 1 和问题 2 就属于这种，所以这种题目可能面试官自己也没有明确的面试答案哈，我这里也整理了一些答案，有的也有 ai 一些整理归纳功劳哈。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2otAG3MlmicQ3Oqent4X9qzge9xHNkFgWMUT6wiciaORHiaCALkVXZTs0O1NobsOXv2rPibcIDRf46oOKQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

问题 1：
-----

#### 系统中`audioflinger`和`surfaceflinger`作为独立的 Native 守护进程运行，而`inputflinger`与`system_server`进程绑定，为什么 inputflinger 不作为独立进程

主要基于以下设计考量：

1.  **事件分发的高实时性要求**  
    
    Input 事件（如触摸、按键）的处理对延迟极其敏感，需要快速响应并分发到应用进程。若`inputflinger`独立为守护进程，需通过 IPC（如 Binder）与`system_server`通信，这会引入额外的进程间通信开销，可能影响事件分发的实时性。而当前集成在`system_server`中的设计，通过线程级交互（如`InputReaderThread`和`InputDispatcherThread`）直接处理事件，避免了跨进程延迟。
    
2.  **与窗口管理的紧密耦合**  
    
    Input 事件的分发逻辑高度依赖窗口管理器（`WindowManagerService`，WMS）。例如，WMS 需为应用进程提供`InputChannel`以接收事件，并处理焦点窗口切换、触摸事件拦截等逻辑。若`inputflinger`独立，需频繁与 WMS 跨进程同步状态，增加复杂性和性能损耗。集成在`system_server`中可减少这类协调成本。
    
3.  **系统服务启动与依赖管理**
    
    `system_server`是 Android 的核心服务管理器，负责按顺序启动和依赖解析。`inputflinger`的初始化依赖于`system_server`提供的环境（如`DisplayThread`的 Looper），且需在 WMS 启动后立即可用。独立化需额外设计进程生命周期管理，可能引入启动顺序或死锁风险。
    
4.  **资源与性能权衡**  
    
    独立进程虽能提升模块隔离性，但会占用额外内存和 CPU 资源。`inputflinger`作为高频调用的服务，独立后需维护常驻进程，可能得不偿失。而`audioflinger`和`surfaceflinger`因涉及硬件资源独占（如音频设备、显示合成），独立进程更利于资源调度。
    
5.  **历史架构演进**  
    
    早期 Android 版本中，`inputflinger`曾尝试过独立进程设计（如通过`main.cpp`启动），但最终因上述问题回归`system_server`集成。后续优化
    

问题 2：
-----

#### surfaceflinger 为什么要设置 binder 线程为 4 个

SurfaceFlinger 自己在 main_surfaceflinger.cpp 里主动把 Binder 线程池的上限从系统默认的 15 条降到了 4 条：

```
// frameworks/native/services/surfaceflinger/main_surfaceflinger.cpp
int main(int argc, char** argv) {
    sp<ProcessState> proc(ProcessState::self());
    ProcessState::self()->setThreadPoolMaxThreadCount(4);   // 关键语句
    …
}


```

之所以只开 4 条线程，而不是让 Binder 驱动自动根据负载无限制地拉起线程，主要是下面几方面的权衡。

并发度需求有限

 • SurfaceFlinger 对外只暴露 ISurfaceComposer、IDisplayEventConnection 等少量接口。

• 绝大部分事务最终都要在主线程（“VSYNC 线程”）里执行，因为它持有全局锁 mStateLock 并驱动合成管线。Binder 线程做的只是把 IPC 转成主线程消息，再多线程也无法突破这把大锁带来的串行瓶颈。

结论：多于 3-4 条线程并不能显著提升吞吐。

避免 CPU 抢占导致掉帧

• SurfaceFlinger 的主线程和合成线程都使用 SCHED_FIFO/SCHED_OTHER + 高优先级。 • 若 Binder 线程数量过多，它们同样是高优先级，容易在调度器里与合成关键路径「抢时间片」，造成 jank／帧间隔抖动。 • 控制线程数可以将 CPU 占用保持在一个可预测范围，降低实时渲染链路被意外打断的概率。

减少内存与上下文切换开销

• 每条 Binder 线程的用户栈默认 1 MiB，再加 Binder 驱动内核栈，线程越多越浪费。

• Binder 事务通常很短（几百微秒级），大量线程反而会导致锁竞争、cache miss 和上下文切换开销上升。

早期设备资源受限的历史包袱

• 最初做参数调优时，主流设备只有 1-2 GiB RAM、4-8 核 CPU，实验表明 4 条线程已经覆盖 99% 使用场景。这个值后来一直沿用。

Dos／恶意调用防护

• 将线程数限定得较小可防止外部 Service 使用海量并发事务拖垮 SurfaceFlinger，属于一种「背压」策略。

总结 SurfaceFlinger 作为图形栈核心进程，对实时性和确定性要求极高；Binder 请求本身对并发需求不大，却可能对调度造成负面影响。综合测试结果后，Google 把线程池上限固定为 4，正好在「满足事务峰值」与「最小化资源占用、保障帧率」之间取得平衡。

问题 3：
-----

#### 讲述一下 binder 机制中 binder 多线程的支持

1.  使用 Binder 的进程在启动之后，通过 BINDER_SET_MAX_THREADS 告知驱动其支持的最大线程数量
    
2.  驱动会对线程进行管理。在 binder_proc 结构中，这些字段记录了进程中线程的信息：max_threads，requested_threads，requested_threads_started
    
3.  binder_thread 结构对应了 Binder 进程中的线程
    
4.  驱动通过 BR_SPAWN_LOOPER 命令告知进程需要创建一个新的线程
    
5.  进程通过 BC_ENTER_LOOPER 命令告知驱动其主线程已经 ready
    
6.  进程通过 BC_REGISTER_LOOPER 命令告知驱动其子线程（非主线程）已经 ready
    
7.  进程通过 BC_EXIT_LOOPER 命令告知驱动其线程将要退出
    
8.  在线程退出之后，通过 BINDER_THREAD_EXIT 告知 Binder 驱动。驱动将对应的 binder_thread 对象销毁
    
    流程图如下：
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2pgwsN22XmgbkDAotBtoDZfZ5lYIoejzQ9wl856Rjl29CV1xVc7LKBkIvoiaN5d2dL5dtAU4qaay0Q/640?wx_fmt=png&from=appmsg&watermark=1)

其他 framework 实战技术干货相关手把手课程资料：

[](https://mp.weixin.qq.com/s?__biz=MzkzOTQ4NDUyNg==&mid=2247484186&idx=1&sn=328a6efaf16b78b1029b3595be03268b&scene=21#wechat_redirect)[Android Framework 开发 rom 实战合集课表 / 车载车机手机高级系统开发工程必会技能](https://mp.weixin.qq.com/s?__biz=MzkzOTQ4NDUyNg==&mid=2247484186&idx=1&sn=328a6efaf16b78b1029b3595be03268b&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2osas0xlUuOGicHsjnhibC1f59PLibaT8XRca0vysZoleXmG6iaiaB6ppyBydjRIt28ibjj4y9t6Zg23JQA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

  

具体优惠购买和成为 vip 学员加入 vip 群可以私聊马哥微信号：

androidframework007

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2psicybK2UNpjjHiagw9kwTgju4GQKtkwYAl5pAE7X6CJVVXDpAyAkSMvmNuUczgLk4n4xnYXkHGwMw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)