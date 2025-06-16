> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_pLQVTcH2shoIAk1upSb7A)

背景：
---

经常有学员出去进行 framework 相关的面试，他们都会给马哥反馈一些面试题目，今天给大家整理一下方便大家进行面试前的准备

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2otAG3MlmicQ3Oqent4X9qzge9xHNkFgWMUT6wiciaORHiaCALkVXZTs0O1NobsOXv2rPibcIDRf46oOKQ/640?wx_fmt=png&from=appmsg)

列出的面试题目大部分都是有答案的，答案可以 vip 群获取，有的是没有答案的，毕竟是去人家公司面试的，面试官也不是马哥，当然也就没有相关的答案，不过看到题目大家一般都可以有自己的答案，大家不确定的可以 vip 群中丢出来讨论。

面试题目汇总：‍‍‍‍‍‍‍‍‍‍‍
------------------

1、socketpair 对比 socket 的区别和关系

2、socketpair 创建是在同一进程，那么如何让 socketpair 达到跨进程通信的效果？

3、app 在主线程 epoll 一般会监听哪几个 fd（looper）

4、binder 通信 oneway 与非 oneway 的区别？oneway 通信时需要注意什么？

5、binder 通信中不同进程的指向同一个 Binder 服务的客户端请求服务端时使用的 handle 是否是相等的呢? 比如 AMS 服务

6、在源码中我们经常会有将 binder 对象当作 token，利用了 binder 的什么原理熟悉

7、Dumpsys window windows 命令和层级结构树的哪一个层级相对应？

8、如何根据 dumpsys 快速找到对应的代码

9、代码中 dump 类可以找到，但是 dumpsys 的指令找不到，这时有什么好的办法呢？

9.1、请问用 winscope 做过什么实战项目相关的问题？

10、Perfetto 的线程运行状态的颜色区分

11、如何查看开机时各阶段的耗时。

12、判断权限的时候，经常会传入一个 uid，这个 uid 跟多用户的 userId 有什么联系吗？

13、binder 通信中经常会调用 clearCallingIdentity，这个用法目的是什么？14、为什么 clearCallingIdentity 之后要进行 restoreCallingIdentity 呢？

15、input 事件的流程？

16、inputreader 是如何通知 inputDispatch 启动的？

17、聊一下 IQ,OQ,WQ

18、聊一下 input anr 中的 not response 的异常？

19、在 onresume 中进行耗时 10s 的操作，会产生 anr 吗？

20、在一个 button 点击，不松手会触发 anr 吗？

21、在 click 中进行延时 10 秒的操作，是否会触发 anr？

22、一般遇到冻屏问题你的分析思路

23、自己进程调用自己进程的 binder 接口，是否通过 binder 驱动，请详细说明每个流程

24、普通第三方应用如果想要检测自己是否 ANR 有哪些方案或者思路

25、车载单屏显示多窗口，多屏你有啥实现思路？

真实面试一些案例面试题：

1、简单说说你做过的项目？

2、有没有遇到过比较难解决的问题？

3、既然后黑屏冻屏的问题，你说说你分析的这个几个问题，是怎么分析的？

4、对 SurfaceFlinger 做过什么修改？

5、BLASTBufferQueue 中数据的流程？

6、你说你熟悉 perfetto，那么线程里面一个方法执行太久了，用 perfetto 怎么去分析？

7、有没有用过 perfetto 去看帧率？

8、对系统启动有没有做过优化？

9、我想开机就启动一个 Native 进程 怎么去启动？

10、系统的启动流程？

相关面试题目答案，欢迎各位 vip 学员群里进行讨论哈

其他 framework 实战技术干货相关手把手课程资料：

[https://mp.weixin.qq.com/s/Qv8zjgQ0CkalKmvi8tMGaw](https://mp.weixin.qq.com/s?__biz=MzkzOTQ4NDUyNg==&mid=2247484186&idx=1&sn=328a6efaf16b78b1029b3595be03268b&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2osas0xlUuOGicHsjnhibC1f59PLibaT8XRca0vysZoleXmG6iaiaB6ppyBydjRIt28ibjj4y9t6Zg23JQA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

  

具体优惠购买和成为 vip 学员加入 vip 群可以私聊马哥微信号：

androidframework007

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DYicOkJDdA2psicybK2UNpjjHiagw9kwTgju4GQKtkwYAl5pAE7X6CJVVXDpAyAkSMvmNuUczgLk4n4xnYXkHGwMw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)