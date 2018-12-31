# USB代码分析 #

增加的usb的代码全部位于：项目目录/kernel/src/arch/aarch64/board/raspi3/usb/下

使用了github上开源的USB HCD驱动：[LdB-ECM/Raspberry-Pi/Arm32\\_64\\_USB](https://github.com/LdB-ECM/Raspberry-Pi/tree/master/Arm32_64_USB)

## 项目代码包括11个文件

- .rs 文件 5 个
	- `mod.rs`：module的主文件
		- 定义了usb module下的其他module
		- 定义了一些对外开放的结构和函数
			- 目前尚未完全开放，仅开放了结构UsbDevice
			- 开放了函数UsbShowTree和get_root_hub
		- 定义了对C提供的ffi函数
			- 输出函数：`rustos_print`
	- `c_struture.rs`, `c_struture_usb_1_11.rs`, `c_struture_usb_2_0.rs`：定义结构体的文件
		- 重新定义了驱动程序里的所有常量、枚举、和结构
		- 因为驱动的剩余部分希望使用rust实现，将会涉及对结构体的访问，所以需要在rust内重新定义
		- 遇到的问题：
			- C中的enum类型的实现
				- 在rust中对应的enum类型外使用注解#[repr(i32)]或#[repr(u32)]
			- C中使用`__attribute__((__packed__, aligned(4)))`的struct类型的实现
				- 在rust中对应的struct类型外使用注解#[repr(C, packed)]
			- C的struct里，对单独的field使用了`__attribute__((aligned(4)))`
				- 在网上查到可使用注解#[repr(align(4))]，但是并没有效果
				- 只能在rust中向结构体的对应位置插入placeholder，如`_placeholder0: [u8; 3],`
	- `c_api.rs`：定义函数接口的文件
		- 将C语言的usb driver的所有接口声明为了rust的函数
		- 对UsbShowTree函数使用rust进行了重新实现
		- 定义了check_size函数，可以检查rust中的结构体和C的结构体的对齐情况
			- 检查对齐部分使用宏`Align_Offset!`和宏`Align_Sizeof!`对要检查的内容打表
			- 调用C的函数`_RustOS_CheckSize`，在C中用相同操作对检查内容打表
			- 对两个表的内容进行比对
- .h 文件 2 个，.c 文件 3 个，.S 文件 1 个
	- `rpi-usb.c`, `rpi-usb.h`：usb驱动的核心代码
	- `emb-stdio.c`：嵌入式stdio
		- 只有一个emb-sprintf函数
		- 输出操作需要调用RustOS提供的输出函数：`rustos_print`
	- `usb-dependency.c`, `usb-dependency.h`, `usb-dependency64.S`：usb驱动的依赖代码
		- 文件里是从源项目中抽取出的代码
		- 主要用途是支持usb驱动的获取时间、访问mailbox、地址转换和探测处理器的Register地址
			- 关于时间操作的核心函数是`usb-dependency.c`里的`timer_getTickCount64`
			- 访问mailbox的操作也在`usb-dependency.c`中，但已放弃调用，转而调用了RustOS内单独开放的函数（原因是处理器的cache使原代码在和mailbox通过内存交互时发生错误）
			- 地址转换的操作是`usb-dependency64.S`的`ARMaddrToGPUaddr`和`GPUaddrToARMaddr`，目前是恒等函数（转换是因为ARM地址是虚地址，GPU地址是物理地址）
			- 探测Register地址由`usb-dependency64.S`的函数`UsbDependencyInit`完成，结果存放`usb-dependency.h`的extern变量里

## 代码运行结果分析 ##

- HCD程序能通过mailbox成功给usb设备加电
- HCD程序在使用DMA和控制器进行信息交互时会出现问题
	- 出错可能1：由于cache原因，导致程序未读到实时数据
		- 经过关mmu测试，已基本排除该可能
	- 出错可能2：由于代码优化，导致内存访问不符合预期
		- 比如：指令对内存乱序访问，此时需要增加memory barrier，即内存屏障
			- 方法1：增加这样的代码段__asm__ __volatile__("" ::: "memory")
			- 方法2：给需要重新读写的变量增加volatile关键字
		- 由于时间原因，尚未对该可能完全测试
