# VDI 远程协议的一些解释

# 桌面协议

虚拟桌面架构（VDI）中，协议是非常关键的一环，其定义了将服务器虚拟出的客户机系统从服务器传输到各类终端的规则，涉及到安全，图像处理，数据压缩，网络传输协议等多个方面，直接决定着虚拟桌面的终端体验。

常用的虚拟桌面协议有：

1. RDP(remote desktop protocol)协议：远程桌面协议，大部分 Windows 系统都默认支持此协议。Windows系统中的远程桌面管理就是基于该协议的。据说也是由思杰开发，支持的功能较少，且主要应用在windows环境中，现在也有[Mac下的RDP客户端](http://www.microsoft.com/mac/remote-desktop-client)和linux下的RDP客户端[rdesktop](http://www.rdesktop.org/). 历经多个版本的开发，RDP最新版也支持了打印机重定向，音频重定向，剪贴板共享等功能。

   RemoteFX是RDP的增强版，提供了vGPU,视频支持，多点触摸，USB重定向等功能。 

2. RFB(Remote FrameBuffer)协议：图形远程管理协议，**VNC**远程管理工具就是基于此协议。

3. SPICE，Simple Protocol for Independent Computing Environment（独立计算环境简单协议）是红帽企业虚拟化桌面版的主要技术组件之一，具有自适应能力的远程提交协议，能够提供与物理桌面完全相同的最终用户体验。借助支持SPICE协议的客户端（如remote-viewer）或者通过浏览器，用户可以访问自己的虚拟云桌面。

4. ICA/HDX：Citrix Independent Computing Architecture （ ICA ）应该是目前最成熟的虚拟桌面协议, 思杰十几年的耕耘使其难以在短时间内被完全超越。ICA除了功能齐全（支持windows areo）之外，还有广泛的移动端支持。ICA的网络协议无关性，使其可以支持TCP/IP 、 NetBIOS 和 IPX/SPX 。ICA不仅支持自家的虚拟化平台XenServer，还支持vSphere和Hyper-V。性能上比较突出的特点是较低的带宽占用，在网络环境差（延迟高）的情况下也能正常使用

   HDX（High Definition Experience）作为ICA的增强版，着力于改善用户体验，包括音视频，多媒体和3D，HDX支持H.264.

5. PCoIP最初由加拿大公司Teradici开发，早期定位于高端图形设计，2008年VMware宣布与Teradici共同开发PCoIP，以改进自己的VDI解决方案VMware View。

6. 其他协议

   1.  [THINC](http://systems.cs.columbia.edu/projects/thinc/)（Thin-Client Internet Computing）是哥伦比亚大学的一个研究项目，不太成熟，只找到一篇论文，可以拿来做学习研究用。

   2. X11 [X windows system](http://en.wikipedia.org/wiki/X_Window_System) 最初设计的时候就是client server的架构方式，因此也能远程访问主机的桌面。

   3. ALP：Sun公司在1999年就推出了一款瘦终端产品Sun Ray，其采用的协议为Sun公司开发的ALP（Appliance Link Protocol），VMware view也支持该协议，不过最近Oracle宣布终止Sun Ray的开发。

   4. EOP(Experience Optimization Protocol)协议是Quest Software的VDI协议，用在其自家的VDI产品vWorkspace（[收购了Provision-Networks](http://www.brianmadden.com/blogs/brianmadden/archive/2007/11/12/quest-software-buys-provision-networks-finally-a-real-challenger-to-citrix-in-the-app-delivery-space.aspx)）中，Quest没有自己的hypervisor，所以vWorkspace支持Citrix VMware 微软等的hypervisor。Quest Software已经被DELL收购。

   5. CHP，号称宇宙无敌最强协议，直接把ICA RDP PCoIP虐出xiang的国产CHP，不说了 上图。

      ![虚拟桌面协议 - songtianyi - songtianyi](https://songtianyi.info/images/004-vdi-03.png)

以上参考 https://songtianyi.info/pages/vdi/004-vdi.html

spice/vnc/rdp, 三者的对比如下：

|              | SPICE                     | VNC          | RDP                 |
| ------------ | ------------------------- | ------------ | ------------------- |
| BIOS屏幕显示 | 能                        | 能           | 不能                |
| 全彩支持     | 能                        | 能           | 能                  |
| 更改分辨率   | 能                        | 能           | 能                  |
| 多显示器     | 多显示器支持（高达4画面） | 只有一个屏幕 | 多显示器支持        |
| 图像传输     | 图像和图形传输            | 图像传输     | 图像和图形传输      |
| 视频播放支持 | GPU加速支持               | 不能         | GPU加速支持         |
| 音频传输     | 双向语音可以控制          | 不能         | 双向语音可以控制    |
| 鼠标控制     | 客户端服务器都可以控制    | 服务器端控制 | 服务器端控制        |
| USB传输      | USB可以通过网络传输       | 不能         | USB可以通过网络传输 |

## spice

SPICE架构包括客户端、SPICE服务端和相应的QXL设备、QXL驱动等，如下图所示。客户端运行在用户终端设备上，为用户提供桌面环境。SPICE服务端以动态连接库的形式与KVM虚拟机整合，通过SPICE协议与客户端进行通信。

![这里写图片描述](https://img-blog.csdn.net/20170803114014696?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGlhbmd4aWFuZ2hlaGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Spice agent运行在客户机（虚拟机）操作系统中。Spice server和Spice client利用spice agent来执行一些需要在虚拟机里执行的任务，如配置分辨率，另外还有通过剪贴板来拷贝文件等。从上图可以看出，Spice client与server与Spice Agent的通信需要借助一些其他的软件模块，如在客户机里面，Spice Agent需要通过VDIPort Driver与主机上 QEMU的VDIPort Device进行交互，他们的交互通过一种叫做输入/输出的环进行。Spice Client和Server产生的消息被写入到设备的输出环中，由VDI Port Driver读取；而Spice Agent发出的消息则通过VDI Port Driver先写入到VDI Port Device输入环中,被QEMU读入到Spice server的缓冲区中，然后再根据消息决定由Spice Server直接处理，还是被发往Spice Client中。

![这里写图片描述](https://img-blog.csdn.net/20170803114616306?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGlhbmd4aWFuZ2hlaGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

SPICE协议最大的特点是其架构中增加的位于Hypervisor中的QXL设备，本质上是KVM虚拟化平台中通过软件实现的PCI显示设备，利用循环队列等数据结构供虚拟化平台上的多个虚拟机共享实现了设备的虚拟化。但是，这种架构使得SPICE协议紧密地依赖于服务器虚拟化软／硬件基础设施，SPICE必须与KVM虚拟化环境绑定。传统的远程桌面传输协议工作在虚拟机Guest OS中，而SPICE协议本身运行在虚拟机服务器中，可以直接使用服务器的硬件资源。

其实KVM虚拟化桌面进行云端访问时也是可以使用VNC通道进行数据传输的，然而VNC虽然通用但是效率比起SPICE还是差的远，资源消耗也是非常大，如果你是用KVM-QEMU虚拟机的话，进行云端桌面访问建议使用SPICE协议。

**SPICE协议特点**

- 和低端(low end)的虚拟桌面协议（如RFB）不同，SPICE协议虚拟出一个图形处理设备QXL，专门用于处理客户机的图形命令（graphic commands）。
- SPICE用不同的信道来传输键盘，鼠标，视频图像和音频等，便于针对性地优化。
- SPICE首先尝试将渲染的工作交给客户端（瘦终端），利用客户端硬件资源来加速，之后会将渲染工作交给主机来处理，这时可以用软件或者GPU来处理。
- SPICE客户端支持linux，windows和[Mac](http://spice-space.org/page/OSX_Client)

 **SPICE协议目前已有的功能**

- 视频/图像压缩，基于MPEG的视频压缩和基于SFALIC，Lempel–Ziv的图像压缩
- 客户端缓存，对图像 调色板 光标进行缓存处理
- 热迁移，虚拟机从当前主机（Host）迁移到另外一个主机时spice的连接不会中断
- 多屏显示，最多支持四个屏幕
- 音频的播放和录制，音频也可以压缩传输
- 加密传输，支持openssl
- 剪贴板共享，瘦终端系统（client OS）和客户机系统（guest OS）可以相互拷贝粘贴
- USB重定向，将瘦终端的USB设备重定向到客户机
- smartcard，支持智能卡登录

## vnc

 VNC (Virtual Network Console)，虚拟网络控制台。VNC系统由客户端，服务端和一个协议组成。VNC的服务端目的是分享其所运行机器的屏幕， 服务端被动的允许客户端控制它。 

VNC客户端（或Viewer） 观察控制服务端，与服务端交互。 VNC 协议 Protocol (RFB)是一个简单的协议，传送服务端的原始图像到客户端（一个X,Y 位置上的正方形的点阵数据）， 客户端传送事件消息到服务端。服务器发送小方块的帧缓存给客户端，在最简单的情况，VNC协议使用大量的带宽，因此各种各样的方法被发明出来减少通讯的开支，举例来说，有各种各样的编码方法来决定最有效率的方法来传送这些点阵方块。协议允许客户端和服务端去协议哪种编码会被使用，最简单的编码，被大多数客户端和服务端所支持的是， 从左到右的像素扫描数据的原始编码， 当原始的满屏被发送后，只发送变化的方块区域。这种编码在帧间只有小部分屏幕变化的情况下工作的非常好（像是鼠标键在桌面移动的情况，或在光标处敲击文字），不过如果大量的像素同时变化带宽将会增加的非常高，像是拖动一个窗口或观看全屏录像。

[noVNC](https://novnc.com/info.html) 是一个 HTML5 的 VNC 客户端，采用 JavaScript 编程实现。其主要功能是和远端的 VNC Server 互通，通过对 [RFB（Remote Frame Buffer）协议](https://en.wikipedia.org/wiki/RFB_protocol) 数据的编解码，一方面接收远端 VNC Server 发送的数据，解码后通过 [Canvas](https://en.wikipedia.org/wiki/Canvas_element) 技术绘制在客户端侧，另一方面将客户端侧的终端输入编码成 RFB 数据发送给远端的 VNC Server。也就是说有了 noVNC，我们可以直接使用支持 HTML5 的 Web 浏览器，譬如 Chrome，就可以访问远端的安装了 VNC Server 的机器的桌面，而不用另外安装原生的 VNC 客户端（譬如我们经常在 Windows 上安装的 [RealVNC Viewer](https://www.realvnc.com/download/viewer/)）。

但需要注意的是，具体传输时，和原生的 VNC 客户端不同，原生的客户端的 RFB 数据是直接承载在 TCP （Raw TCP）上，而 noVNC 处理的 RFB 数据是承载在 Websocket 之上。由于目前大多数 VNC 服务器都不支持 WebSockets，所以 noVNC 是不能直接连接 VNC 服务器的，怎么办呢？这就需要一个代理来实现 Websockets 和 Raw TCP 之间的转换，这个代理就是 Websockify。

一个典型的从客户端浏览器到 VNC 服务器，中间经过 Websockify 转换的网络如下，注意其中 noVNC 作为一个 HTML5 的客户端，虽然一开始是存放在 websockify 所在的代理服务器上，但其主体 js 代码会在客户端浏览器访问 websockify 服务时被下载到客户端的浏览器中执行。

![img](http://tinylab.org/wp-content/uploads/2019/09/novnc-guide/network.png)

参考http://tinylab.org/guide-to-novnc/

# spice图像处理

## 名词解释

- JPEG/MJPEG(MotionJPEG)

  - JPEG是一种静止图像的压缩标准，它是一种标准的帧内压缩编码方式。当硬件处理速度足够快时，JPEG能用于实时动图像的视频压缩。在画面变动较小的情况下能提供相当不错的图像质量，传输速度快，使用相当安全，缺点是数据量较大。
  - M-JPEG源于JPEG压缩技术，是一种简单的帧内JPEG压缩，压缩图像质量较好，在画面变动情况下无马赛克，但是由于这种压缩本身技术限制，无法做到大比例压缩，录像时每小时约1-2GB空间，网络传输时需要2M带宽，所以无论录像或网络发送传输，都将耗费大量的硬盘容量和带宽，不适合长时间连续录像的需求，不大实用于视频图像的网络传输。

  MJPEG是spice默认算法，它只单独的对某一帧进行压缩（帧内压缩），而基本不考虑视频流中不同帧之间的变化。通过此压缩技术可获取清晰度很高的视频图像，而且可灵活设置每路的视频清晰度和压缩帧数，并且压缩后的画面还可任意剪接。但它的缺陷也非常明显：

  - 丢帧现象严重、实时性差，在保证多台瘦客户机都必须是高清晰的前提下，很难完成实时压缩，通常用做单台虚拟机测试播放1080P 高清视频；
  - 压缩效率低，传输带宽和存储空间占用大

- H.264是 ITU-T和ISO共同成立的JVT联合视频工作组制定的新一代视频编码标准，用来实现视频的高压缩比、高图像质量、良好的网络适应性等目标。H.264 不仅比MJPEG节约了80%以上的码率，而且对网络传输具有更好的支持功能。H.264引入了面向IP包的编码机制，有利于网络中的分组传输，支持网络中视频的流媒体传输，支持不同网络资源下的分级编码传输，从而获得平稳的图像质量。H.264可以在更低的带宽下实现720p、 1080i/p的广播级高清视频分辨率。

  尽管MJPEG能够获得比较好的单幅图像质量，但由于它在运动性、带宽占用方面均有致命缺陷，所以更需要H.264视频编码方式来满足低带宽，实时多并发连接的桌面虚拟化使用场境.

- h.265是H.264的升级版,H.265标准保留H.264原来的某些技术，同时对一些相关的技术加以改进。新技术使用先进的技术用以改善码流、编码质量、延时和算法复杂度之间的关系，达到最优化设置；

  具体的研究内容包括：提高压缩效率、提高鲁棒性和错误恢复能力、减少实时的时延、减少信道获取时间和随机接入时延、降低复杂度等。H264由于算法优化，可以低于1Mbps的速度实现标清数字图像传送；H265则可以实现利用1~2Mbps的传输速度传送720P（分辨率1280*720）普通高清音视频传送。H.265旨在在有限带宽下传输更高质量的网络视频，仅需原先的一半带宽即可播放相同质量的视频。这也意味着，我们的智能手机、平板机等移动设备将能够直接在线播放1080p的全高清视频。H.265标准也同时支持4K(4096×2160)和8K(8192×4320)超高清视频。可以说，H.265标准让网络视频跟上了显示屏“高分辨率化”的脚步。

- MPEG是ISO与IEC于1988年成立的专门针对运动图像和语音压缩制定国际标准的组织。MPEG标准主要有以下五个，[MPEG-1](https://baike.baidu.com/item/MPEG-1)、MPEG-2、[MPEG-4](https://baike.baidu.com/item/MPEG-4)、MPEG-7及[MPEG-21](https://baike.baidu.com/item/MPEG-21)等。当年的dvd碟片就是用的mpeg-2视频编码，广电mpeg-2用的比较普遍。

- YUV420和YUV444. RGB是构成多种颜色的三基色（红绿蓝），也称为加成色。主要是图像的采集和显示。YUV是优化彩色视频信号的编码和传输，和rgb相比，YUV占用的带宽少。YUV中Y表示的是亮度，UV表示的是色度，定义了颜色的两个方面的色度和饱和度。YUV分为YUV444，YUV422，YUV420等，含有不同色度分量的编码方式。每个部分用8位或者1个字节来标识。由于YUV444没有色彩二次采样的问题，也就是说我们不会从原图像中去除任何色彩信息，这样YUV444就会有完整的24bits。但是对于YUV420来说，由于我们移除了一半的水平和垂直色彩信息，以便降低带宽的使用，这最终导致了我们只用了12bits来代表YUV420，仍然是8bits的亮度，但是只有4bits而不是16bits的色彩度。你可以查看此网站(https://en.wikipedia.org/wiki/Chroma_subsampling#4:4:4) 了解更多的采样系统.人类肉眼通常对于光线或者亮度比较敏感，所以无论YUV420还是YUV444都保证了亮度，但是色彩因人而异，不同的人对于不同的色彩的理解有一定的差异，同时在高帧率（high frame per second）的场景下，您往往无法很快察觉到颜色的差异，但是亮度差异会更加明显。所以，YUV420是采用了一种折中的方式来平衡显示效果和资源占用。

  - 对于色彩准确度上，YUV444相比YUV420 有很大的改善
  - UV444每像素24比特，YUV420每像素仅12比特，所以YUV444的带宽消耗比YUV420高出两倍左右
  - YUV444比YUV420使用硬件编码（NVENC）情况下略有延迟（编码量更多）
  - 基于Linux的瘦客户机可能不支持YUV444

  **没有最好，只有最合适，我们需要在不同场景中进行平衡的选择。建议在用户对颜色准确性敏感的3D VDI使用场景中使用YUV444。**



## 图像处理

VDI传输的图像画面有2种处理方式：图片和视频。

- 图片场景

  如果桌面的屏幕大多数没有变化，那么传输的图像完全可以是一帧帧的图片，比如用户使用office和erp应用等。此时可以使用例如Citrix Bitmap remoting（位图远程控制），基于JPG 压缩和RLE（Rung Length Encoding）技术。位图远程（Bitmap Remoting），也叫做“Thinwire”是一种针对静态内容“Static Content”可以高效利用带宽的远程控制协议。当然，在spice协议中我们可以使用无损压缩来得到同样的体验（无损压缩是在不丢失任何信息的情况下将数据压缩更小，算法有lz4/glz）。

  - 对资源消耗非常低，不需要特别的终端硬件配置
  - 使用cpu进行位图压缩编码
  - 不能使用GPU编码

- 视频场景

  位图（Bitmap Remoting）对于静态图像支持的非常好，但是当涉及到移动图像和视频播放时，情况则有所不同了。当帧率越高（30fps，60fps），就有越多的图像需要被位图远程控制(Bitmap Remoting)所传输，这将严重影响带宽需求和总体的使用体验。所以，这种情况下视频编解码正好有所作用。视频编解码的技术有JPEG/M-JPE、H.264/H265和MPEG等，结合YUV方案分别有如下的视频场景下的技术方案：

  | 技术方案                       | 带宽 | 延时 | GPU加速             | 硬件要求                  | 图像质量（SSIM） | 适用场景                                                     |
  | ------------------------------ | ---- | ---- | ------------------- | ------------------------- | ---------------- | ------------------------------------------------------------ |
  | H.264 YUV420                   | 中   | 低   | 支持                |                           | 0.8310           | 一般场景                                                     |
  | H.264 YUV444                   | 高   | 中   | 支持                | Citrix linux终端不支持    | 0.9836           | 对颜色敏感，带宽会增加                                       |
  | H.265 YUV420                   | 低   | 低   | 支持                | 需要指定的CPU/GPU以及终端 | 0.8311           | 节约带宽                                                     |
  | H.265 YUV444                   | /    | /    | 少部分专业GPU卡支持 | /                         | /                | /                                                            |
  | Citrix 无损构建（H.264/H.265） | 极低 | 低   | 支持                |                           | 0.9999           | 一般办公（office 等）,不适合2D/3D设计等(过程画面有损，最终清晰，锐化现象) |
  | Citrix 混合编码                | 极低 | 高   | 视频编码时支持      |                           | 0.9999           | 结合了位图编解码和视频编解码优点。热点区域使用视频编码，非热点使用位图。无法将硬件编码用于位图编解码所以延迟大。 |

## xView player的参数设置

在xView player使用中，和图像传输相关的主要参数为`--spice-video-param={desktop_type,quality_level,encode_type,fps,CRF,VBR Min,VBR Max,CBR,QP}`总共9个参数，具体可以参看[xView player参数说明]( https://www.tapd.cn/56574401/markdown_wikis/show/#1156574401001000231) （yuv 444计划再添加第10个参数（或者扩展encode_type），1或者0,1代表yuv 444）.

1. 画面质量为无损。等同于上一节提到的Citrix Bitmap。此时将不再关注除了desktop_type,quality_level之外的参数，并且添加无损压缩算法参数`--spice-prefered-compression=glz`
2. 非GPU桌面，画面质量为高中低以及自定义，可以设置详细的参数来控制画面传输，包括h264/h265
3. 非GPU桌面使用CPU来进行图像编解码，也可以在服务器上调用GPU，加速编码，节省CPU资源
4. GPU直通、vGPU桌面，是由运行在虚机桌面里的agent调用GPU来进行编码；xview-player在客户端使用CPU或者本机GPU进行解码
5. 混合编码，动态调整，xView 应该还是没有实现的。



 

