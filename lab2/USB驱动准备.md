# 准备写USB驱动 #
---

## 任务一：了解硬件配置 ##

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
	- 使用的USB芯片是Synopsys IP的产品，了解细节请参考DWC\_otg\_databook.pdf，可在链接页下载
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
