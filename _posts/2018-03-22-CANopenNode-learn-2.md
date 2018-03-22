---
layout: post
title: CANopenNode学习（2/2）
date: 2018-3-22
categories: blog
tags: [CANopen，CANopenNode，学习，代码，]
description: 学习使用CANopenNode开源库开发CANopen网络。
---
> 学习CANopenNode的最终目的是：结合其API在PIC18F单片机上开发CANopen通信节点，并采集、传递和处理信号。  
> 英语及专业水平有限，如有纰漏和表达不周，请及时与我联系，谢谢。

文中使用的CANopenNode V1.1源码和文件可以在[这里下载](https://sourceforge.net/projects/canopennode/files/canopennode/CANopenNode-1.10/)   
新的工程已经移步到[GitHub](https://github.com/canopennode)可以支持PIC32等更多的MCU。

由于v1.1版本涵盖了PIC18F的实例和工程文件，并有详细的手册和使用指南，所以选择v1.1版本作为入门学习。下面的章节主要依据 [教程][R1]。

CANopenNode V1.1采用许可`GNU Free Documentaion License`.

接上文：[CANopenNode学习（1/2）][L0]

### 3.4 功率单元的编程




    2018/3/22更新，很快就要完成了。

:snail:
 
[L0]:https://li-qw.github.io/blog/2018/03/21/CANopenNode-learn/ "上一部分" 
[R1]:https://sourceforge.net/projects/canopennode/files/canopennode/CANopenNode-1.10/ "《CANopenNode Turorial》  V1.10"  
[R2]:https://sourceforge.net/projects/canopennode/files/canopennode/CANopenNode-1.10/ "《CANopenNode Manual》 V1.10"  
[R3]:mailto:janez.paternoster@siol.net "作者 Janez Paternoster"
[R4]:http://www.winmerge.org/ "文件比较工具"
[P1-1]:https://us1.myximage.com/2018/03/22/1184ad33502524a63969c282b9040d7f.png "简单的CANopen网络"
[P2-1]:https://us1.myximage.com/2018/03/22/505b807b391ee5b3b38ae4a5c772b07a.png "CAN收发器与PIC MCU的连接"
[P3-1]:https://us1.myximage.com/2018/03/22/b34e867948786dfb5afd0e0824a6152c.png "路径设置"