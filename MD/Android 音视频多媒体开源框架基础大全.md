> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Hb_IcdTYd10QZaOSf7RTIQ)

![](https://mmbiz.qpic.cn/mmbiz_png/mpfXeUqqSEibNCC5dnzdJSeM0CHLsB4IzHBF473UhWIfsCibqgzqzwvf7okLaqwQBCsNPm2fVI3Vibteqsiaiapw5Cg/640?wx_fmt=png&from=appmsg)

**Android**

**音视频多媒体**

开源框架基础大全

安卓多媒体开发框架中，从音频采集，视频采集，到音视频处理，音视频播放显示分别有哪些常用的框架？分成六章，这里一次帮你总结完。

框架综述

音视频的主要流程是录制、处理、编解码和播放显示。本文也遵循这个流程展开，多媒体框架有视频驱动框架 V4L2/UVC，显示驱动框架 DRM，音视频处理框架 openMAX，播放框架 EXoplayer，以及显示渲染框架 VLC。

如何使用这些开源框架？

在一些大厂，一般和多媒体工程师和影像工程师进行区分，影像负责音视频的采集和处理；多媒体负责音视频的编解码和显示渲染；驱动和底层系统工程师负责提供底层的系统支持，以及框架内部的适配。

因为本人上下层都有涉猎，所以更加系统的把从多媒体的底层到上层，更为全面的角度审视这些多媒体框架。

更新说明

**有博友已经催更到朋友圈，所以做个说明：**

**最近同时面对工作和出差，身体抵抗力有所下降，每次写文章都忙到 12 点以后，考虑到健康状况，所以更新频率降低，恢复之后就继续安排上。因为文章皆为原创，同时为了保证质量，计划优先稳定在两周一更的频率。**

**01**

多媒体框架综述

![](https://mmbiz.qpic.cn/mmbiz_jpg/mpfXeUqqSEibNCC5dnzdJSeM0CHLsB4Iz7f7Udth8rRsWUU5h8o5IjGmribYvvMJZfCBOaQ3jt3aEetuLRYJt7Tg/640?wx_fmt=jpeg&from=appmsg)

个人觉得通过上层到应用侧，通过 JAVA API 可以串联整个音视频的流程，更容易进行理解。

**01**

核心服务架构

Android 的多媒体框架同样也遵循这些流程，音视频解析 / 解码 / 播放 / 录制相关核心服务架构和流程有:

MediaExtractor/MediaCodec/MediaRecorder/MediaPlayerService/libstagefright/OpenMAX 等，使用它们从而完成音视频的采集、处理、编码、封包、mp4 输出到最终的播放。

**02**

安卓相机的业务实现流程

在收集数据之前，对 Camera 设置一些参数；然后设置 PreviewCallback；之后获取到 Camera 的原始 NV21 数据；再创建一个 H264Encoder 类，在里面进行编码操作，并将编码后的数据存储到文件；结束编码的时候将相关的资源释放掉。这部分内容之后以高通为例，单独展开来说。

**03**

库和框架侧重不同

本文是对上一章《

Android 音视频多媒体开源库基础大全

》的补充。  

库是一组可以重复使用的代码，提供了一些特定的功能，程序员可以在自己的程序中调用这些功能。比如在音视频多媒体系统开发中，库可能只负责音视频的编解码（如 FFmpeg）、音视频处理（如 OpenCV）或视频渲染（如 OpenGL ES）等某个特定的功能模块。

框架则是提供了一整套的架构和机制，通过提供了大量的预定义代码和可复用组件‌从而实现一个完整的解决方案，框架会负责从音视频数据的获取、解码、处理到最终的显示渲染等整个流程的管理和组织。程序员是在这个框架的基础上实现自己的应用逻辑。

**02**

OpenMAX 多媒体框架

![](https://mmbiz.qpic.cn/mmbiz_jpg/mpfXeUqqSEibNCC5dnzdJSeM0CHLsB4IzqVjyhYLzTOUp2k1MJJVsh6kXXWZz9micZ0LB8CDRlT4HFjqYR1KTd8A/640?wx_fmt=jpeg&from=appmsg)

OpenMAX（OMX）一个多媒体应用程序的框架标准，Android 系统默认已经集成。

框架的特点是提供标准化的音视频处理 API 接口，便于集成和扩展；

因为其跨平台和标准化特点，在需要优化视频处理性能的时候，最常应用于嵌入式系统和移动设备中的音视频处理中。

程序员通过调用 OpenMAX API 来管理编解码器、设置编解码参数、发送编解码命令等。也可以发送音视频数据到编解码器进行处理，并接收处理后的数据。这些 API 通常通过 JNI 在 Java 层和 Native 层之间进行调用。

**01**

框架分层

OMX 的三个主要层次是 DL（开发层）、IL（集成层）和 AL（应用层）。

IL 层在 Android 中是最常用的，IL 层满足了操作系统到硬件的差异和多媒体应用的差异，IL 提供了与硬件编解码器交互的接口。

硬件多媒体编解码框架，定义标准化接口对接 DSP/GPU 加速组件，提升编解码效率，如 MediaCodec 底层实现依赖此标准。

IL 层主要实现了各种组件，组件是 IL 层实现的核心内容，通过输入端口消耗 buffer，通过输出端口填充 buffer，由此多组件相连接可以构成流式处理。

IL 层对上层给 AL 层等框架层调用，也可以给应用程序直接调用；对下层可以调用 DL 层的接口，也可也直接调用各种 codec 实现。

OpenMAX IL 层接口示例：

（1）组件加载与卸载接口：OMX_GetHandle、OMX_FreeHandle

（2）组件控制与配置接口：OMX_SetParameter、OMX_GetParameter、OMX_SendCommand

（3）组件间通信接口：OMX_SetupTunnel、OMX_UseBuffer

**02**

使用流程

使用 OpenMAX 框架音视频开发的流程，和整体安卓多媒体的开发流程一致，主要包括以下几点：

（1）创建 OpenMAX 组件，使用 OpenMAX IL API 创建编解码器、源、解复用器等组件的实例。

（2）配置组件，设置组件的参数，如输入输出格式、缓冲区大小等。

（3）连接组件将不同的组件通过隧道化方式连接起来，形成处理流。

（4）发送数据，将音视频数据发送到编解码器进行处理。

（5）接收和处理结果，接收编解码器处理后的数据，并进行后续处理（如渲染、存储等）。

（6）资源释放，在处理完成后，释放组件和相关资源。

**03**

在安卓系统应用

openmaxIL 可以以插件的形式加入到 android 系统中，从而可以再 Android 系统的编解码开发、播放器组件以及中间件开发中都发挥作用。作为安卓系统的多媒体开发人员，对此毫不陌生。

（1）编解码器开发和管理。

通过 OpenMAX IL 层，音视频和图像编解码器能够与多媒体框架进行交互，提供统一的接口来支持编解码功能。Android 系统通过 OpenMAX IL 层封装了硬件编解码器的功能，使得上层应用能够透明地利用硬件加速能力进行音视频编解码，从而提高接口处理效率。

（2）构成播放器组件。

在 Android 中，如 NuPlayer 或 AwesomePlayer 等播放器框架就使用了 OpenMAX 来管理编解码器、源、输出、解复用器 demux 等组件。

使用 OpenMAX 的组件化设计，播放器可以灵活地组合不同的模块，灵活实现音视频数据的解码、渲染和输出。

（3）实现多媒体中间件接口。

OpenMAX AL 层提供了应用程序和多媒体中间件之间的标准化接口。应用开发着只需关注标准化的功能实现，差异化交给下层。

在 Android 中这有助于多媒体中间件与上层应用之间的集成，不同的厂商都热衷于设计和完成自己的中间件，比如编解码器库、图形库或者平台库等。

**03**

音视频采集框架

V4L2（Video for Linux 2）在 Android 音视频开发中的应用主要集中在视频采集和处理方面。V4L2 是 Linux 内核中用于视频设备的一个 API 框架，旨在提供对视频捕捉、输出和处理设备的支持。

V4L2 和 UVC 的驱动基础我们在之前的文章中已经分享过，本文更加关注框架层应用。

https://mp.weixin.qq.com/s/puaNQrfn1KFNe0uSN9SmKQ

[https://mp.weixin.qq.com/s/JG2XDjI1E2jEU6fDlQZCCw](https://mp.weixin.qq.com/s?__biz=MzkxMTUyNTkyNw==&mid=2247483877&idx=1&sn=7ae116d9c8a21e3418eef7f8a8601a73&scene=21#wechat_redirect)

**01**

V4L2 的使用

在 linux 中使用 V4L2，使用 open() 函数打开视频设备文件 / dev/videoX，使用 ioctl() 函数进行设置和处理视频帧。

在 Android 中使用 V4L2，只需要在内核配置菜单中选择 Video capture adapters”，确保 V4L2 驱动被选中并编译进内核，之后按照 V4L2 提供的 API 来访问和控制视频设备，使用 ioctl 等系统调用设置和处理视频帧。使用 v4l2-ctl 等命令行工具来调试和测试 V4L2 设备信息。通过 logcat 等工具来查看调试信息。

在驱动开发中，V4L2 可以作为摄像头、视频采集卡等设备的驱动框架。驱动工程师根据 V4L2 的规范来编写或修改驱动程序，使设备能够在 Android 系统上正常工作。

在系统开发中，通过 V4L2 以直接与 Android 设备上的摄像头硬件进行交互，实现视频帧的捕获。系统工程师可以使用 V4L2 接口来开发特定的视频采集应用，如安防监控软件、实时视频流传输软件等。

在应用开发中，比如视频编辑软件、实时视频特效软件时，可以通过 V4L2 与 GPU 或其他硬件加速技术结合使用，提高处理效率。

**02**

UVC 的使用

在 Linux 音视频开发中，UVC 是 USB 视频类驱动框架，标准化 USB 摄像头在 Linux 内核中的协议实现‌，主要用在处理 USB 摄像头和视频输入设备时。在 Linux 系统中，UVC 设备通常会被识别为 / dev/videoX（X 为设备编号）的文件。

使用 UVC 简化了视频采集的流程，Linux 系统能够直接识别和使用 USB 摄像头，而无需安装额外的驱动程序。同样 UVC 摄像头通常支持多种视频格式和分辨率，所以 UVC 更容易应用于快捷视频接入的场景。

在视频监控中，UVC 摄像头可以通过 Linux 系统实现视频的捕获、编码和传输，满足远程监控和管理的需求。

在视频会议和实时通讯应用中，UVC 摄像头能够提供清晰的视频画面，支持语音通话和视频通话。

**03**

音频采集‌Oboe‌

Android SDK 提供了两套音频采集的 API 接口：

 MediaRecorder 和 AudioRecord。

而 Android NDK 则可以通过 OpenSL ES 、AAudio 和 Oboe 完成音频采集。

Oboe 是 Android 团队开发的 C++ 音频框架，封装 AAudio 和 OpenSL ES，支持高性能、低延迟音频采集与播放，用于构建高性能的音频应用程序，比如实时音频分析工具 Audio Analyze 就是使用的该框架。

更多资料：

https://github.com/google/oboe/blob/main/docs/FullGuide.md

https://github.com/googlearchive/android-audio-high-performance/

**04**

显示驱动框架

![](https://mmbiz.qpic.cn/mmbiz_jpg/mpfXeUqqSEibNCC5dnzdJSeM0CHLsB4IzRJ1989oSAyAn8esWTp99mGC9t1v86t6Pp1SVFIaufReztNJacErfBw/640?wx_fmt=jpeg&from=appmsg)

DRM（Direct Rendering Manager）是 Linux 内核显示渲染框架，管理 GPU 资源分配与多屏合成调度。Linux DRM 聚焦图形硬件的高效利用，而 Android DRM 侧重内容安全。

DRM 的驱动基础我们在之前的文章中已经分享过，本文更加关注框架层应用。

Linux 系统中的 DRM 框架 **--------**

DRM 是 Linux 内核中管理 GPU 的核心框架，其开发使用可分为：内核层和应用层开发。

最终的应用也比较广泛，在嵌入式系统，DRM 驱动实现多屏显示（如 HDMI+LVDS）；图形服务器中，利用 DRM 直接访问 GPU 进行窗口合成，避免 X Server 的间接渲染开销。

（1）内核层开发，通过 drm_driver 结构体注册 GPU 驱动，实现 drm_crtc（显示控制器）、drm_plane（图层）、drm_framebuffer（帧缓冲区）等核心对象的生命周期管理。

（2）内核层开发，使用 KMS（Kernel Mode Setting）API 进行显示模式配置（如分辨率、刷新率），通过 GEM（Graphics Execution Manager）管理显存分配与跨进程共享。

（3）内核层开发，调试工具链包括：modetest：测试显示模式与帧缓冲操作；drm_info：查询 GPU 设备信息；apitrace：跟踪 OpenGL/Vulkan 调用。

（4）用户层开发，则通过 libdrm 库调用 ioctl 接口，或者结合 GBM（Generic Buffer Management）或 EGL 实现 GPU 加速渲染。

Android 系统中的 DRM 框架 **------**

Android 的 DRM 框架主要用于数字版权保护，包括 DRM 引擎和播放器集成，比如 ExoPlayer 通过 DefaultDrmSessionManager 与 DRM 服务器通信。

最终的应用有流媒体保护和企业应用，比如 Netflix 使用 Widevine L1 级保护 4K 内容，密钥通过安全通道传输；Android for Work 通过 DRM 限制企业数据访问权限，防止数据泄露。

**05**

多媒体播放框架

Android 媒体框架通过 MediaPlayer 类来控制音视频的播放，MediaPlayer 会将解码后的视频帧输出到指定的 Surface 上进行显示。而具体的显示渲染则依赖于 SurfaceView 或 TextureView 等视图组件。

综合来说，Stagefright 作为早期框架已被取代，NuPlayer 集成于系统但扩展性有限，ExoPlayer 则是开发的推荐选择，尤其在需要流媒体支持和高度定制的场景。

5.1Stagefright 和 NuPlayer**----**

Stagefright 和 NuPlayer 是 Android 系统中用于多媒体编解码与播放的底层框架，主要处理音频、视频的解析、解码和渲染。它通过 OpenMAX 接口与硬件编解码器交互，支持 H.264、MP3 等主流格式，Stagefright 应用在 Android 2.0 至 5.1 版本，Android 5.1 后被 NuPlayer 取代。

关键组件中，AwesomePlayer 是核心播放器模块，管理音视频同步与渲染。MediaExtractor 负责解析媒体文件，分离音视频流。OMXCodec 则基于 OpenMAX 的编解码器，支持硬件加速。

Android 中的 Stagefright 框架，包括 Video Playback 流程、与 OpenMAX 的运作、选择 Video Decoder、Video Buffer 传输、Video Rendering 以及 Audio Playback 流程。Stagefright 使用 OpenMAX 进行编解码，并通过 AudioPlayer 处理 audio 部分，实现 Video 与 Audio 的同步。

<table><thead><tr><th><section><br></section></th><th>Stagefright</th><th>NuPlayer</th></tr></thead><tbody><tr><td>API</td><td>通过<code>MediaPlayer</code>类调用，</td><td>通过<code>MediaPlayer</code>类调用</td></tr><tr><td>使用</td><td>直接使用<code>MediaPlayer</code>或<code>MediaCodec</code>等高级 API，底层由 Stagefright 处理编解码与渲染</td><td>基于<code>StagefrightPlayer</code>基础类开发</td></tr><tr><td>流媒体支持</td><td>仅本地播放</td><td>支持 HTTP Live、RTSP 等流媒体协议</td></tr></tbody></table>

5.2ExoPlayer**--------**

ExoPlayer 是谷歌开源播放框架，支持自定义渲染逻辑、自适应流（DASH/HLS）及 DRM 加密播放，可扩展性强于 MediaPlayer，ExoPlayer 是构建在 MediaPlayer 之上的库，用于替代 Android 原生的 MediaPlayer，提供了更强大的功能和性能，当然也更复杂。

**06**

显示渲染框架

在音视频多媒体系统开发中，显示和渲染框架通常是指那些提供了一整套架构和机制，这些框架通常会包含多个模块，负责从音视频数据的解码、处理到最终的显示，显示和渲染基本在一个框架中实现。

**01**

VLC 和 SDL

VLC 是一个非常流行的开源多媒体播放器，它背后也有一个强大的多媒体处理框架。

VLC 框架内部有自己的一套机制来处理音视频的显示和渲染。它支持多种视频输出模块，可以根据不同的平台和需求选择合适的输出方式，例如在 Windows 上可以使用 Direct3D 进行视频渲染，在 Linux 上可以使用 X11 等。

FFmpeg +SDL 构成一个简单的显示和渲染框架。

FFmpeg 负责音视频的解码等工作，而 SDL （Simple DirectMedia Layer）则负责将解码后的视频帧渲染到屏幕上。SDL 提供了跨平台的多媒体处理功能，包括音频播放和视频显示。

这种组合比较轻量级，适合一些对性能要求不是特别高，但需要快速实现音视频播放和显示的场景。由于 FFmpeg 和 SDL 都是开源的，也具有很好的可定制性。

**02**

渲染视图组件

组件的使用，通过使用 MediaPlayer 类加载音视频文件；通过 SurfaceView 或 TextureView 将视频帧渲染到屏幕上，ImageView 则用于显示静态图像；使用 MediaCodec 进行音视频的解码，然后通过 OpenGL ES 或其他图形库进行自定义渲染。ExoPlayer 更是支持自定义渲染器，可以通过实现 Renderer 接口来实现自定义的音视频渲染逻辑。GLSurfaceView 则用于实现复杂的 3D 图形渲染比如游戏、AR 等场景。

文后总结

![](https://mmbiz.qpic.cn/mmbiz_jpg/mpfXeUqqSEibNCC5dnzdJSeM0CHLsB4IzOBW4xUH4xKdgiaPPyKn9MslqKzibx9B2t5ibbx4qrq0epO3gdHENbxrwg/640?wx_fmt=jpeg&from=appmsg)

**赞和关注 ---------**

看到这里还不帮忙点个赞和关注，十分感谢！

本文根据博主工作经验，汇总了影像系统从上到下，程序员都会用到的哪些开源框架。

掌握这些，有助于更好的分析问题以及查漏补缺，成为多媒体领域的全栈工程师。

**挖个坑 ---------**

本文是开源方案第二章，流媒体框架 GStreamer 之后单独一张补充，主要原因是做流媒体这一块的核心业务和流程和本地多媒体有蛮多差异。

本文只做简单的梳理，如有学习需要可以参考其更多资料。

原创不易，身体刚好，持续更新，觉得有用，欢迎点赞支持。

![](https://mmbiz.qpic.cn/mmbiz_jpg/mpfXeUqqSEicrP4670V1qRfL82xHxJm8Na03h47YnQJaib96xfHWL9v54jQ3HXvfHBa2FoPbQ9gCC095FMc8CyvQ/640?wx_fmt=jpeg&from=appmsg)