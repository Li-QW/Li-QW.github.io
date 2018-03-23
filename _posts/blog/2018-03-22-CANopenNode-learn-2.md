---
layout: post
title: CANopenNode学习（2/2）
date: 2018-3-22
categories: blog
tags: [CANopen,CANopenNode,学习,代码]
description: 学习使用CANopenNode开源库开发CANopen网络。
---
> 学习CANopenNode的最终目的是：结合其API在PIC18F单片机上开发CANopen通信节点，并采集、传递和处理信号。  
> 英语及专业水平有限，如有纰漏和表达不周，请及时与我联系，谢谢。

文中使用的CANopenNode V1.1源码和文件可以在[这里下载](https://sourceforge.net/projects/canopennode/files/canopennode/CANopenNode-1.10/)   
新的工程已经移步到[GitHub](https://github.com/canopennode)可以支持PIC32等更多的MCU。

由于v1.1版本涵盖了PIC18F的实例和工程文件，并有详细的手册和使用指南，所以选择v1.1版本作为入门学习。下面的章节主要依据 [教程][R1]。

CANopenNode V1.1采用许可`GNU Free Documentaion License`.

接上文：[CANopenNode学习（1/2）][L0]

### 3.4 动力装置的编程

动力装置基于_blank_project。 所有的差异都会标识出来。它将是相当先进的智能控制设备，可根据网络状态，运行状态，其他节点的状态，远程传感器的温度以及温度上限和下限的内部保持变量来控制加热器和冷却器。

打开工程*Tutorial_Power*

#### 3.4.1 向对象字典添加变量 – CO_OD.c文件

添加下面表中的变量：

索引与子索引|变量名 |类型 |存储类型 |访问类型  
---------- |-----|-----|----------|-----------  
0x3000, 0x00 |`TempLo` |UNSIGNED 16 |Flash |Read/Write  
0x3001, 0x00 |`TempHi` |UNSIGNED 16 |Flash |Read/Write  
0x3100, 0x00 |`Status` |UNSIGNED 8 |RAM |Read only  
0x3200, 0x00 |`RemoteTemperature` |UNSIGNED 16 |RAM |  Read/Write  

`TempLo`单位是摄氏度，若温度低于它，则加热器打开。同样`TempHi`单位也是摄氏度，温度比它高时，冷却器打开。`TempLo`和`TempHi`都是保持性的，并且与PDO没有关系。它们可以从网络端（通过SDO客户端）写入，但不能通过程序直接更改。 在写入期间，微控制器被冻结约20ms。 作为保持性变量的替代方法，可以将EEPROM与RAM结合使用。例如变量`PowerOnCounter`。

`Status`和`RemoteTemperature`是两个RAM变量，可以映射到PDO。这里讨论了PDO和变量之间连接的不同方法。

`Status`包含有关加热器和冷却器的状态信息。它有自己的变量。稍后会显示代码，它将确定状态的变化，然后将变量的内容复制到TPDO并触发PDO。

`RemoteTemperature`是从远程传感器接收的以摄氏度为单位的温度。它使用来自RPDO 0的前两个字节的内存。所以当RPDO到达时，可以立即读取`RemoteTemperature`而无需另外复制。

`Status`使用自己的变量并不是规定的。它也可以使用TPDO中的内存，与`RemoteTemperature`相似。它可以由事件定时器触发。但是不可能确定状态的改变。

有时需要对RPDO使用单独的变量。如果程序正在读取RPDO数据，而RPDO到达并以高优先级中断进行处理，然后程序读取后半部分数据，则可能会有危险。程序有一半来自旧RPDO数据，一半来自新RPDO！因此最好在读取期间禁用中断。在我们的案例中，这不是必需的，因为我们只使用一个字节。

从低优先级定时器中断函数安全读取RPDO，示例如下：

```C
if(CO_RPDO_New(0)){
    CO_DISABLE_CANRX_TMR();
    ODE_Write_Digital_Output.DWORD[0] = CO_RPDO(0).DWORD[0];
    CO_RPDO_New(0) = 0;
    CO_ENABLE_CANRX_TMR();
}
```

变量定义并赋初值。在*Manufacturer specific variables*部分添加代码：

```C
/*0x3000*/  ROM UNSIGNED16 TempLo = 18;
            //低于此值加热器打开
/*0x3001*/  ROM UNSIGNED16 TempHi = 25;
            //高于此值冷却器开启
/*0x3100*/  tData1byte Status = 0;
/*0x3200*/
#define RemoteTemperature   CO_RPDO(0).WORD[0]
```

确认写入对象字典的值，在`CO_OD_VerifyWrite`函数中添加代码：

```C
case 0x3000: //TempLo
    if((*((unsigned int*)data) > 35) ||
       (*((unsigned int*)data) >= TempHi))
            return 0x06090031L;     //Value of param. written too high
    break;
case 0x3001: //TempHi
    if((*((unsigned int*)data) < 5) ||
        (*((unsigned int*)data) <= TempLo))
            return 0x06090032L;     //Value of param. written too low
    break;
```

将变量添加到对象字典。此时要留心，索引和子索引必须在整个CO_OD数组中排序。在*Manufacturer specific*部分添加代码：

```C
OD_ENTRY(0x3000, 0x00, ATTR_RW|ATTR_ROM, TempLo),
OD_ENTRY(0x3001, 0x00, ATTR_RW|ATTR_ROM, TempHi),
OD_ENTRY(0x3100, 0x00, ATTR_RO, Status),
OD_ENTRY(0x3200, 0x00, ATTR_RWW, RemoteTemperature),
```

关于*CO_OD.c*文件的部分在这里就结束了。在*CO_OD.h*中将声明上面的变量和PDO映射。在*User.c*中将使用变量，并触发TPDO。

#### 3.4.2 配置工程 - CO_OD.h文件

代码以等宽字符显示，与_blank_project不同的地方已做标识。

*Setup CANopen*部分代码：

```C
#define CO_NO_SYNC              0   //<<
#define CO_NO_EMERGENCY         1   
#define CO_NO_RPDO              1   //<<
#define CO_NO_TPDO              1   //<<
#define CO_NO_SDO_SERVER        1
#define CO_NO_SDO_CLIENT        0
#define CO_NO_CONS_HEARTBEAT    2   //<<
#define CO_NO_USR_CAN_RX        0
#define CO_NO_USR_CAN_TX        0
#define CO_MAX_OD_ENTRY_SIZE    20
#define CO_SDO_TIMEOUT_TIME     10
#define CO_NO_ERROR_FIELD       8
#define CO_PDO_PARAM_IN_OD
#define CO_PDO_MAPPING_IN_OD
#define CO_TPDO_INH_EV_TIMER
#define CO_VERIFY_OD_WRITE
#define CO_OD_IS_ORDERED
#define CO_SAVE_EEPROM
#define CO_SAVE_ROM
```

在这里未使用SYNC对象，从采集单元接收RPDO0（两字节），用TPDO0（一字节）来报告状态，并监控另外两个节点。

*Default values for object dictionary*部分代码保持默认状态。

*Heartbeat consumer*部分代码如下：

```C
#define ODD_CONS_HEARTBEAT_0    0x000605DCL //<<
#define ODD_CONS_HEARTBEAT_1    0x000805DCL //<<
```

这里设置监控ID为6和8的节点，若Heartbeat信号丢失超过1500ms，则动力装置切换到`Pre-Operational`状态。

*0x1400 Receive PDO parameters* 部分代码：

```C
#define ODD_RPDO_PAR_COB_ID_0   0x40000286L //<<
#define ODD_RPDO_PAR_T_TYPE_0   255
```

节点会直接从采集单元（COB-ID=0x286）接收PDO。

*0x1600 Receive PDO mapping* 部分代码：
```C
#define ODD_RPDO_MAP_0_1 0x32000010L    //<<
#define ODD_RPDO_MAP_0_2 0x00000000L
#define ODD_RPDO_MAP_0_3 0x00000000L
#define ODD_RPDO_MAP_0_4 0x00000000L
#define ODD_RPDO_MAP_0_5 0x00000000L
#define ODD_RPDO_MAP_0_6 0x00000000L
#define ODD_RPDO_MAP_0_7 0x00000000L
#define ODD_RPDO_MAP_0_8 0x00000000L
```

RPDO0的长度是两字节。接收的PDO必须具有同样的长度，就像在映射中定义的一样，否则将会产生错误。RPDO数据也可以通过对象字典索引0x3200,子索引0（“变量”`RemoteTemperature`位置）访问。

*0x1800 Transmit PDO parameters* 部分代码：

```C
#define ODD_TPDO_PAR_COB_ID_0   0
#define ODD_TPDO_PAR_T_TYPE_0   254
#define ODD_TPDO_PAR_I_TIME_0   100     //<<
#define ODD_TPDO_PAR_E_TIME_0   1000    //<<
```

PDO COB-ID(11位CAN标识符)将保持默认——0x180+NodeID。PDO 0的抑制时间（Inhibit time）为100 *100μs，因此PDO更新将不会比间隔10ms更快。PDO 0将在状态改变时与每秒钟定时（1000ms）发送。传输类型是254-制造商特定的。

*0x1A00 Transmit PDO mapping* 部分代码：

```C
#define ODD_TPDO_MAP_0_1 0x31000008L    //<<
#define ODD_TPDO_MAP_0_2 0x00000000L    //<<
#define ODD_TPDO_MAP_0_3 0x00000000L
#define ODD_TPDO_MAP_0_4 0x00000000L
#define ODD_TPDO_MAP_0_5 0x00000000L
#define ODD_TPDO_MAP_0_6 0x00000000L
#define ODD_TPDO_MAP_0_7 0x00000000L
#define ODD_TPDO_MAP_0_8 0x00000000L
```

TPDO0的长度是1字节（8位）。TPDO数据也可以通过对象字典索引0x3100,子索引0（“变量”`Status`位置）访问。

*Default values for user Object Dictionary Entries* 部分代码：

```C
#define ODD_CANnodeID   0x07    //<<
#define ODD_CANbitRate  3
```

动力装置的NodeID为7，CAN比特率为125kbps。

#### 3.4.3 应用程序接口 - User.c文件

代码以等宽字体显示。与 ~~Example_generic_IO~~ _blank_project不同的地方已标出。

头文件中的部分定义：

```C
#include "CANopen.h"
/*0x3000*/ extern ROM UNSIGNED16    TempLo;
/*0x3001*/ extern ROM UNSIGNED16    TempHi;
/*0x3100*/ extern tData1byte        Status;
/*0x3200*/
        #define RemoteTemperature CO_RPDO(0).WORD[0]
#define STATUS_BYTE     Status.BYTE[0]
#define STATUS_COOLER   Status.BYTEbits[0].bit0
#define STATUS_HEATER   Status.BYTEbits[0].bit1
#define DigOut_COOLER   PORTCbits.RC4
#define DigOut_HEATER   PORTCbits.RC5
#define ERROR_TEMP_LOW  ERROR_USER_4
```

更改`User_Init`函数：

```C
void User_Init(void){
    ODE_EEPROM.PowerOnCounter++;

    TRISCbits.TRISC4 = 0; PORTCbits.RC4 = 0;    //<<
    TRISCbits.TRISC5 = 0; PORTCbits.RC5 = 0;    //<<
}
```

更改`User_Process1msIsr`函数：

```C
void User_Process1msIsr(void){
    // 对于状态变化的传输Inhibit timer是必要的
    extern volatile unsigned int CO_TPDO_InhibitTimer[];
    
    if(CO_NMToperatingState == NMT_OPERATIONAL &&
       CO_HBcons_AllMonitoredOperational == NMT_OPERATIONAL){
        
        // 控制加热器和冷却器
        if(RemoteTemperature < TempLo){
            DigOut_COOLER = 0;
            DigOut_HEATER = 1;
        }
        else if(RemoteTemperature > TempHi){
            DigOut_COOLER = 1;
            DigOut_HEATER = 0;
        }
        else{
            DigOut_COOLER = 0;
            DigOut_HEATER = 0;
        }
        
        // 温度过低发送急停消息
        if(RemoteTemperature < 5)
            ErrorReport(ERROR_TEMP_LOW, RemoteTemperature);
        else if(ERROR_BIT_READ(ERROR_TEMP_LOW))
            ErrorReset(ERROR_TEMP_LOW, RemoteTemperature);
        
        // PDO状态变化
        if(CO_TPDO_InhibitTimer[0] == 0 && (
           CO_TPDO(0).BYTE[0] != STATUS_BYTE)){

            CO_TPDO(0).BYTE[0] = STATUS_BYTE;
            if(ODE_TPDO_Parameter[0].Transmission_type >= 254)
                CO_TPDOsend(0);
        }
    }

    else{
        // 关掉一切
        DigOut_COOLER = 0;
        DigOut_HEATER = 0;
    }

    // 写状态
    STATUS_COOLER = DigOut_COOLER;
    STATUS_HEATER = DigOut_HEATER;
}
```

函数每ms执行一次。也可以使用*User_ProcessMain*函数代替。<u>无论如何，所有的代码都必须解锁。</u>（XW_180322： 这里不是很明白，原文`Anyway all code must be unblocking.`）

#### 3.4.4 电子数据表EDS - Tutorial_Power.eds文件

Eds文件可以与CANopen监视器一起使用，以便在对象字典中轻松而清晰地设置参数。它可以用商业程序进行编辑。因为它是一个文本文件，所以也可以手工编辑，如下所示。 您可以将 *Tutorial\_Power.eds* 与 *\_blank\_project.eds* 进行比较以查看差异。

待续……


    2018/3/22更新，很快就要完成了。


 
[L0]:https://li-qw.github.io/blog/2018/03/21/CANopenNode-learn-1/ "上一部分" 
[R1]:https://sourceforge.net/projects/canopennode/files/canopennode/CANopenNode-1.10/ "《CANopenNode Turorial》  V1.10"  
[R2]:https://sourceforge.net/projects/canopennode/files/canopennode/CANopenNode-1.10/ "《CANopenNode Manual》 V1.10"  
[R3]:mailto:janez.paternoster@siol.net "作者 Janez Paternoster"
[R4]:http://www.winmerge.org/ "文件比较工具"
[P1-1]:https://us1.myximage.com/2018/03/22/1184ad33502524a63969c282b9040d7f.png "简单的CANopen网络"
[P2-1]:https://us1.myximage.com/2018/03/22/505b807b391ee5b3b38ae4a5c772b07a.png "CAN收发器与PIC MCU的连接"
[P3-1]:https://us1.myximage.com/2018/03/22/b34e867948786dfb5afd0e0824a6152c.png "路径设置"