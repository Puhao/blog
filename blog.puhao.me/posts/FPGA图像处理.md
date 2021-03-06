---
title: FPGA图像处理
date: '2013-08-24'
description:
categories:
-FPGA
-技术
tags:
-FPGA
-Verilog
-图像处理
-System Generator
-Vivado
-HLS
-Simulink

---
突然想写篇这方面的blog，主要是想着把N年前写的blog也汇总到这里来。以前曾经写过[FPGA视频采集](http://www.openhw.org/sain_1989/blog/12-05/279347_f3591.html)和[FPGA存储控制器](http://www.openhw.org/sain_1989/blog/12-07/281798_e0f79.html#articletop),这正好涉及到了视频图像的采集，存储，应该把处理的尾巴给续上去。这儿不涉及到具体的技术细节，只是谈下整个设计流程，更像一篇科普文章。

图像获取
-----
[FPGA视频采集](http://www.openhw.org/sain_1989/blog/12-05/279347_f3591.html)里面主要是提了一下如何获取PAL视频源的问题，通过AD转换芯片和时序逻辑组合获取原始像素。我们也可以直接从摄像头获取视频图像，以OV7670为例。    
这是一款非常主流的摄像头，被用于很多嵌入式设备中。它支持标准的SCCB接口，兼容I2C接口，而且提供丰富的图像格式输出，包括RawRGB,RGB(GRB4:2:2,RGB565/555/444),YUV(4:2:2)和YCbCr(4:2:2)。  
我们只需通过接口完成对芯片的配置，然后按照接口时序完成对摄像头输出的数据解析。根据我们后面的算法是基于YUV的或者RGB的来控制这款摄像头的视频输出格式，直接存储，方便后面的视频处理。  
关于如何读取这款芯片的输出图像，可以去[阿莫电子论坛](http://www.amobbs.com/forum.php)找下，里面有很多网友共享的实例。不过很多通过MCU来获取的（网站里也有FPGA获取的实例），考虑到MCU处理速度慢的缘故，有些模块还会加个FIFO来做帮助，用FPGA就可以直接获取了，不需要FIFO的帮助。

图像存储
-----
[FPGA存储控制器](http://www.openhw.org/sain_1989/blog/12-07/281798_e0f79.html#articletop)介绍了Xilinx的MCB的使用。因为芯片出来的图像已经变成了顺序的的了，考虑到后面的图像处理算法，我们需要把一帧图像缓存下来，一般情况下我们会缓存好几帧图像。存储器我们可以考虑SRAM,SDRAM，DDR2等存储芯片。综合到图像的大小，存取速率问题，DDR2或者DDR3存储芯片是我们的首选，但是针对不同的存储芯片，在我们设计FIFO村读取的时候，还需要解决一个芯片控制器的问题。一种方法是去[OpenCores](http://opencores.org/)上找开源的IP Core作为我们的存储控制器，另外一种方法就是使用FPGA芯片厂商给我们提供的硬核控制器。  
以Xilinx的Spartan系列FPGA为例，它给用户提供了MCB这个硬核，存储控制模块，我们只需按照这个IP的时序就可以很轻松的完成视频图像的存储与读取。用硬核的一个好处就是不占用FPGA提供给我们的逻辑门，而且在速度上面也有很大的优势，而且很好的支持[AXI](http://baike.baidu.com/view/2375625.htm)总线。

图像处理
-----
看到这儿，一般做视频图像处理的人会觉得这也太麻烦了吧？我openCV里面直接一行代码就可以读取一帧图像了，到你这里怎么麻烦了？
这里以使用USB摄像头为例，说明下图像处理的流程。我们的摄像头处理芯片通过感光元件(感光元件还有CMOS,CCD之分)获取了图像，并且通过USB输送PC机。USB的全称就是Universal Serial Bus，一种计算机总线，而我们的摄像头又有OS下的USB驱动，OS又完成了一次对图像的缓存，接口封装，因此用户软件只需通过OS接口就能直接读取图像了，把前面做的各种繁琐的步骤全部完成了。  

这个时候我们需要注意的是，以我们写的C++图像处理算法为例。计算机是如何处理执行我们的算法的？学过计算机组成的都知道，编译器将我们的代码编译出了汇编语言。计算机读取汇编程序，完成取值，译码，执行操作，一条条执行完我们的代码，又转换回了0101格式。从这个流程大家就知道了CPU做图像处理慢的劣势了，因此PC上又诞生了GPU，而在嵌入式领域有了DSP等专用处理芯片，两者的原理上是差不多的。  
FPGA有一大天生的优势就是能够集成软件的易用与硬件的高效，因此FPGA也被人越来越多的运用到了图像处理领域。  

以Verilog HDL这种用于FPGA的硬件语言为例，hardware language和software language的最大区别在于:前者是一种并行的语言，后者是一种串行的语言。在软件界，并行计算一直是研究的热点，如何让程序并行处理，而硬件语言天生就是并行的。在写Verilog的时候，我们脑子里实际想的就是一个个module，每个module都对应着一个个实际的电路，当我们有些算法需要串行执行的时候，这个时候一般会考虑设计一个FSM来完成。

<img src="{{urls.media}}/FPGA图像处理/RTL.jpg" alt="RTL图" width="600" height="400">  
Verilog对应的RTL图
  
因此设计基于FPGA的图像处理算法的时候，我们就是直接在操作一个个像素点，而且是同时的，而不是像传统的做法那样还要靠CPU的指令翻译顺序执行。
    
软件的好处就是有一个个库可以直接让我们使用，让我们在一个更高的抽象层来设计我们的算法，而且很多现场的库供我们使用，可以大大减少工作量，不用重复造一些没必要的轮子。  

以做一个简单的[sobel边缘检测](http://baike.baidu.com/view/676368.htm)算法为例，我们只需做一个简单的加减运算就可以了，但是在硬件电路里面我们拥有的只是原始的and，or操作，学过计算机组成的知道我们这个时候就需要设计一个加法器，乘法器了。做到这里我们就已经有些崩溃了，连这么基本的轮子我们都得重复造啊？这个时候就可以考虑下面介绍的几个工具。  

*	Simulink HDL Coder
*	Xilinx System Generator
*	Altera DSP Builder
*	Vivado

一些非IT领域的工程师，在做一些控制算法的时候，他们很喜欢使用Simulink来做控制算法，这种基于MBD的开发方法，大大节省了设计，验证的时间。一个个模块的重复使用，很好的仿真测试，这些对于软件工程师来说是最熟悉不过的东西了。
Simulink HDL Coder就是这么一个工具，我们还是可以在simulink里面设计算法，然后进行仿真，然后直接利用HDL Coder生成HDL代码。Xilinx和Altera两个厂家针对自己的FPGA芯片也推出了各自可以在simulink里面使用的MBD设计工具System Generator和DSP Builder。  
我用过Xilinx的System Generator做过一个project。在project里面，有一些数学算法需要处理。

<img src="{{urls.media}}/FPGA图像处理/wave.jpg" alt="" width="600">  
主要是几个简单的三角函数关系

这个时候我就直接使用了System Generator搭建了算法模型，一共有好几个System Generator算法模型，然后把生成的代码与其他Verilog代码整合到一块，有一点很讨厌的就是带上了System Generator生成代码的project，编译，布局布线耗费的时间特别长，让人难以接受，那个project从编译到最后生成bit需要花费4个小时，非常耗时。  

关于System Generator，我非常推荐一本书：[《多媒体处理FPGA实现--System Generator篇》](http://item.jd.com/10067534.html),书里面很详细的介绍了如何使用System Generator来搭建一些基本的图像处理算法，通过System Generator可以很容易搭建一个Sobel边缘检测算法。  

<img src="{{urls.media}}/FPGA图像处理/sobel.jpg" alt="" width="600">  
书中的sobel边缘检测实例

如今Xilinx推出了更加逆天的设计工具Vivado，它里面集成了一个叫做HLS的工具，可以让我们的C++图像处理代码直接变成HDL代码执行。  
在2012的x-fest展会上，有人展示了一个基于xilinx的zynq做的一个愤怒的小鸟的边缘检测算法。  

<embed src="http://player.youku.com/player.php/sid/XNDMxNTI1NTY4/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

从演示视频看到，用FPGA可以很流畅的处理高清视频。

用System Generator或者HLS生成的模块，可以很好的集成到AXI总线上，因此我们可以很方便的设计一个SOPC系统，它就带有了一个专门的硬件加速处理模块。用硬件来做图像处理也是一个趋势，很多芯片厂商直接在芯片里面集成了一些常用的视频处理算法，比如人脸检测算法，这样的SOC就大大提高了视频处理的效率。  

我们可以YY这样一个场景，比如做汽车的道路偏移检测的（高档汽车带有这个功能），里面有一个很重要的事情就是边缘检测然后Hough变换检测直线，这么一个算法对于用C/C++代码来实现的话是个很简单的事情。但是汽车是在高速行进的，因此我们对于算法的处理速度有限制要求，这个时候我们就想到使用FPGA来做，因此想着使用Vivado来实现这个功能，这样我们就可以让普通汽车也带有这样的功能了。  
<img src="{{urls.media}}/FPGA图像处理/hough.jpg" alt="" width="600">  
上面这张图是我开车时候道路图像比较理想情况拍下的，在matlab里面利用hough找到了车道边缘。 
 
P.S现在做汽车周边的很多，有做道路偏移的，有做行人检测的，有做360°停车影像的，他们的商业模式都是想让普通车都拥有高档车才有的功能。有一次我在学校的阿波罗店里打牌，坐在我后面桌子的三个人就在那边讨论做这么一个事情，他们连商业计划什么都做好了，准备流片生产这样的SOC，或者销售licence。