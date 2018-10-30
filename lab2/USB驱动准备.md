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

了解USB控制芯片

- 在【处理器架构/外围设备说明书】中，第200页
	- 使用的USB芯片是Synopsys IP的产品，了解细节请参考DWC\_otg\_databook.pdf，可在链接页下载
	- [书中的链接【网页】](https://www.synopsys.com/dw/ipdir.php?ds=dwc_usb_2_0_hs_otg)，此页有三个可下载的说明文档，但是文件名都对不上
	- 三个相关文档（官网下载要填申请表）：
		- [DesignWare IP Prototyping Kit for USB 2.0 HS OTG](usb/ip_prototyping_kit_usb2_hs_otg.pdf)
		- [DesignWare IP Prototyping Kits](usb/ip_prototyping_kits_ds.pdf)
		- [DesignWare USB 2.0 Controller IP](usb/dwc_usb2.pdf)

（至此，官方文档及勘误表整理完毕。这项工作让我体会到，做调研需要有足够的耐心`=OvO=`）
