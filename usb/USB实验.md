# USB驱动的实验 #

实验引用的USB HCD驱动代码：

[github: LdB-ECM/Raspberry-Pi/Arm32\\_64\\_USB](https://github.com/LdB-ECM/Raspberry-Pi/tree/master/Arm32_64_USB)

迄今为止的实验结果：

1. 原项目能正常编译执行
	- 可启动USB电源
	- 可检测、输出USB设备
2. 已将原项目移植进RustOS
	- 路径：kernel/src/arch/aarch64/board/raspi3/usb/
	- 移植了输出模块emb-stdio，以支持sprintf功能
	- 将标准输出用FFI对接到Rust的print!输出，以实现printf功能
3. 在RustOS中添加了对原项目的结构的重定义，以及函数的声明
	- `c_structure.rs, c_structure_usb_2_0.rs, c_structure_usb_1_11.rs`
		- 使用Rust FFI重新封装了rpi-usb.h中所有的常量、枚举、结构
	- `c_api.rs`
		- 声明了rpi-usb.h中所有函数对应的rust函数
		- 定义函数`check_size`，以消除Rust和C间封装实现的不对称
			- 对需要使用的特定结构，计算其大小，计算其中每个域的偏移地址
			- 调用`usb-dependency.c::_RustOS_CheckSize`，进行相同的计算，将结果返回
			- 一一比较每个变量的长度和位置是否一致
			- 可以根据输出的不对称信息，在Rust结构中增加placeholder使结构与C对齐

USB接口函数一览：
<pre>
/*--------------------------------------------------------------------------}
{                        PUBLIC USB DESCRIPTOR ROUTINES                     }
{--------------------------------------------------------------------------*/
    pub fn HCDGetDescriptor (pipe: UsbPipe,                         // Pipe structure to send message thru (really just uint32_t)
                         type0: usb_descriptor_type,                // The type of descriptor
                         index: u8,                                 // The index of the type descriptor
                         langId: u16,                               // The language id
                         buffer: *mut u8,                           // Buffer to recieve descriptor
                         length: u32,                               // Maximumlength of descriptor
                         recipient: u8,                             // Recipient flags
                         bytesTransferred: *mut u32,                // Value at pointer will be updated with bytes transfered to/from buffer (NULL to ignore)
                         runHeaderCheck: bool,                      // Whether to run header check
    ) -> RESULT;
/*--------------------------------------------------------------------------}
{                    PUBLIC GENERIC USB INTERFACE ROUTINES                  }
{--------------------------------------------------------------------------*/
    pub fn UsbInitialise() -> RESULT;
    pub fn IsHub(devNumber: u8) -> bool;
    pub fn IsHid(devNumber: u8) -> bool;
    pub fn IsMassStorage(devNumber: u8) -> bool;
    pub fn IsMouse(devNumber: u8) -> bool;
    pub fn IsKeyboard(devNumber: u8) -> bool;
    pub fn UsbGetRootHub() -> *mut UsbDevice;
    pub fn UsbDeviceAtAddress(devNumber: u8) -> *mut UsbDevice;
/*--------------------------------------------------------------------------}
{                    PUBLIC USB CHANGE CHECKING ROUTINES                    }
{--------------------------------------------------------------------------*/
    pub fn UsbCheckForChange();
/*--------------------------------------------------------------------------}
{                    PUBLIC DISPLAY USB INTERFACE ROUTINES                  }
{--------------------------------------------------------------------------*/
    pub fn UsbGetDescription(device:*mut UsbDevice) -> *const u8;
/*--------------------------------------------------------------------------}
{                        PUBLIC HID INTERFACE ROUTINES                      }
{--------------------------------------------------------------------------*/
    pub fn HIDReadDescriptor(devNumber: u8,                         // Device number (address) of the device to read 
                              hidIndex: u8,                         // Which hid configuration information is requested from
                              Buffer: *mut u8,                      // Pointer to a buffer to receive the descriptor
                              Length: u16,                          // Maxium length of the buffer
    ) -> RESULT;
    pub fn HIDReadReport(devNumber: u8,                             // Device number (address) of the device to read
                          hidIndex: u8,                             // Which hid configuration information is requested from
                          reportValue: u16,                         // Hi byte = enum HidReportType  Lo Byte = Report Index (0 = default)  
                          Buffer: *mut u8,                          // Pointer to a buffer to recieve the report
                          Length: u16,                              // Length of the report
    ) -> RESULT;
    pub fn HIDWriteReport(devNumber: u8,                            // Device number (address) of the device to write report to
                           hidIndex: u8,                            // Which hid configuration information is writing to
                           reportValue: u16,                        // Hi byte = enum HidReportType  Lo Byte = Report Index (0 = default) 
                           Buffer: *mut u8,                         // Pointer to a buffer containing the report
                           Length: u16,                             // Length of the report
    ) -> RESULT;
    pub fn HIDSetProtocol(devNumber: u8,                            // Device number (address) of the device
                           interface: u8,                           // Interface number to change protocol on
                           protocol: u16,                           // The protocol number request
    ) -> RESULT;
</pre>

现在的实验效果：

- 因为缓存的原因，HCD总线的部分操作未能顺利执行，需要在必要的位置做清理缓存的操作