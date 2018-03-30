---
layout: wiki
title: 常用函数和变量命名规则
categories: base
description: 常用函数和变量命名法总结
keywords: 基础, 命名法
---

> 良好的命名习惯，不但可以增加程序可读性，也可以有效地提高程序的可维护性。正确而形象的命名，也是编程风格的一种体现。

# 几种常见的命名规则


- 匈牙利命名法
- 驼峰命名法
- 帕斯卡命名法（大驼峰法）
- 下划线命名法

## 匈牙利命名法

由 Microsoft 程序员查尔斯 · 西蒙尼(Charles Simonyi) 提出。主要思想是「在变量和函数名中加入前缀以增进人们对程序的理解」。

显著的特点是：标识符的名字以一个或多个小写字母开头作为前缀，用以标识出变量的作用域，类型等；前缀之后的是首字母大写的一个单词或多个单词的组合，该单词主要指明变量的用途。

例如：`lpszStr`, 表示指向一个以 `\0` 结尾的字符串（sz）的长指针（lp）变量。

| 前缀  | 类型                   | 描述               |
| ----- | ---------------------- | ------------------ |
| a     | Array                  | 数组               |
| b     | BOOL                   | 布尔               |
| by    | BYTE                   | 无符号字符         |
| c     | char                   | 字符               |
| cb    | Count of bytes         | 字节数             |
| cr    | Color reference value  | 颜色值             |
| cx,cy | Count of x,y(short)    | 长度               |
| dw    | DWORD                  | 双字(无符号长整形) |
| f     | Flags                  | 标志               |
| fn    | Function               | 函数               |
| g_    | Global                 | 全局的             |
| h     | HANDLE                 | 句柄               |
| i     | Integer(int)           | 整数               |
| l     | Long(long)             | 长整数             |
| lp    | Long point             | 长指针             |
| m_    | Data member of a class | 类的数据成员       |
| n     | Short(short)           | 短整型             |
| np    | Near point             | 短指针             |
| p     | Point                  | 指针               |
| s     | String                 | 字符串             |
| sz    | Zero terminated string | 以\0结尾的字符串   |
| tm    | Text metric            | 文本规则           |
| u     | Unsigned int           | 无符号整数         |
| ul    | Unsigned long(ULONG)   | 无符号长整数       |
| w     | WORD                   | 无符号短整数       |
| x,y   | x,y coordinates(short) | 坐标               |
| v     | Void                   | 空                 |

有关项目的全局变量用 `g_` 开始，类成员变量用 `m_`。

| 前缀 | 类型     | 例子                  |
| ---- | -------- | --------------------- |
| C    | 类       | CDocument, CPrintInfo |
| m_   | 成员变量 | m_pDoc, m_nCustomers  |
| g_   | 全局变量 | g_Servers             |

## 驼峰命名法

Java 上使用较为广泛，近些年来愈加流行。正如它的名称所表示的那样，其特点是：混合使用大小写字母来构成标识符的名字；其中第一个单词首字母小写，余下的单词首字母大写；函数名中每一个逻辑断点都有一个大写字母来标记。

例如：`printEmployeePaychecks()`

## 帕斯卡命名法（大驼峰法）

帕斯卡命名法与驼峰法类似，只不过骆驼命名法是第一个单词首字母小写，而帕斯卡命名法第一个单词首字母是大写。故而，这种命名法也被称为「大驼峰命名法」。

例如：`DisplayInfo()`; `UserName`

在C#中，以帕斯卡命名法和骆驼命名法居多。事实上，很多程序设计者在实际命名时会将骆驼命名法和帕斯卡结合使用，例如变量名采用骆驼命名法，而函数采用帕斯卡命名法。

## 下划线命名法

下划线法是随着C语言的出现流行起来的，在UNIX/LIUNX这样的环境，以及GNU代码中使用非常普遍。

### 函数命名

采用下划线分割小写字母的方式命名，主要格式为：

    设备名_操作名()

操作名一般采用：
- 谓语，此时设备名作为宾语或者标明操作所属的模块
- 谓语 + 宾语／表语，此时设备名作为主语或者标明操作所属的模块

如：  

    tic_init()
    adc_is_busy()
    uart_tx_char()

中断函数的命名直接使用 `设备名_isr()` 的形式，如：

    timer2_isr()

### 变量的命名

同样，变量的命名也采用下划线分割小写字母的方式。命名应准确，不引起歧义，且长度适中。如：

    int length
    uint32 test_offset

单字符的名字常用于函数内部的循环计数，作为局部变量。tmp 常用于作为临时变量。局部静态变量应加 `s_` 前缀，如：

    static int s_lastw

全局变量（特别是供外部访问的），应加 `g_` 前缀,如：

    void (* g_capture_hook)(void)

### 常量及宏的命名

采用下划线分割大写字母的方式命名，一般应以设备名作为前缀，防止模块间命名的重复。如：

    #define TIMER0_MODE_RELOAD 2
    #define TIMER2_COUNT_RETRIEVE(val) ((uint16)(65536 - (val)))

当然，看作接口的宏可以按照函数的命名方法命名，例如：

    #define timer2_clear()          (TF2 = 0)
    #define timer0_is_expired()     (TF0)

# 嵌入式 C 语言命名

标识符的命名要清晰、明了，有明确含义。使用完整的单词或大家基本可以理解的缩写，避免使阅读者产生误解。

标识符应采用英文单词或其组合，切忌使用汉语拼音。

坏的命名：
    int a
    Age1
    XueshengAge

好的命名：
    int StudentAge;

为了防止某一软件库中的一些标识符和其它软件的冲突，可以为各种标识符加上能反应软件性质的前缀。例如三维图形标准 OpenGL 的所有库函数均以 `gl` 开头，所有常量或宏定义均以 `GL` 开头。

## 变量名

### 不同作用域变量的命名

- 局部变量，小写字母命名；（下划线法）
- 全局变量，首字母大写方式命名；（帕斯卡式）
- 定义类型和宏定义常数以大写字母命名；
- 变量的作用域越大，它的名字所带有的信息就应该越多。

```c
int student_age;        // 局部变量
int StudentAge;         // 全局变量
#define STUDENT_NUM 10  // 宏定义常数
typedef INT16S int;     // 类型定义
```

## 函数名

- 应像全局变量一样采用首字母大写方式，帕斯卡式；
- 变量和参数用驼峰式；
- 函数名开始应以 `模块名_` 的格式注明函数所属模块。

# 常用的缩略词

| 原词           | 缩写      |
| -------------- | --------- |
| addition       | add       |
| answer         | ans       |
| array          | arr       |
| average        | avg       |
| buffer         | buf或buff |
| capture        | cap或capt |
| check          | chk       |
| count          | cnt       |
| column         | col       |
| control        | ctrl      |
| decode         | dec       |
| define         | def       |
| delete         | del       |
| destination    | dst或dest |
| display        | disp      |
| division       | div       |
| encode         | enc       |
| environment    | env       |
| error          | err       |
| float          | flt       |
| frequency      | freq      |
| header         | hdr       |
| index          | idx       |
| image          | img       |
| increment      | inc       |
| initalize      | init      |
| iteration      | itr       |
| length         | len       |
| memory         | mem       |
| middle         | mid       |
| make           | mk        |
| message        | msg       |
| multiplication | mul       |
| number         | num       |
| operand        | opnd      |
| optimization   | opt       |
| operator       | optr      |
| packet         | pkt       |
| positon        | pos       |
| previous       | pre或prev |
| payload type   | pt        |
| pointer        | ptr       |
| return code    | rc        |
| record         | rcd       |
| receive        | recv      |
| result         | res       |
| return         | ret       |
| source         | src       |
| stack          | stk       |
| string         | str       |
| subtraction    | sub       |
| table          | tab       |
| temporary      | tmp或temp |
| total          | tot       |
| time stamp     | ts        |
| value          | val       |

# 现状

没有一种命名规则可以让所有的程序员赞同。而这多种命名规则也确实各有利弊。

没有必要花太多的精力试图发明最好的命名规则，而是应当制定一种令大多数项目成员满意的命名规则并切实执行。标识符命名的一致性自然会体现出代码的优雅。

当然，如果你的程序使用了第三方的代码，而这些模块经验证确实是正确无误的。
那么也没有必要一味追求命名的一致性，而去修改这些已经定型的模块中的函数和变量名。

# 词穷了怎么办？

> There are two hard things in Computer Science:cache invalidation and naming things.
> 计算机科学的两件难事：缓存失效和命名。

如何不改变我们的平时的习惯就能日积月累的收获变量命名的经验呢？
回顾我们平时的习惯：1.查单词；2. 比较单词语义；3. 比较代码上下文；4.确定命名。

只要把这几个步骤缩短就能节省大量时间。[CODELF](http://unbug.github.io/codelf/) 就是帮我们缩短这些步骤的一个变量名搜索工具。

支持直接搜索中文，当你查中文的时候，Codelf 会直接查好单词和单词的近义词给你，然后再搜索Github, Bitbucket, Google Code, Codeplex, Sourceforge, Fedora Project上的开源项目的源码匹配出与这些词汇相关的变量名和函数名。




