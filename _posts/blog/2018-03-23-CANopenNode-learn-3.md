---
layout: post
title: CANopenNode学习（3）
date: 2018-3-22
categories: blog
tags: [CANopen，CANopenNode，学习，代码]
description: 学习使用CANopenNode开源库开发CANopen网络。
---
> 学习 CANopenNode 的最终目的是：结合其 API 在 PIC18F 单片机上开发 CANopen 通信节点，并采集、传递和处理信号。  
> 英语及专业水平有限，如有纰漏和表达不周，请及时与我联系，谢谢。

文中使用的 CANopenNode V1.1 源码和文件可以在 [这里下载](https://sourceforge.net/projects/canopennode/files/canopennode/CANopenNode-1.10/)   
新的工程已经转移到 [GitHub](https://github.com/canopennode) 可以支持 PIC32 等更多的 MCU。

由于 v1.1 版本涵盖了 PIC18F 的实例和工程文件，并有详细的手册和使用指南，所以选择 v1.1 版本作为入门学习。下面的章节主要依据 [教程][R1]。

CANopenNode V1.1 采用许可 `GNU Free Documentaion License`.

接上文：   
- [CANopenNode学习（1）][L1]  
- [CANopenNode学习（2）][L2]

前面两部分主要介绍了 CANopenNode 的基本配置方法，并选择了具有三个节点的典型案例进行说明。并对采集单元和动力装置部分的程序进行了解析，该页面主要进行剩下的工作，即第三个节点——操作单元的编程。

### 3.5 操作单元的编程

操作单元基于空白工程 *\_blank\_project*。所有的差异已经标出。它的作用是显示加热器和冷却器的状态及当前温度值。它也是 **NMT Master**，可以改变其它节点的运行状态。

打开工程文件 *Tutorial\_Command*

#### 3.5.1 配置工程 - CO_OD.h 文件

代码以等宽字符显示，与 *\_blank\_project* 不同的地方已标出。我们将尽可能地简化这个装置，所占存储空间也会减小。

*Setup CANopen* 部分代码：

```c
#define CO_NO_SYNC              0   //<<
#define CO_NO_EMERGENCY         1
#define CO_NO_RPDO              2   //<<
#define CO_NO_TPDO              0   //<<
#define CO_NO_SDO_SERVER        1
#define CO_NO_SDO_CLIENT        0
#define CO_NO_CONS_HEARTBEAT    0   //<<
#define CO_NO_USR_CAN_RX        0
#define CO_NO_USR_CAN_TX        1   //<<
#define CO_MAX_OD_ENTRY_SIZE    4   //<<
#define CO_SDO_TIMEOUT_TIME     10
#define CO_NO_ERROR_FIELD       8
//#define CO_PDO_PARAM_IN_OD         //<<
//#define CO_PDO_MAPPING_IN_OD       //<<
//#define CO_TPDO_INH_EV_TIMER       //<< 
//#define CO_VERIFY_OD_WRITE         //<<
#define CO_OD_IS_ORDERED
#define CO_SAVE_EEPROM
//#define CO_SAVE_ROM                //<<
```

未使用 SYNC 对象，使用 2 个RPDO，一个用来接收采集单元发送的温度值，另一个用来接收动力装置发送的状态。未使用 TPDO 和监控。

创建 NMT 主机时将用到一个自定义的 CANtx 消息。

对象字典中的变量最大限制为 4 个字节，因此不用使用SDO的分段传输。

PDO 参数不在对象字典中存在。它们仍将作为变量出现，但不会被访问。闪存占用将会减小，因为变量不会被添加到 `CO_OD[]` 数组中。

未使用映射，变量也不存在。在这种情况下，如果我们有 TPDO，它的数据长度将固定为 8 个字节。由于我们有 RPDO，所以任何数据长度都可以被接受（accepted）。

宏 `CO_SAVE_ROM` 被禁用，所以 SDO 客户端将无法写入位于闪存中的变量。例如，位于索引 0x1017 处的 `Producer_Heartbeat_Time` 将不可写。
出于这个原因，我们也可以从项目中删除文件 *memcpyram2flash.h* 和 *memcpyram2flash.c* 。

*Default values for object dictionary*   
「对象字典默认值」部分的代码保持缺省。

*0x1400 Receive PDO parameters*  
「接收 PDO 参数」部分代码：

```c
#define ODD_RPDO_PAR_COB_ID_0   0x40000286L //<<
#define ODD_RPDO_PAR_T_TYPE_0   255     
#define ODD_RPDO_PAR_COB_ID_1   0x40000187L //<<
#define ODD_RPDO_PAR_T_TYPE_1   255
```

该节点将直接从采集单元（COB-ID = 0x286）和动力装置（COB-ID = 0x187）接收 PDO。

*Default values for user Object Dictionary Entries*  
「用户对象字典入口默认值」部分代码：

```c
#define ODD_CANnodeID   0x08    //<<
#define ODD_CANbitRate  3
```

操作单元的 Node-ID 为 8，CAN 比特率为 125 kbps。





待续……


    2018/3/23 更新，很快就要完成了。


[L1]:https://li-qw.github.io/2018/03/21/CANopenNode-learn-1/  
[L2]:https://li-qw.github.io/2018/03/22/CANopenNode-learn-2/  
[L3]:https://li-qw.github.io/2018/03/23/CANopenNode-learn-3/  

[R1]:https://sourceforge.net/projects/canopennode/files/canopennode/CANopenNode-1.10/ "《CANopenNode Turorial》  V1.10"  
[R2]:https://sourceforge.net/projects/canopennode/files/canopennode/CANopenNode-1.10/ "《CANopenNode Manual》 V1.10"  
[R3]:mailto:janez.paternoster@siol.net "作者 Janez Paternoster"
[R4]:http://www.winmerge.org/ "文件比较工具"
[P1-1]:https://us1.myximage.com/2018/03/22/1184ad33502524a63969c282b9040d7f.png "简单的CANopen网络"
[P2-1]:https://us1.myximage.com/2018/03/22/505b807b391ee5b3b38ae4a5c772b07a.png "CAN收发器与PIC MCU的连接"
[P4-1]:https://us1.myximage.com/2018/03/23/c20cbfc03670fbc8a0361795ba7f9e09.png "三个节点的CANopen网络连接"

[P3-1]:https://us1.myximage.com/2018/03/22/b34e867948786dfb5afd0e0824a6152c.png "路径设置"