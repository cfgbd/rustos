# 准备写USB驱动 #
---
[Part-1](USB驱动准备\(Part%201\).md)
[Part-2](USB驱动准备\(Part%202\).md)
[Part-3](USB驱动准备\(Part%203\).md)
<!--这里空一行，不要留标题-->
---

## <a name="task2">任务二：</a>从Linux系统代码中找出树莓派3B+的USB Core的交互操作 ##

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
