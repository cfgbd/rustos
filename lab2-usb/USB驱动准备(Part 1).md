# 准备写USB驱动 #
---

## <a name="Task1">任务一：</a>了解硬件配置 ##

- 驱动运行的平台：Raspberry PI Model 3B+
- 处理器：Broadcom BCM2837B0, Cortex-A53 64-bit SoC @ 1.4GHz
- 处理器架构：ARM v8
- USB控制芯片：Synopsys DesignWare USB 2.0 Device Controller IP

了解处理器架构

- 树莓派官网/[处理器BCM2837](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2837/README.md)
	- 摘要：BCM2837的底层架构和BCM2836一样，唯一显著的变化是把ARMv7四核处理器替换成了四核ARM Cortex A53(ARMv8)处理器
	- Also see：树莓派官网/[处理器BCM2836](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2836/README.md)
		- 摘要：BCM2836的底层架构和BCM2835一样，唯一显著的变化是把ARM1176JZF-S处理器换成了四核Cortex-A7处理器
		- Also see：树莓派官网/[处理器BCM2835](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2835/README.md)
			- 终于给了参考文档：[外围设备说明书](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2835/BCM2835-ARM-Peripherals.pdf)【[本地链接](Raspberry/BCM2835-ARM-Peripherals.pdf)】
			- 注意：参考文档有大量的错误，这是已知的[勘误表](https://elinux.org/BCM2835_datasheet_errata)
- 附：本组的开发板配置说明书[《Raspberry Pi 3 Model B+ product brief》](Raspberry/Raspberry-Pi-Model-Bplus-Product-Brief.pdf)

了解DMA控制器

- 在外围设备说明书中，第38页【第4节】
	- 这里主要说了DMA控制器的配置，并介绍这块处理器板提供了16个DMA通道
	- 说明书详细说明了DMA寄存器的参数，包含很多控制方法（好多的寄存器），但我没看到怎么利用DMA接收外设发送的数据
	- 我关注到的东西
		- DMA比较复杂，要搞清楚说明书里的DMA控制器，我需要找一个懂DMA的人帮忙解释
		- 16个DMA通道，各有一个中断状态和使能状态[Page 58~60]

了解USB控制芯片

- 在外围设备说明书中，第200页【第15.1节】
	- 使用的USB芯片是Synopsys（新思科技）的产品，了解细节请参考DWC\_otg\_databook.pdf，可在链接页下载
	- [书中的链接【网页】](https://www.synopsys.com/dw/ipdir.php?ds=dwc_usb_2_0_hs_otg)，此页有三个可下载的说明文档，但是文件名都对不上
	- 三个相关文档（官网下载要填申请表）：
		- [DesignWare IP Prototyping Kit for USB 2.0 HS OTG](usb/ip_prototyping_kit_usb2_hs_otg.pdf)
		- [DesignWare IP Prototyping Kits](usb/ip_prototyping_kits_ds.pdf)
		- [DesignWare USB 2.0 Controller IP](usb/dwc_usb2.pdf)
	- 真正的DWC\_otg\_databook.pdf在此处
		1. 点击`Downloads and Documentation`
		2. 在【Databooks】下，有一个文件叫`DesignWare Cores USB 2.0 Hi-Speed On-The-Go (OTG) Databook`，就是他
		3. 点击文件，会提示需要登录帐号
		4. 点击创建帐号，尝试用个人邮箱，会告知创建帐号要Corporation Email（企业邮箱）
		5. 网易163的企业邮箱收费是200一年，但是一次性至少开通5个邮箱，因为是企业邮箱。唉~放弃吧！
		6. 又试了下Tsinghua Email，发现进入了下一步！？
		7. 原来他们只是不知道Tsinghua Email是不是企业邮箱，所以说该site要提交认证码o(X_X)o
		8. 编了个认证码点确定，然后网站说需要2天的人工审核，然后就没有然后了（文件的保密性真够可以的）
- 在外围设备说明书中，第202页【第15.2节】
	- USB block的基地址是`0x7E98_0000`
	- 后文提供了几个USB register的参数
	- 剩下的字节，并不知道能用来干什么
	- 外围设备说明书上说，有关USB block每个字节更详细的内容，需要参考上面说的手册DWC\_otg\_databook.pdf
- 其他思路
	- 在github上尝试寻找树莓派上的USB驱动，发现全是基于Linux USB Framework实现的，并没有任何直接与USB Core交互的代码

（这项工作让我体会到，调研一个新项目需要有足够的耐心`=OvO=`，因为调研的路上踩不到开拓者的足迹，即便找来别人的思路和线索，也不足以成为依据，还需要自己做尝试~）

## <a name="Task2">任务二：</a>从Linux系统代码中找出树莓派3B+的USB Core的交互操作 ##

目标：找到对应Synopsys DesignWare USB 2.0 Device Controller IP这个USB控制器的代码

1. 知识准备
	- USB的传输模式（理解transfer mode的概念）
		- Low   Speed, 慢速 1.5 Mbits/s，在USB 1.0标准中提出，兼容至USB2.0标准
		- Full  Speed, 全速  12 Mbits/s，在USB 1.0标准中提出，兼容至USB2.0标准
		- High  Speed, 高速 480 Mbits/s，在USB 2.0标准中追加提出，由于处理器总线访问的限制，无法充分利用带宽
		- Super Speed, 超速 5.0 Gbits/s，在USB 3.0标准中追加提出，USB 3.0需要设备安装后向兼容插件对旧设备兼容（backward compatible直译为后向兼容、对以往兼容，但意思是向前兼容）
		- Super Speed+, 超速+ 10.0 Gbits/s，在USB 3.1 Gen 2标准中提出，速度是USB 3.0的二倍
		- 总结
			- USB 1.X Low-speed Full-speed
			- USB 2.0 Low-speed Full-speed High-speed
			- USB 3.0 SuperSpeed
			- USB 3.1 SuperSpeed+
	- USB的几个控制器标准（理解Linux的USB host driver文件）
		- OHCI标准：Open Host Controller Interface，OHCI for USB是OHCI标准之一，仅支持USB 1.1
		- UHCI标准：Universal Host Controller Interface，由Intel提出，支持USB 1.X，该标准和OHCI不兼容
		- EHCI标准：Enhanced Host Controller Interface，该标准同时兼容USB 2.0、UHCI和OHCI设备
		- xHCI标准：Extensible Host Controller Interface，该标准支持USB 3.1 SuperSpeed+, USB 3.0 SuperSpeed, USB 2.0 Low-, Full-, and High-speed, USB 1.1 Low- and Full-speed
		- WHCI标准：Wireless Host Controller Interface
2. 开始挖矿
	1. 看芯片的规格文档，知道芯片实际上是EHCI(OTG)控制器，所以应该寻找有关EHCI Host Controller的驱动
	2. 但是Linux系统驱动代码层次太多，还每层都有包装，需要把Linux的整套实现挖出来
		1. 中断处理
			- 结构irq用来处理中断
			- `arch/arm64/kernel/irq.c`: `init_IRQ(void)`
			- `kernel/irq/manage.c`: `request_irq(***)` `free_irq(***)`
			- 这块的代码调用的宏太多了，凭对中断的理解，能猜出代码意图，但不容易找到实现
				- 因为有的宏最后引用了奇怪的东西，只能猜测这东西在汇编里
		2. USB Hub驱动
			- 负责在总线上探测、读写，对1有依赖
			- `drivers/usb/host/ehci.h`: `ehci_readl(***)`
				- 该函数为EHCI芯片的读入操作，在hub驱动中被频繁使用
				- 该函数会调用`arch/arm64/...`中的函数`readl(***)`（我搜索到的其实是个宏）
		3. USB Host Controller驱动
			- 负责控制Host，对2有依赖
		4. USB Device驱动
			- 负责控制设备，对3有依赖
