MIPS32处理器需求文档
=====

##概述

###项目背景

本项目来源于清华大学计算机科学与技术系计算机组成原理课程、软件工程课程、操作系统课程、编译原理课程联合实验。其目标是设计一颗部分兼容于MIPS32体系结构的CPU，基于FPGA硬件平台实现，并能够运行ucore操作系统。本项目可使学生深入理解计算机系统原理，并在计算机系统级底层开发方面得到训练。

###需求方

- 计算机组成原理课程：刘卫东老师
- 软件工程课程：白晓颖老师
- 操作系统课程：向勇老师
- 编译原理课程：王生原老师

###术语定义

本文档中出现的术语缩写定义如下

术语          | 定义
------------ | -------------
MIPS | 无内部互锁流水线微处理器
CPU  | 中央处理器
ALU  | 算数逻辑单元
MMU  | 内存管理单元
TLB  | 翻译后备缓冲区
RAM  | 随机访问存储器
ROM  | 只读存储器
BIOS  | 基本输入输出系统
Flash | 快闪存储器
CP  | 协处理器

##需求描述

###CPU

#####基本功能

CPU需在系统时钟的驱动下，在一个至多个周期内获取并执行一条指令，而后继续获取执行下一条指令，如此往复。支持的指令集为MIPS32指令集的子集，该指令集包含但不限于如下指令:

- 加载、存储指令：LB、LH、LW、SB、SH、SW
- 简单算数运算指令：ADDI、SUB
- 逻辑运算类指令：ANDI、ORI、SLTI、XOR、CLO、SLL、SRA
- 乘除法相关指令：MUL、DIV、MFHI、MTLO
- 分支与跳转指令：J、JAL、JR、BEQ、BGEZ
- 条件移动指令：MOVZ、MOVN
- 异常相关指令：SYSCALL、ERET
- CP0相关指令：MFC0、MTC0
- TLB相关指令：TLBWI

完整的指令集在附录中列出。

为提高运行效率，CPU采用经典5级流水方式，5个阶段分别是取指、译码、执行、访存、回写。取指阶段CPU从指令总线获取一条指令，每次取指在单个时钟周期完成。译码阶段对指令编码进行解释，并获取通用寄存器的值。执行阶段按照指令执行实际运算操作。访存阶段通过数据总线读取或写入内存单元，在个操作在单周期内完成。会写阶段将结果写入通用寄存器。

大部分运算指令在ALU中执行，只消耗一个时钟周期。但乘除法运算过程较复杂，不能在单周期内完成，使用专用的乘除法单元完成运算，多周期运算时暂停流水线。

#####异常处理

由于操作系统的需求，本处理器需要支持必要的异常和中断处理。

处理器需要支持的异常如下：

- Reset：硬件复位
- Interrupt：外部触发中断
- TLBL：无效的加载地址引发异常
- TLBS：无效的存储地址引发异常
- Sys：系统调用指令触发
- RI：无效指令
- Ov：算数运算溢出异常
- AdEL：未对齐的加载地址引发异常
- AdES：未对齐的存储地址引发异常
- TLB Mod：对TLB违规写操作

处理器还需要支持多个中断信号，包括：

- 系统定时器中断：用于操作系统调度
- 串口中断：表示串口收到数据
- 键盘中断：表示键盘收到按键

在流水线设计中，要求支持精确异常处理。即处理器会准确记录发生异常的指令位置（包括位于延迟槽中的指令），并确保异常发生之前的指令均完整执行，之后的指令取消。

#####CP0

#####TLB

###总线

###串口控制器

###RAM控制器

###Flash控制器

###键盘控制器


##软件环境

本系统设计的目标软件是ucore操作系统。启动操作系统之前，需要首先运行Bootloader，准备必要的系统启动条件。Bootloader固化在FPGA内部ROM中，负责将操作系统代码从Flash拷贝到RAM中，之后跳转到RAM中操作系统入口所在地址，将执行权交给操作系统。Flash控制器在本阶段被使用。

操作系统初始化过程中，与CPU相关的主要步骤依次为CP0配置、TLB初始化、中断控制器初始化、串口初始化、定时器中断初始化。CPU需要正确地支持这些初始化操作。

之后，在操作系统运行过程中，时钟中断、外设中断和TLB等异常会时常发生，异常处理程序入口有预先放置的代码用于处理异常。

##硬件平台

本系统将在真实硬件平台上运行验证，该平台由计算机原理课程实验室提供，技术参数如下：

组件     | 数量   | 型号/参数
--------|-------|----------
FPGA     | 1     | Xilinx<sup>&reg;</sup> Spartan<sup>&reg;</sup>-6 XC6SLX100
SRAM     | 4     | 总共 2M &times; 32bits
Flash    | 1     | 4M &times; 16bits
CPLD     | 1     | Xilinx<sup>&reg;</sup> XC95144XL
串口      | 3     | 
数码管    | 2     |
LED      | 16     |
PS/2接口  | 1     |
拨码开关   | 32    |
按钮开关   | 4     |
以太网控制器| 1     | DM9000A Fast Ethernet Controller
VGA接口    | 1     | 3bits DAC / Channel
USB-OTG控制器 |1    |ISP1362

##附录 

###指令集

###CP0寄存器

##参考文献

1. MIPS32<sup>TM</sup> Architecture For Programmers Volume I: Introduction to the MIPS32<sup>TM</sup> Architecture
2. MIPS32<sup>TM</sup> Architecture For Programmers Volume II: The MIPS32<sup>TM</sup> Instruction Set
3. MIPS32<sup>TM</sup> Architecture For Programmers Volume III: The MIPS32<sup>TM</sup> Privileged Resource Architecture