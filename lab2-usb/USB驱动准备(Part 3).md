# 准备写USB驱动 #
---
[Part-1](USB驱动准备\(Part%201\).md)
[Part-2](USB驱动准备\(Part%202\).md)
[Part-3](USB驱动准备\(Part%203\).md)
<!--这里空一行，不要留标题-->
---

## <a name="Task3">任务三：</a>解读EHCI手册 ##

前言
--

&ensp;
尽管任务二并未完成，但是一个新的发现，驱使我直接启动了任务三。
这个发现就是：我终于找到了[EHCI手册](usb/ehci-specification-for-usb.pdf)。
发现的过程嘛……
就是我在百度、Github、Wikipedia上死活找不到关于EHCI的知识了。
就终于翻墙试了试Google，搜索下“EHCI protocol”，结果这篇文档直接排到了Rank 1。
哇塞！总算找到你了！Google就是好！

解读
--

USB通信中，设备分为Host和Client，如图为USB Host的架构：

![说明图片：illustration/1.png](illustration/1.png)

看右侧路线，从上层到底层依次为：系统软件（客户端驱动-USB总线驱动-EHCI驱动）-硬件（EHCI控制器-USB设备）

而我们要实现的部分，就是自底向上的完成系统软件部分，即：先写EHCI驱动，（然后USB总线驱动就算了，）和USB键盘的客户端驱动

EHCI寄存器
--

EHCI的控制寄存器有PCI配置寄存器和内存映射USB主机控制器寄存器

PCI，Peripheral Component Interconnect(外设部件互连标准)，可不考虑相关内容

- 需要交代的一点是，PCI寄存器中有个寄存器的作用是配置内存映射寄存器的base地址，但对于不提供PCI接口的EHCI控制器（比如树莓派上的），不适用这种规则

内存映射寄存器，共两部分：

- Host Controller Capability Registers
	- 只读，描述EHCI控制器的限制、约束和能力
	- CAPLENGTH: Offset 00h, Size 1byte=8bits
		- 描述CAP段的长度，单位字节
		- REG_BASE+CAP_LENGTH就是Operational Register Space的起始地址
- Operational Register Space
	- 读写，