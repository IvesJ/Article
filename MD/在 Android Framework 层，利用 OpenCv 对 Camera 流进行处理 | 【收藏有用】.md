> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DttcHPLpNAf8kjdhNVxDpw)

OpenCv 在**计算机视觉**方面大有用处，这篇文章主要记录在 Android Framework 集成 OpenCv，对 Camera 数据流进行处理，遇到的问题记录备忘。

**openc 源码下载地址：**

https://opencv.org/releases/

**一、OpencV 在 framework 层集成**

（下面的三方算法，是指的三方算法采用到了 opencv）

### **1、三方算法是源码**

(如下面所示，image.cpp 是三方算法源码，该源码 image.cpp 文件中调用到 opencv)

```
// -----Android.bp
cc_library_shared {
name: "libimage",
rtti:true,
srcs: [
"image/image.cpp",
    ],
include_dirs: [
"frameworks/av/services/camera/libcameraservice/image/include",
    ],
shared_libs: [
"libopencv_java4",
"liblog",
    ],
export_include_dirs: ["."],
cflags: [
"-Wall",
"-Wextra",
"-Werror",
"-Wno-ignored-qualifiers",
"-fPIC",
"-ferror-limit=0",
    ],
cppflags: [
"-fno-rtti",
"-fexceptions",
    ],
}

```

### **2、三方算法是 so 库**

1 ） 如下所示，libimage 是三方算法提供的 so 库，我们自己封装了个 libProcess 的 so，libProcess 中调用三方算法的 libimage, 三方的 libimage 算法库，会调用到 opencv；

```
//----- Android.bp内容
cc_prebuilt_library_shared {
name: "libopencv_java4",
arch: {
arm: {
             srcs: ["image/lib/armeabi-v7a/libopencv_java4.so"],
         },
arm64: {
             srcs: ["image/lib/arm64-v8a/libopencv_java4.so"],
         }
     },
shared_libs: ["libandroid", "libc++", "liblog", "libmediandk", "libjnigraphics","libz","libdl","libm"],
check_elf_files: false,
}
cc_prebuilt_library_shared {
name: "libimage",
arch: {
arm: {
srcs: ["./image/lib/armeabi-v7a/libimage.so"],
         },
arm64: {
srcs: ["./image/lib/arm64-v8a/libimage.so"],
         }
     },
shared_libs: ["libopencv_java4", "libandroid", "libc++", "liblog", "libmediandk", "libjnigraphics","libz","libdl","libm"],
check_elf_files: false,
}
cc_library_shared {
name: "libProcess",
//编译引用到opencv相关头文件，需要设为true，否则编译报错
    rtti:true, 
srcs: [
"image/ImageProcess.cpp",
    ],
//调用到的opencv,opencv相关的头文件路径
    include_dirs: [
"frameworks/av/services/camera/libcameraservice/image/include",
    ],
shared_libs: [
"libimage",
"libopencv_java4",
"liblog",
    ],
export_include_dirs: ["."],
cflags: [
"-Wall",
"-Wextra",
"-Werror",
"-Wno-ignored-qualifiers",
"-fPIC",
"-ferror-limit=0",
    ],
cppflags: [
"-fno-rtti",
"-fexceptions",
    ],
}

```

2 ） 正常来说，我们调用算法的 so 库，其实是不需要再单独封装一个 so 库。这个封装成一个 so 库的原因是，三方算法调用到了 opencv，opencv 头文件的编译，需要 rtti 设为 true，而 rtti 设为 true, 会影响到 cameraservice 里面其它代码的编译。

所以比较好的解决方法就是，单独封装一个调用三方算法库的 so，在这个 so 里面，rtti 设为 true，这样不影响其它代码的编译。

```
----- 单独封装的调用三方算法库(libimage) 的 ImageProcess.cpp文件内容：
#include <iostream>
#include <opencv2/opencv.hpp>
#include <fstream>
#include <ctime>
#include<cmath>
//三方算法库头文件
#include "process.h"
using namespace cv;
using namespace std;
void imageProcess(uint8_t* buffer, uint width, uint height)
{
    //process是三方算法库的接口
    process(buffer, width, height);
}

```

在 framework 层获取 camera 相关数据，可以查看下面代码。

在下面代码逻辑里面，可以拿到拍照的 jpeg 以及录像的 yuv 数据。

frameworks/av/services/camera/libcameraservice/device3/Camera3OutputStream.cpp

```
Camera3OutputStream::returnBufferCheckedLocked{
if (getFormat() == HAL_PIXEL_FORMAT_BLOB && getDataSpace() == HAL_DATASPACE_V0_JFIF && mImageDumpMask) {
   dumpImageToDisk(timestamp, anwBuffer, anwReleaseFence);
    }
    ALOGE("%s:getFormat():%d ,width:%d ,HAL_PIXEL_FORMAT_YCbCr_420_888:%d ", __FUNCTION__,getFormat(),buffer.stream->width,HAL_PIXEL_FORMAT_YCbCr_420_888);
     //新添加内容，判断到是yuv format,才走下面的逻辑
    if (getFormat() == 34) {
        processYuv(anwBuffer,buffer.stream->width, buffer.stream->height,anwReleaseFence);
    }
}
void Camera3OutputStream::processYuv(
         ANativeWindowBuffer* anwBuffer, int width,int height,int fence) {
    ALOGE("%s: width: %d,height: %d,usage: %d", __FUNCTION__, width, height, usage);
    // Lock the image for CPU read
    sp<GraphicBuffer> graphicBuffer = GraphicBuffer::from(anwBuffer);
    void* mapped = nullptr;
    base::unique_fd fenceFd(dup(fence));
    status_t res = graphicBuffer->lockAsync(GraphicBuffer::USAGE_SW_READ_OFTEN, &mapped,
            fenceFd.get());
    if (res != OK) {
        ALOGE("%s: Failed to lock the buffer: %s (%d)", __FUNCTION__, strerror(-res), res);
        return;
    }
    //这个地方，mapped拿到的就是yuv数据,就可以把mapped送给算法进行处理.
    processImage((uint8_t*)mapped,width,height);
   //也可以把yuv数据保存到本地进行debug.
}

```

**二、opencv 在 Android 源码集成遇到的问题汇总**
----------------------------------

### **1、not specified in shared_libs**

【action】 这类错误，就是我们的目标文件，需要引用到的一些 so 库，但是我们的 bp 或者 mk 文件没有申明。

如下图所示，错误信息里面，也有 Fix suggestion，根据提示来做修改就好。

```
DT_NEEDED "liblog.so" is not specified in shared_libs
 Fix suggestions:
 Android.bp: shared_libs: ["libc++_shared", "libc", "libdl", "libjnigraphics", "liblog", "libm", "libmediandk", "libz"],
  Android.mk: LOCAL_SHARED_LIBRARIES := libc++_shared libc libdl libjnigraphics liblog libm libmediandk libz
  If the fix above doesn't work, bypass this check with:
  Android.bp: check_elf_files: false,
  Android.mk: LOCAL_CHECK_ELF_FILES := false

```

### **2、(native:vendor) can not link against libc++_shared (native:platform)**

【action】问题原因是，android8.0 之后，system 和 vendor 库进行了区分，除了一些特殊的 so 库，很多 so 库，比 vendor 模块的代码，不能去访问到 system 下的 so 库。vndk ?

```
vendor/mediatek/proprietary/hardware/mtkcam3/***/Android.mk: error: "libopencv_java4 (native:vendor) can not link against libc++_shared (native:platform)"

```

类似上面这种报错，如果是 vendor 下去访问 system 下的 so，我们可以找到 system 下该 so 源码的 Android.bp 文件，添加 vendor_available: true 来进行尝试。

```
diff --git a/frameworks/base/native/graphics/jni/Android.bp b/frameworks/base/native/graphics/jni/Android.bp
index 1709dfd973d..22432812430100644
--- a/frameworks/base/native/graphics/jni/Android.bp
+++ b/frameworks/base/native/graphics/jni/Android.bp
@@ -47,6 +47,9 @@ cc_library_shared {
     static_libs: ["libarect"],
+    
+    proprietary:true,
+

```

如果是涉及到多个 so 的嵌套使用，就会比较麻烦，如下面的，libjnigraphics 里面，会用到 libhwui，而 libhwui 也是不允许 vendor 去访问的，libhwui 里面，又会用到其它的 vendor 无法访问的 so。类似这种情况？关闭 vndk？

网友提供的一些方案（https://blog.csdn.net/qq_35345103/article/details/110390454），试了没能解决。

vndk 相关内容，可以查看官网介绍：

https://source.android.google.cn/docs/core/architecture/vndk/build-system?hl=ca

#### **3、error: cannot use 'throw' with exceptions disabled**

【action】

Android.bp 中添加如下内容可以解决：

#### **4、 error: use of ***requires -frtti**

【action】

```
android.bp中添加如下内容可以解决：
rtti: true,
(如果是cppflags中添加如下内容，会继续报该错误)
cppflags: [
        "-frtti",
    ],

```

#### **5、vndk 能否关闭掉，怎么关闭？**

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Alm9kic5yxKzbCUbyzzia3uppuxK3NKr1I76aiatliaLstXHNbrpfibuo1qgr3xp1PPGNYV0DskmdxDCVhibw1DNO9Sw/640?wx_fmt=jpeg&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz/ziadDDQxbCJFDq1MaoNclaJOpElczibia6UXtxCadkIvdKUiaLVkuVnhkx32A3JCM6I4howpH175pChoFaXn9gKSicA/640?wx_fmt=png)

《Android Camera 开发入门》、《Camx 初认识》已经上架，可以点击了解 -> [![](https://mmbiz.qpic.cn/sz_mmbiz_png/Alm9kic5yxKxsNyrgHDiaF0upjeOafQWDKjcgFe1RaxiboUpRgLZgs7uibtZanRI2Asq1U1gicnnBo81a9GqeXMh7ZA/640?wx_fmt=png&from=appmsg)](http://mp.weixin.qq.com/s?__biz=MzA3ODMzMTM1NA==&mid=2247489149&idx=1&sn=965199a48dac877b1fe8e436428f06a4&chksm=9f453ac8a832b3deb8b2289823e1e4a72198e2f1be91b48cfc173b888984745c1569f71bb5fa&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Alm9kic5yxKzDfrluP8iav8ZshiaRoicm5R21pbtyZK8QyR66bUI8Azngq9MzBqjGjETe7dkBFSvsQSaxW1a9uDibSQ/640?wx_fmt=png&from=appmsg)

**觉得不错，点个赞呗** ![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/v1.3.10/assets/Expression/Expression_67@2x.png)