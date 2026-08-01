# ==STM32文件讲解与使用方法==





[TOC]



## 一.对于新建工程文件的讲解



[TOC]



### 1.文件夹中的一些文件

| Drivers                  | 用于存放硬件相关的底层驱动文件                               |
| ------------------------ | ------------------------------------------------------------ |
| **Listings**             | **存放编译器编译时产生的C/汇编/链接的列表清单**              |
| **Output**               | **存放编译产生的调试信息、HEX文件、预览信息，封装库等**      |
| **Project**              | **用来存放工程**                                             |
| **System**               | **用来系统自带的驱动文件**                                   |
| **User**                 | **用来存放用户自己编写的驱动文件**                           |
| **STM32F1xx_HAL_Driver** | **存放标准库的.c和.h文件，其中inc放置.h文件，src放置.c文件** |



### 2.Drivers/CMISS

#### （1）、什么是Drivers/CMISS

CMSIS驱动程序规范是一个软件API，它描述中间件和用户应用程序的外围驱动程序接口。CMSIS-Driver属于底层硬件与上层中间件之间的代码层，它隔离了底层的不同硬件确保了对上层中间件统一的接口大大提高了软件的可移植性。

**总结：**Drivers/CMISS的作用是提高不同硬件之间的代码移植性。



#### （2）、为什么需要CMSIS-Driver?

CMSIS驱动程序API设计为通用的，独立于特定的RTOS，使其可在各种受支持的微控制器设备上重用。简而言之就是提高程序的利用率和可移植性。



#### （3）、怎么使用CMSIS-Driver?

1. 通过MDK中的Manage Run-Time Environment工具可以非常方便的以窗口向导的形式来添加各种代码组件其中就包括CMSIS-Driver

   ![Manage Run-Time Environment](https://s2.loli.net/2023/10/28/Ci4qRVvDjLdkQfT.png)

2. 直接从对应的芯片pack的安装目录(如：\ARM\PACK\Keil\STM32F4xx_DFP\2.10.0\CMSIS\Driver)中将对应源文件与头文件复制到自己的工程中

3. 使用自定义的基于CMSIS标准的CMSIS-Driver文件。



**总结：**Drivers/CMISS就是一个驱动文件库，里面包含了我们驱动程序，我们可以直接安装pack库也可以自己在Manage Run-Time Environment中寻找并配置。



### 3.Drivers/CORE

CORE 文件夹包含的是一些内核相关的函数和宏定义，例如核内寄存器定义、部分核内外设的地址等等，这些都是非常底层的函数，上层的一些函数直接调用它们了，别名固件库，初学者不用太关心。

**总结：**用来存放核心文件和启动文件，并配置寄存器。我们STM32单片机的所有操作都是对寄存器的操作，所以没有寄存器就无法使用我们的单片机。



### 4.HALLIB

HALLIB文件夹包含我们的HAL库中的所需库函数，通过添加不同的库函数驱动不同的外设，其中以下库函数是必须需要添加的

![库函数](https://s2.loli.net/2023/10/28/OZ6Ta5372DHkcPR.png)

在后续的培训中，我们还会相继引用许多库函数，例如:

```c
stm32f1xx_hal_tim.c
stm32f1xx_hal_uart.c
stm32f1xx_hal_exti.c
```

等等。



### 5.USER

USER文件夹用来存放用户自己编写的驱动文件，例如我们驱动LED的文件，驱动小车的文件，驱动蜂鸣器的文件，等等。



### 6.System

System文件夹用来存放一些系统自带的驱动文件，一般就是sys.c和delay.c两个文件，用于我们的延时。





## 二.文件的命名：

​	在[嵌入式开发](https://so.csdn.net/so/search?q=嵌入式开发&spm=1001.2101.3001.7020)中，通常会使用C语言编写程序。C语言的程序通常被分成两个文件：.c文件和.h文件。

那么两种类型的文件分别存储怎样的代码呢？

### ①.c文件

（1）==**.c**==文件包含了程序的`实现部分`，其中包含了`函数的实现`和`变量的定义`等内容。==**.c**==文件是可以被编译成可执行文件的。

`led.c`

```c
#include "led.h"

// 定义 LED 状态变量
static LedStatus led_status = LED_OFF;

// 打开 LED
void led_open(void)
{
    led_status = LED_ON;
}

// 关闭 LED
void led_close(void)
{
    led_status = LED_OFF;
}
```



`main.c`

```c
#include "led.h"

int main(void)
{
    // 打开 LED
    led_open();
    return 0;
}
```



### ②.h文件

（2）==**.h**==文件包含了程序的`接口部分`，其中包含了`函数的声明`和`结构体的定义`等内容。这些代码不是可执行代码，而是`提供给其他模块使用的接口`。其他模块可以导入这些头文件，并通过调用头文件中声明的函数和定义的结构体来与该模块进行交互。

`led.h`

```c
/*如下为键盘驱动的头文件*/
#ifndef __LED_H_ 
//防重复引用，如果没有定义过_LED_H_，则编译下句
#define __LED_H_ 
//此符号唯一， 表示只要引用过一次，即我们的#include，则定义符号_LED_H_/

// 定义 LED 状态
typedef enum {
    LED_OFF = 0,
    LED_ON // 不赋值，会根据第一个值计算为 1
} LedStatus;

void led_open(void);	// 打开 LED
void led_up(void);		// 关闭 LED

#endif

//在#endif一定要空一行否则会有Warning:
//warning: no newline at end of file [-Wnewline-eof] #endif  /* __LED_H */
```



如果不懂为什么要加`#ifndef`,请看下面的解释：

```
当一个头文件被多次包含时，预处理器会将该头文件的内容复制到每个包含它的源文件中。
如果一个头文件被重复包含多次，就会导致重复定义的问题。
当第一次包含头文件时，头文件保护宏被定义，后续再包含头文件时，头文件保护宏已经被定义，预处理器会直接跳过头文件的内容。
头文件保护宏可以确保头文件只被包含一次，避免重复定义问题，同时也提高了编译速度
```



**总结：**

在嵌入式代码的开发中，我们遵循

1. 先编写 .h文件，如：led.h;
2. 再编写.c文件，如 led.c;
3. 在mian.c 或其他文件中导入 led.h 使用定义好的函数。



## 三.STM32注释风格：

基于STM32的Doxygen使用简明手册

①为了能使代码能够被Doxygen识别，必须遵循Doxygen的书写规则。注释必须以==/**==打头，以==*/==结束。



#### 一、添加类型

##### ==（1）、 添加首页(mainpage)：==

格式：

/**

  \mainpage RIOM DSP Software Library

  *

  \* <b>Introduction</b>

  *

  \* This user manual describes the CMSIS DSP software library

  */



**关键字：**

**\mainpage**

描述：

用以显示在首页中，一般用于对整个工程进行描述。

![mainpage](https://s2.loli.net/2023/10/28/uVJP4MUCnmD6Fr5.png)

##### **（2）、 添加define分组(defgroup)：**

格式：

/** @defgroup ZHM2

 \* @{

 */

\#define XXX YYY

/**

 \* @}

 */

关键字：

**@defgroup name**

**@{**

**@}**

描述：

定义一个define分组，用以显示在生成的文件中，一般多出现在.h文件中。



##### **（3）、 添加到分组(addtogroup)**

格式：

/** @addtogroup STM32F2xx_StdPeriph_Driver

 \* @{

 */

XXXX

/**

 \* @}

 */

关键字：

**@addtogroup name**

**@{**

**@}**

描述：

把一些东西添加到某个分组中去，该分组可以定义在其他文件下，Doxygen会自动搜索该分组，然后将需要添加的添加到该分组。可以进行跨文件关联。

通过addtogroup可以形成树结构，如果原来不存在该分组，它会自动新建该分组，然后添加到该分组。



##### ==（4）、 文件注释：==

格式：

/**

******************************************************************************

 \* @file  main.c

 \* @author name

 \* @version V1.0.0

 \* @date  2023/10/25

 \* @brief  This file provides all the detail functions.

******************************************************************************

 \* @copy

 \* 

 \* THE PRESENT FIRMWARE WHICH IS FOR GUIDANCE ONLY AIMS AT PROVIDING CUSTOMERS

 \* WITH CODING INFORMATION REGARDING THEIR PRODUCTS IN ORDER FOR THEM TO SAVE

 \* TIME. AS A RESULT, STMICROELECTRONICS SHALL NOT BE HELD LIABLE FOR ANY

 \* DIRECT, INDIRECT OR CONSEQUENTIAL DAMAGES WITH RESPECT TO ANY CLAIMS ARISING

 \* FROM THE CONTENT OF SUCH FIRMWARE AND/OR THE USE MADE BY CUSTOMERS OF THE

 \* CODING INFORMATION CONTAINED HEREIN IN CONNECTION WITH THEIR PRODUCTS.

 *

 \* <h2><center>© COPYRIGHT 2010 STMicroelectronics</center></h2>

 */

关键字：

@file：文件名，xx.c; zz.h等

@author：作者

@version：版本号

@date：日期

@brief：简介

@copy/@attention：详细描述

描述：

用以说明整个文件的各种信息。

![brief](https://s2.loli.net/2023/10/28/DyrVFKgAQdPi69E.png)





## 四.Keil软件的使用方法：

#### 1.文件操作1

ctrl+N 添加文件

ctrl+S 保存文件



#### 2.文件操作2

鼠标右击文件点后：

点Close关闭该文件

点Close All But This关闭该文件以外的其他文件

点Close All关闭所有文件

点Copy Full Path复制该文件的目录位置

点Open Containing Folder打开该文件位置

点New Horizontal Tab Group 将代码界面分为上下，上面为其他文件，下面为该文件，再次右击该文件（该文件在下面），点击Move To Previous  Group，返回原始代码界面

点击New Vertical Tab Group 将代码界面分为左右，左面为其他文件，右面为该文件，再次右击该文件（该文件在右面），点击Move To Previous  Group，返回原始代码界面



#### 3.Options for Target

![Options for Target](https://s2.loli.net/2023/10/29/qUmi2VYpchjtKHI.png)

从图标看就是魔法棒

##### ***1.Device设备（器件）***

新建工程第一个就是选择设备（器件）。强调一点就是：器件可以通过输入查找，也可以通过列表查找。

![image-20231029155935412](https://s2.loli.net/2023/10/29/xyCkTNZwt3PFMGD.png)



##### 2.***Target（目标）***

从内容可以看得出来是工程目标的调试晶振频率、选择的编译器、RAM和ROM分配的地址空间等。

![image-20231029160611909](https://s2.loli.net/2023/10/29/VLbuj4xWrYwDKUp.png)

第1处：晶振频率
这个值主要用于仿真调试用，一般我们使用硬件调试可以不用管这个值。

第2处：操作系统
是否选择Keil自带的RTX操作系统，一般我们都不选。

第3处：系统预览文件
这里我们一般是默认使用系统自带，不选择自己定义的。

第4处：生成代码所选择的编译器，我们使用的是版本5

第5处：使用交叉模块优化、使用微库
交叉模块一般我们不使用，微库这个功能常用与printf函数。

第6处：ROM存储地址
这里的ROM存储指的是程序储存的地址，分片外和片内两种。
程序存储在片内好理解（初学者一般下载程序都是下载到片内FLASH）,片外存储程序对于初学者来说比较少见，一般都是项目做大了，或有特殊要求时，片内不够使用了才将程序存储在片外。

第7处：RAM存储地址
RAM存储地址和ROM道理一样，可以分片内和片外。



##### ***3.Output（输出）***

输出一系列相关的内容。输出分两类：
1.输出（创建）可执行文件，我们下载到处理器里面的程序就是该类；
2.输出库，对于初学者来说一般不使用库，但对于很多从事特殊行业技术开发的公司来说，可能比较常用该功能。

![image-20231029161219384](https://s2.loli.net/2023/10/29/cqdCTtszajH6lUb.png)

第1处：输出路径
输出路径就是在工程编译的过程中，输出这些文件保存的文件夹。Keil V5一般默认是保存在Objects文件夹下面，我建立工程一般也使用这个默认的路径。【其内容可以全部删除，最好配置在单独一个文件夹下面，代码备份时方便删除】



第2处：输出可执行文件名
输出的可执行文件和库的名称就是在这里定义。比如我们常见输出Hex文件，其名称就是这里定义的。



第3处：输出可执行文件（重点）
这里和输出库是二选一，选择了输出可执行文件就不能选择输出库。重要一点：输出这些信息都很费时间，如果都不勾选这些选项，编译速度会很快。
Debug Infomation：输出调试信息。勾选上这个选项，我们才可以进行调试。
Create HEX File：输出可执行Hex文件，很多初学的朋友问：“在哪里设置生成Hex?”，这里勾选上就行了。（在Objects里）
Browse Information：输出浏览信息。勾选上这个我们才能使用go to definition of这个功能。很多人问：“为什么我不能跟踪代码了”，原因就在这里



第4处：输出库
拓展一点：这里输出（生成）的是静态库，并非动态库。初学者可以不用去理解。



##### ***4.Listing（列表）***

这个选项是关于生成列表相关的选项，对代码分析比较透彻的工程师就需要了解这个选项。常见的就是map地址的分布，就是在这里配置生成的。

![image-20231029162412590](https://s2.loli.net/2023/10/29/DFmGuZIkwcRfHjA.png)

第1处：输出路径、宽高
选择列表文件输出的文件夹。可设置文件页面的宽度，长宽

第2处：输出汇编列表
勾选上会输出汇编列表信息（产生后缀为 .lst的文件）。如果工程中没汇编文件，则不会输出信息。

第3处：C编译列表
C编译程序列表选项，勾选上可生成.txt, .i文件。

第4处：链接列表
可选择生成或禁止生成.map文件。可设置生成代码的详细信息。可选择性的选取输出MAP文件。



##### ***5.User（用户选项）***

这个选项是针对用户而设计的，一般不常用，方便用户执行一些程序。比如：编译完代码之后，我要将生成的Hex文件拷贝到其它地方。纵观下图可以看见，第1、2、3处作用相同，都是让用户运行程序，只是运行的条件不同而已。上面说的用户程序，勾选上，可以“DOS16模式”运行。

![user](https://s2.loli.net/2023/10/29/CuS3p7fLtZ812IY.png)

第1处：编辑之前运行用户程序。

第2处：编译之前运行用户程序。

第3处：编译之后运行用户程序。

第4处：编译之后执行条件。

Run “After Build” conditionally：执行条件

Beep When Complete：编译完成发出声音

Start Debugging：启动调试程序



##### 6.***C/C++（选项）***

这后面五项中，C/C++选项最为重要，因此部分功能需要重点强调。看选项标题“C/C++”，针对的主要就是C/C++，和后一个选项“Asm”有类似之处。

![img](https://s2.loli.net/2023/10/29/mJBft5jLYEhqDOu.png)

第1处：预处理（Preprocessor Symbols）
这里主要就是预定义功能，相当于在程序中的#define xxxx。我上面预定义USE_HAL_DRIVER,STM32F103xB，文件中就不用定义了。



第2处：语言代码生成（Language / Code Generation）
**Language/Code Generation**语言代码生成，可以理解成编译、链接到最后生成代码。这部分功能对于代码优化比较重要，初学者可以不用过多理解，对代码大小、运行速度等性能要求较高的人就需要深入理解
**Execute only Code：**只生成执行代码
[设置编译器命令行：–execute_only]只生成执行代码防止编译器生成任何数据访问代码部分。
**Optimization：**优化选择项，有Level0 - Level3四个选项
[设置编译器命令行：-Onum]初学者、在线调试建议使用Level0，也就是不优化，这样执行的效果才和代码一样。如果配置成Level3，在线调试可能有些地方优化而不能打断点（断点就是一步一步的执行代码）。
**Optimization for Time：**优化时间，即优化代码中费时的地方。
[设置编译器命令行：-Otime] --split_sections比如有些算法，本身代码量就比较大，运行需要很长时间（假如需要2秒），这个时候勾选上该功能，会发现运行时间有比较明显的减少（或许不到1秒时间）。
**Split Load and Store Multiple：**加载和存储多个分裂。
[设置编译器命令行：–split_ldm]非对齐数据采用多次访问方式。当 LMD/STM 指令有 4 个以上产生时，列分裂LMD 和 STM 指令，以减不中断延迟。
**One ELF Section per Function：**优化每一个函数 ELF 段（建议都勾选上）。
[设置编译器命令行：–split_sections]每个函数都会产生一个 ELF 段，勾选上，允许优化每一个 ELF 段。这个选项可以减少潜在的共享地址、数据和函数之间的字符串。
直白的意思：可以减少代码量ROM的大小（内存RAM不会减小）。
举一个例子，勾选之前和勾选之后，编译后存储大小对比：
勾选之前：
Program Size: Code=2540 RO-data=336 RW-data=40 ZI-data=1024
勾选之后：
Program Size: Code=908 RO-data=320 RW-data=40 ZI-data=1024
**Strict ANSI C：**标准（严格）的ANSC
[设置编译器命令行：–strict]也就是说：编译时严格按照标准的ANSI C进行检查。
**Enum Container always int:**枚举总是int型
[设置编译器命令行：–enum_is_int]很容易理解，我们枚举时成员变量类型为int型。
**Plain Char is Signed：**纯字符标记为字符
[设置编译器命令行：–signed_chars]代码举例：char a[] = “abcd”; 也就是说将“abcd”标记为字符型。
**Read-Only Position Independent：**为常量生成独立的代码空间。
[设置编译器命令行：–apcs=/ropi]比如：我们定义字库变量为常量，勾选该选项，会将这些字库变量放在独立的代码空间。
Read-Write Position Independent：为可读写代码生成独立的代码空间。
[设置编译器命令行：–apcs=/rwpi]
**Warnings：**警告
[No Warnings设置编译器命令行：-W]
No Warnings：不会有警告提示和输出；
All Warnings：所有警告提示和输出。
Thumb Mode：Thumb模式。
指定设置文件或文件夹（组）为Thumb模式。【注意：在工程中该模式为默认，也就是不能选择】
**No Auto Includes：**不自动添加头文件（一般不勾选）。
不勾选该选项，编译器就会在Keil安装路径寻找你工程中.h文件。
举例：我们定义uint8_t是定义在stdint.h文件里面的，但是我们工程目录下一般是没有stdint.h文件。这时候，编译器就会在Keil路径下去寻找stdint.h文件。
**C99 Mode:**C99标准模式。
[设置编译器命令行：–c99]C语音有标准有多个版本，如C89、C90、C99等。



第3处：包含路径（Include Paths）
包含路径是使用Keil（及类似）软件必须掌握的一项。包含路径就是指定我们工程中使用文件所在的位置，让编译器找到相应的文件。是初学者、高级软件工程师都必须掌握的一项。



第4处：多功能控件（Misc Controls）
指定没有单独的对话框控件。例如：错误消息用日本语言来显示消息。【不常用】



第5处：编译器控制字符串（Compiler control string）
这里是针对编译器执行的命名，显示当前在编译器命令行指令。
在上面“第2处：语言代码生成”中有一个中括号【设置编译器命令行：里面的命名就显示在这里。



##### ***7.Asm（选项）***

和前面一个选项“C/C++”类似，只是这里针对的是Asm。

![img](https://s2.loli.net/2023/10/29/tyklx56gRvK8hwu.png)

第1处：有条件的装配控制符号（Conditional Assembly Control Symbols）
指定汇编条件，这里类似上一章节C/C++选项中的预处理。

第2处：语言代码生成（Language / Code Generation）和上一章节类似。
Read-Only Position Independent：为常量生成独立的代码空间。
Read-Write Position Independent：为可读写代码生成独立的代码空间。
Thumb Mode：Thumb模式。
Split Load and Store Multiple：加载和存储多个分裂。
Execute only Code：只生成执行代码。
No Auto Includes：不自动添加头文件（一般不勾选）。

后面和C/C++同理



##### ***8.Linker（选项）***

这个选项Linker链接，也就是是链接器配置选项。可以修改、编辑和查看链接的文件。第1、2处是重点，第3、4处和C/C++选项一样。

![img](https://s2.loli.net/2023/10/29/3CvUfLExpN4cR8d.png)

第1处：使用分散文件加载对话框Target页面（Use Memory Layout from Target Dialog）
Make RW Sections Position Independent：使RW段独立
【设置编译器命令行：–rwpi】
启用时：变量区域（包含RW和ZI）具有独立地址。
禁用时：变量区域（包含RW和ZI）位于绝对的内存地址。
Make RO Sections Position Independent：使RO段独立
【设置编译器命令行：–ropi】
启用时：常量和代码区域（RO）具有独立地址。
禁用时：常量和代码区域（RO）位于绝对的内存地址。
Don’t Search Standard Libraries：不搜索标准库
【设置编译器命令行：–noscanlib】
禁用默认编译器运行时库的扫描。
Report ‘might fail’ Conditions as Errors：报告’might fail’条件认为是错误
【设置编译器命令行：–strict】
报告的条件可能导致失败的错误，而不是警告。
X/O Base：X/O基地址
【设置编译器命令行：–xo_base=address】
R/O Base：R/O基地址
【设置编译器命令行：–ro_base=address】
R/W Base：R/W基地址
【设置编译器命令行：–rw_base=address】
disable Warnings:警用警告
【设置编译器命令行：–diag_suppress】



第2处：分散文件（Scatter File）
这里可以加载、查看和编辑分散文件。点击后面就的三点“…”可以加载文件；点击“Edit…”查看和编辑对应的文件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528214558118.png)





##### ***9.Debug（选项）***

这个选项比较重要，主要用于（软件仿真、硬件在线）调试使用。由于软件仿真和硬件在线调试配置界面基本一致，而现在我们基本都是硬件在线调试。因此，只选择硬件在线调试界面进行讲述。

![img](https://s2.loli.net/2023/10/29/XUm8lgzkqC5wbIJ.png)

第1处：选择硬件在线调试
下载调试器的选择不用多说，主要说一下后面“Setting”。很多人常用J-Link下载调试器，而调试STM32时，可以使用四线SWD模式。如果使用J-Link进行SWD调试。这个时候就需要在“Setting”里面选择“SW”模式。



第2处：选择硬件在线调试
**Load Application at Startup：**启动时加载应用程序
**Run to main()：**程序执行到main()函数
进入调试模式时，程序自动运行到main函数处
**Initialization File：**加载、编辑初始化文件
这里在某些情况下可以使用，比如：在RAM中调试代码



第3处：复位调试会话设置（Restore Debug Session Settings）
这里复位设置就是恢复设置的意思，如果勾选上，点击一下“复位”就会恢复到之前的状态。包括：断点Breakpoints、窗口Watch Windows、性能分析器 Performance Analyzer、内存窗口Memory Window、工具箱Toolbox、系统查阅器System Viewer等。



第4处：DLL文件（最好默认）
这里的配置属于Keil自身的配置，最好不要修改。
CPU/Driver DLL - Parameter：CPU驱动文件和参数
Dialog DLL - Parameter：会话框DLL文件和参数



第5处：管理组件描述文件
Manage Component Viewer Description Files这里一般不用去管理。



##### ***10.Utilities（选项）***

![image-20231029171652602](https://s2.loli.net/2023/10/29/Yx2hiz69BJ3Cnfu.png)

第1处：配置FLASH菜单命名（Configure Flash Menu Command）
这里是二选一选项，一般我们使用上面的“Use Target Driver for Flash Programming”。
Use Debug Driver：使用调试驱动
Update Target Before Debugging：调试之前更新目标
一般都勾选上，因为我们下载程序之前检测到代码修改了，就会重新编译程序（也就是更新目标）
Setting：设置
很多人下载程序之后，需要复位一下程序才运行，原因在于没有勾选“Reset and Run”，如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528214947365.png)



第2处：配置图像文件的处理（Configure Image File Processing）

这个选项不常用，感兴趣的话，自行了解。



#### 4.如果代码界面发生变化：

![image-20231029172440035](https://s2.loli.net/2023/10/29/yDbFA2dLSTkQ13W.png)

点**Reset View to Defaults**恢复默认界面。

