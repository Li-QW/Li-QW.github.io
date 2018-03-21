---
layout: post
title: 180321 CANopenNode学习
date: 2018-3-21
categories: blog
tags: [CANopen,学习,代码]
description: 你以为这里会有什么，其实什么也没有。
---
> 学习CANopenNode的最终目的是：结合其API在PIC18F单片机上开发CANopen通信节点，并采集、传递和处理信号。  

CANopenNode v1.10源码和文件可以在[这里下载](http://sourceforge.net/projects/canopennode/) http://sourceforge.net/projects/canopennode/   
新的工程已经移步到[GitHub](https://github.com/canopennode)https://github.com/canopennode  可以支持更多32位的MCU。  

CANopenNode V1.1采用`GNU Free Documentaion License`.

## 1. 介绍  
通过[教程][R1]，学习使用CANopenNode开源库开发CANopen网络。 它将显示如何：制作通用输入/输出设备，使用过程数据对象（传输和映射），在对象字典中使用保持性变量，为智能设备制作自己的程序，为NMT主设备使用自定义CAN消息，使用紧急消息 针对自定义错误等。

在后面的章节中将介绍如何使用 `panel_with_PIC+LCD+keypad`或`Web_interface_with_SC1x`示例（都包含在CANopenNode中）或使用标准配置工具配置网络。

[教程][1]介绍了CANopenNode的大部分功能。 这是相当多的，教程中有很多信息。 源代码已经过测试，所以用户不应该遇到问题。 如果有问题或疑问，请联系[作者][3]。

涉及到的名词：  
- `Heartbeat`     心跳
- `NMT`   网络管理

### 1.1 准备
1. [CANopenNode 源码 v1.10](http://sourceforge.net/projects/canopennode/);
2. [MPLAB IDE](http://www.microchip.com);
3. [MPLAB C18 V3.00 +](http://www.microchip.com);
4. 三块带有微控制器PIC18F458（或其他具有CAN的PIC18F）和CAN收发器的电路板;
5. PIC编程器（使用[MPLAB ICD2](http://www.microchip.com)工作正常）。
6. 熟悉使用MPLAB IDE和MPLAB C18。
7. 有关CANopen协议的知识（在线培训[can-cia](http：//www.can-cia.org/canopen/)或[esacademy](http://www.esacademy.com/myacademy/)，[书籍](www.canopenbook.com/))。
8. 建议：CANopen配置工具，如`panel_with_PIC+LCD+keypad`（CANopenNode的一部分）或
   `Web_interface_with_SC1x`（也是CANopenNode的一部分）或任何其他商用CANopen配置工具。

### 1.2 说明
网络例程是带有远程传感器和命令接口的空调机组。它由三个CANopen设备组成（图1.1）：  
1. 传感器——独立温度传感器：  
    - 基于通用输入/输出设备配置文件（CiADS401）。
    - 在每次状态改变时传输感测温度，并定期使用事件定时器（TPDO 1）。
    - 每秒产生一次心跳。
2. 功率单元——由加热器和冷却器制成的空调单元：
    - 从传感器接收温度（RPDO 0）。
    - TempLo和TempHi变量位于“对象字典”中。 可以通过SDO通信对象使用CANopen配置工具访问和更改它们。变量具有保持性（关机后不丢失）。 单位是[℃ 摄氏度]。
    - 打开或关闭冷却器或加热器。决定基于来自传感器，TempLo和TempHi的温度。 如果设备未处于运行状态，则冷却器和加热器将关闭。
    - 在每次状态改变时传送冷却器和加热器的状态，并周期性地使用事件定时器（TPDO 0）。
    - 每秒产生一次心跳。
    - 监测来自传感器和命令接口的心跳。
3. 命令接口——一个按钮和两个LED二极管，可选LCD显示器：
    - 接收来自传感器（RPDO 0）的温度并将其显示在LCD上（可选）。
    - 从电源单元（RPDO 1）接收冷却器和加热器的状态，并将其显示在LED二极管上。
    - 与按钮连接的NMT主设备。如果按下按钮，则所有网络将被设置为预操作`pre-operational`状态，并且如果再次按下按钮，则所有网络将被设置为可操作的NMT状态。如果更长时间（5秒）按下按钮，则网络上的所有节点都将重置。
    - 每秒产生一次心跳。

除上述通信对象外，每个节点还使用SDO，紧急和NMT（从）通信对象。

> 待续...

# 参考资料
 该文主要作为CANopenNode的学习记录，参考资料：  
[R1]: http://sourceforge.net/projects/canopennode "《CANopenNode Turorial》  V1.10"  
[R2]: http://sourceforge.net/projects/canopennode "《CANopenNode Manual》 V1.10"  
[R3]: janez.paternoster@siol.net "作者 Janez Paternoster"
