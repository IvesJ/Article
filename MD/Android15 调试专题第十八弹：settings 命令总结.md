> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/YnRKKo5koUuZSai9Kq0Tpw)

**点击下方👇关注** **Android 系统攻城狮**

第 156 篇原创文章

作者：一箭咸鱼

每日充电：OS+MultiMedia 学习之旅

**学习理念：一次理解一个知识点**

**源码环境：Android15  
**

**硬件平台：Tensor GS101**

**学习内容：settings 命令总结**

****难度：★★★☆☆****

**阅读时间：2.2 分钟**

**01**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**list 命令**

**1. 用途：列出所有配置项**

**应用场景 1：列出 global 空间配置 (对应路径:/data/system/users/0/settings_global.xml)**

```
settings list global


```

**应用场景 2：列出 secure 空间配置 (对应路径:/data/system/users/0/settings_secure.xml)**

```
settings list secure

```

**应用场景 3：列出 system 空间配置 (对应路径:/data/system/users/0/settings_system.xml)**

```
settings list system 

```

**02**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**get 命令**

**2. 用途：获取某个键值对**

**应用场景 1：获取当前用户的屏幕超时时间（单位：毫秒）**

```
settings get system screen_off_timeout 
6000000

```

**应用场景 2：获取用户 0 蓝牙名称**

```
settings get --user 0 secure bluetooth_name 
AOSP on oriole

```

**03**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**put 命令**

**3. 用途：修改或添加键值对**

**应用场景 1：启用飞行模式**

```
settings put global airplane_mode_on 1

```

**应用场景 2：修改屏幕超时时间为 1000ms**

```
settings put system screen_off_timeout 1000

```

**应用场景 3：随意增加一个设置选项**

```
settings put system test_01 2
查看是否设置成功:
settings get system test_01 
2

```

**04**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**delete 命令**

**4. 用途：删除键值对**

**应用场景 1：删除自定义配置项: test_01**

```
settings delete system test_01
Deleted 1 rows
查看是否删除成功:
settings get system test_01
null

```

**05**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/eZ1UicZh3icjkUVWC0Enmicxa43VwlVaAg1y7N2XaYtiaiaLtcDvGMsf9s75wGuPWAFtVBTrwRRsHYjiatNNWzJvNJJg/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**reset 命令**

**5. 用途：重置配置项**

**应用场景 1：重置 com.android.car.radio 在 global 空间配置**

```
settings reset global com.android.car.radio

```

**应用场景 2：以 untrusted_clear 模式重置 secure 空间配置（包含三种模式：untrusted_defaults, untrusted_clear, trusted_defaults）**

```
settings reset secure untrusted_clear

```

*   若读者朋友发现有错误、疑问的地方，或者好的建议，欢迎拍砖！！！
    

**★★★★★★★★★★<END>****★****★★★★★★★★★**

**关注，回复【****1024****】, 限时领取：OS+Multimedia 资料包**

**精彩文章合集**  

 **专栏推荐**

☞【专栏一】[基础原理系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2NTI3NDI5MQ==&action=getalbum&album_id=2318389963295375364#wechat_redirect)

☞【专栏二】[Android 音频系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2NTI3NDI5MQ==&action=getalbum&album_id=2789502080410648581#wechat_redirect)

☞【专栏三】[Android 编解码系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2NTI3NDI5MQ==&action=getalbum&album_id=3055046478127366144#wechat_redirect)

☞【专栏四】[Android 相机系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2NTI3NDI5MQ==&action=getalbum&album_id=3003914888358068225#wechat_redirect)

☞【专栏五】[](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzUxMjEyNDgyNw==&action=getalbum&album_id=1598710257097179137#wechat_redirect)[Android 图形系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2NTI3NDI5MQ==&action=getalbum&album_id=3028698257297965057#wechat_redirect)

☞【专栏六】[](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzUxMjEyNDgyNw==&action=getalbum&album_id=1502410824114569216#wechat_redirect)[Android 系统系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2NTI3NDI5MQ==&action=getalbum&album_id=2987704999789133826#wechat_redirect)

☞【专栏七】[](http://mp.weixin.qq.com/s?__biz=MzUxMjEyNDgyNw==&mid=2247496985&idx=1&sn=c3d5e8406ff328be92d3ef4814108cd0&chksm=f96b87edce1c0efb6f60a6a0088c714087e4a908db1938c44251cdd5175462160e26d50baf24&scene=21#wechat_redirect)[WSL 进击 AOSP 系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2NTI3NDI5MQ==&action=getalbum&album_id=1400721183586484224#wechat_redirect)

☞【专栏八】[Android15 调试系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2NTI3NDI5MQ==&action=getalbum&album_id=3332030181901008900&scene=173&subscene=&sessionid=svr_55394a47769&enterid=1741183483&from_msgid=2247490152&from_itemidx=1&count=3&nolastread=1#wechat_redirect)

********原创不易****, 谢谢****支持~~********