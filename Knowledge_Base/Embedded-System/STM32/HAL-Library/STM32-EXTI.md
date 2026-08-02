---
title: STM32-EXTI
date: 2026-08-01
tags: [Embedded, STM32, HAL]
aliases: []
---

# ==STM32第四次培训—EXTI==



[TOC]



## 一.什么是中断？

![什么是中断](https://s2.loli.net/2023/11/10/vnpAfsuMgcjlQzr.png)

简单理解的话，这就是中断。

以下是对于中断的详细介绍：



![什么是中断](https://img-blog.csdnimg.cn/img_convert/daf264a3945cfadb190e1e062b4afa68.png)



## 二.**中断的作用和意义**

作用：

1.实时控制：在确定时间内对相应事件作出响应，如：温度监控。

2.故障处理：检测到故障，需要第一时间处理，如：电梯门夹人了。

3.数据传输：不确定数据何时会来，如：串口数据接收。

意义：

中断的意义：**高效**处理紧急程序，不会一直占用CPU资源。



## 三.NVIC

![GPIO外部中断简图](https://s2.loli.net/2023/11/10/bAzYXIiZvHChqSG.png)

**在正式学习外部中断前，我们需要了解一些NVIC基础知识，不需要掌握**

#### 1.NVIC基本概念

**N**ested **V**ectored **I**nterrupt **C**ontroller，嵌套向量中断控制器，属于内核（M3/4/7）

NVIC支持：256个中断（16内核 + 240外部），支持：256个优先级（执行顺序），**允许裁剪！**（裁剪就是不用那个）

但 STM32 并没有使用 CM3 内核的全部东西，而是只用了它的一部分。STM32 有 84 个中断，包括 16 个内核中断和 68 个可屏蔽中断，具有 16 级可编程的中断优先级。而我们常用的就是这 68 个可屏蔽中断，但是 STM32 的 68 个可屏蔽中断，在 STM32F103 系列上面，又只有 60 个（在 107 系列才有 68 个）。

![NVIC](https://s2.loli.net/2023/11/10/XUc9RHwSyWkA46g.png)



总结：我们只需要知道STM32F103系列只用了10个内核中断（保留6个），外部中断60个（保留180个），16个中断级（16个执行的级别）

对于M3/M4/M7内核的MCU，每个中断的优先级都是用寄存器中的8位来设置的。8位的话就可以设置2^8^ = 256级中断，实际中用不了这么多，所以芯片厂商根据自己生产的芯片做出了调整。比如ST的STM32F1xx，F4xx和H7只使用了这个8位中的高四位[7:4]，低四位取零，这样2^4^=16，只能表示16级中断嵌套。



**总结：我们需要的学习的外部中断包含在NVIC里。**



我们知道了NVIC后我们需要知道我们的**中断服务函数**，也就是我们发生中断的**入口**。而中断服务函数又定义在**中断向量表**里

那什么又是中断向量表呢？



#### 2.中断向量表

定义一块固定的内存，以4字节对齐（STM32是32位），**存放各个中断服务函数程序的首地址**

中断向量表定义在**启动文件（.s文件）**，当发生中断，CPU会自动执行对应的中断服务函数

![中断服务函数](https://s2.loli.net/2023/11/10/lukre7ZtUDfEQsF.png)

我们正常是main函数占用我们的CPU，而发生中断后，我们的中断服务函数则会争夺CPU的使用权。

```c
__Vectors       DCD     __initial_sp               ; Top of Stack
                DCD     Reset_Handler              ; Reset Handler
                DCD     NMI_Handler                ; NMI Handler
                DCD     HardFault_Handler          ; Hard Fault Handler
                DCD     MemManage_Handler          ; MPU Fault Handler
                DCD     BusFault_Handler           ; Bus Fault Handler
                DCD     UsageFault_Handler         ; Usage Fault Handler
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     SVC_Handler                ; SVCall Handler
                DCD     DebugMon_Handler           ; Debug Monitor Handler
                DCD     0                          ; Reserved
                DCD     PendSV_Handler             ; PendSV Handler
                DCD     SysTick_Handler            ; SysTick Handler

                ; External Interrupts
                DCD     WWDG_IRQHandler            ; Window Watchdog
                DCD     PVD_IRQHandler             ; PVD through EXTI Line detect
                DCD     TAMPER_IRQHandler          ; Tamper
                DCD     RTC_IRQHandler             ; RTC
                DCD     FLASH_IRQHandler           ; Flash
                DCD     RCC_IRQHandler             ; RCC
                DCD     EXTI0_IRQHandler           ; EXTI Line 0
                DCD     EXTI1_IRQHandler           ; EXTI Line 1
                DCD     EXTI2_IRQHandler           ; EXTI Line 2
                DCD     EXTI3_IRQHandler           ; EXTI Line 3
                DCD     EXTI4_IRQHandler           ; EXTI Line 4
                DCD     DMA1_Channel1_IRQHandler   ; DMA1 Channel 1
                DCD     DMA1_Channel2_IRQHandler   ; DMA1 Channel 2
                DCD     DMA1_Channel3_IRQHandler   ; DMA1 Channel 3
                DCD     DMA1_Channel4_IRQHandler   ; DMA1 Channel 4
                DCD     DMA1_Channel5_IRQHandler   ; DMA1 Channel 5
                DCD     DMA1_Channel6_IRQHandler   ; DMA1 Channel 6
                DCD     DMA1_Channel7_IRQHandler   ; DMA1 Channel 7
                DCD     ADC1_2_IRQHandler          ; ADC1_2
                DCD     USB_HP_CAN1_TX_IRQHandler  ; USB High Priority or CAN1 TX
                DCD     USB_LP_CAN1_RX0_IRQHandler ; USB Low  Priority or CAN1 RX0
                DCD     CAN1_RX1_IRQHandler        ; CAN1 RX1
                DCD     CAN1_SCE_IRQHandler        ; CAN1 SCE
                DCD     EXTI9_5_IRQHandler         ; EXTI Line 9..5
                DCD     TIM1_BRK_IRQHandler        ; TIM1 Break
                DCD     TIM1_UP_IRQHandler         ; TIM1 Update
                DCD     TIM1_TRG_COM_IRQHandler    ; TIM1 Trigger and Commutation
                DCD     TIM1_CC_IRQHandler         ; TIM1 Capture Compare
                DCD     TIM2_IRQHandler            ; TIM2
                DCD     TIM3_IRQHandler            ; TIM3
                DCD     TIM4_IRQHandler            ; TIM4
                DCD     I2C1_EV_IRQHandler         ; I2C1 Event
                DCD     I2C1_ER_IRQHandler         ; I2C1 Error
                DCD     I2C2_EV_IRQHandler         ; I2C2 Event
                DCD     I2C2_ER_IRQHandler         ; I2C2 Error
                DCD     SPI1_IRQHandler            ; SPI1
                DCD     SPI2_IRQHandler            ; SPI2
                DCD     USART1_IRQHandler          ; USART1
                DCD     USART2_IRQHandler          ; USART2
                DCD     USART3_IRQHandler          ; USART3
                DCD     EXTI15_10_IRQHandler       ; EXTI Line 15..10
                DCD     RTC_Alarm_IRQHandler        ; RTC Alarm through EXTI Line
                DCD     USBWakeUp_IRQHandler       ; USB Wakeup from suspend
__Vectors_End
```



通过.s文件我们便可以清楚的看到我们STM32F103C8T6芯片上的内核中断和外部中断。

其中**__Vectors**代表中断向量表开始， **__Vectors_End**代表中断向量结束。DCD代表定义以4字节对齐。



**总结：中断向量表存放各个中断服务函数程序的首地址。**



#### 3.NVIC相关寄存器介绍

我们需要知道的是我们所有的函数是操作的寄存器，所以我们需要知道这些寄存器

| NVIC相关寄存器                        | 位数 | 寄存器个数 | 备注                                  |
| ------------------------------------- | ---- | ---------- | ------------------------------------- |
| 中断使能寄存器（ISER）                | 32   | 8          | 每个位控制一个中断                    |
| 中断除能寄存器（ICER）                | 32   | 8          | 每个位控制一个中断                    |
| 应用程序中断及复位控制寄存器（AIRCR） | 32   | 1          | 位[10:8]控制优先级分组                |
| 中断优先级寄存器（IPR）               | 8    | 240        | 8个位对应一个中断，而STM32只使用高4位 |

NVIC还有：中断挂起，解挂，激活标志等非常用功能，不做介绍！



<img src="https://s2.loli.net/2023/11/11/6i4z1qUfkGFheOn.png" alt="image-20231111101438618"  />

从图中可以看到每个寄存器的作用，注意：AIRCR控制8种分组，但其实我们只有5个分组。



#### 4.STM32中断优先级基本概念

1，抢占优先级(pre)：高抢占优先级可以**打断**正在执行的低抢占优先级中断

2，响应优先级(sub)：当抢占优先级相同时，响应优先级高的先执行，但是**不能互相打断**

3，抢占和响应都相同的情况下，自然优先级越高的，先执行

4，自然优先级：中断向量表的优先级

5，数值越小，表示优先级越高



所以简单来说：

①数值越小，优先级越高。

②抢占优先级数值越低，越先执行。

③抢占优先级相同，响应优先级越低，越先执行。

④响应优先级相同，自然优先级越低，越先执行。

⑤只有抢占优先级可以打断



#### 5.STM32中断优先级分组

| 优先级分组 | AIRCR[10:8] | IPRx bit[7:4]分配 | 分配结果                     |
| ---------- | ----------- | ----------------- | ---------------------------- |
| **0**      | 111         | None ：[7:4]      | 0位抢占优先级，4位响应优先级 |
| **1**      | 110         | [7] ：[6:4]       | 1位抢占优先级，3位响应优先级 |
| **2**      | 101         | [7:6] ：[5:4]     | 2位抢占优先级，2位响应优先级 |
| **3**      | 100         | [7:5]   ：[4]     | 3位抢占优先级，1位响应优先级 |
| **4**      | 011         | [7:4] ：None      | 4位抢占优先级，0位响应优先级 |

从表中我们可以看到STM32只有5个中断优先级分组，那上表什么意思呢？我们通过下面的图简化以下：

![中断分组](http://6.eewimg.cn/news/uploadfile/2022/0411/20220411045513916.png)

**特别提示：一个工程中，一般只设置一次中断优先级分组。**



我们可以练习一下：

STM32中断优先级举例（假设分组是2）

| **编号** | **自然优先级** | **对应外设** | **抢占** | **响应** | **执行顺序** |
| -------- | -------------- | ------------ | -------- | -------- | ------------ |
| **3**    | 10             | RTC          | 2        | 1        | 2            |
| **6**    | 13             | EXTI0        | 3        | 0        | 4            |
| **7**    | 14             | EXTI1        | 2        | 0        | 1            |
| **-1**   | 6              | Systick      | 3        | 0        | 3            |

EXTI1和RTC可以打断：EXTI0和Systick的中断，获得优先执行！

参考：STM32F10xxx参考手册（中文版）.pdf，9.1.2节，表55



**以上为了解内容，以下为掌握内容：**



#### 6.STM32-NVIC的使用

①设置中断分组——AIRCR[10:8]，HAL_NVIC_SetPriorityGrouping

②设置中断优先级——IPRx bit[7:4]，HAL_NVIC_SetPriority

③使能中断——ISERx，HAL_NVIC_EnableIRQ



- **特别提醒**：中断分组已经在我们的HAL_Init()中已经设置为分组为4，我们可以在**==hal.c==**文件里的HAL_Init()函数中查看，也就是说我们  有4位抢占优先级，0位响应优先级。也就是说我们只有16个抢占优先级分组（0-15）  

- 了解寄存器：

​		1，SCB_AIRCR

​		2，NVIC_IPRx

​		3，NVIC_ISER



下面将外设配置中经常用到的两个函数做个说明：

```c
 HAL_NVIC_SetPriority()

 HAL_NVIC_EnableIRQ()
```

首先，这两个函数定义在我们的**==crotex.c==**文件里，我们在Functions里面可以快速找到

​	1.**void HAL_NVIC_SetPriority(IRQn_Type IRQn, uint32_t PreemptPriority, uint32_t SubPriority)**

​	函数描述：

​	此函数主要用于设置中断的抢占优先级和子优先级。

​	①第一个参数是中断请求号，类型为枚举类型。每个中断请求号对应我们的中断向量表里的位置

​	②第二个参数是抢占优先级，根据分组来看可以设置哪些

​	③第三个参数为响应优先级，根据分组来看可以设置哪些

​	**注意：由于我们设置分组为4，所以只有（0-15）16个抢占优先级，即使设置了响应优先级也没有用！！！**



​	2.**void HAL_NVIC_EnableIRQ(IRQn_Type IRQn)**

​	函数描述：

​	此函数主要用于使能中断。

​	①只有一个参数，为我们的中断请求号



## 四.EXTI

#### 1.EXTI的基本概念

EXTI全名：**Ext**ernal(Extended)  **I**nterrupt/event Controller，外部(扩展)中断事件控制器

包含20个产生事件/中断请求的边沿检测器，即总共：20条EXTI线（F1）

**中断和事件的理解：**

中断：要进入NVIC，有相应的中断服务函数，需要CPU处理（NVIC相当于是管家）

事件：不进入NVIC，仅用于内部硬件自动控制的，如：TIM、DMA、ADC



**EXTI支持的外部中断/事件请求**

| **中断线**                     | **F1** | **F4** | **F7** | **H7**                        |
| ------------------------------ | ------ | ------ | ------ | ----------------------------- |
| EXTI线0~15：对应GPIO PIN 0~15  | ✔      | ✔      | ✔      | ✔                             |
| EXTI线16：PVD输出              | ✔      | ✔      | ✔      | 参考H7参考手册（中文版）657页 |
| EXTI线17：RTC闹钟事件          | ✔      | ✔      | ✔      | 参考H7参考手册（中文版）657页 |
| EXTI线18：USB OTG FS唤醒事件   | ✔      | ✔      | ✔      | 参考H7参考手册（中文版）657页 |
| EXTI线19：以太网唤醒事件       |        | ✔      | ✔      | 参考H7参考手册（中文版）657页 |
| EXTI线20：USB OTG HS唤醒事件   |        | ✔      | ✔      | 参考H7参考手册（中文版）657页 |
| EXTI线21：RTC 入侵和时间戳事件 |        | ✔      | ✔      | 参考H7参考手册（中文版）657页 |
| EXTI线22：RTC 唤醒事件         |        | ✔      | ✔      | 参考H7参考手册（中文版）657页 |
| EXTI线23：LPTIM1 异步事件      |        |        | ✔      | 参考H7参考手册（中文版）657页 |
| …                              |        |        |        |                               |

**对于中断我们现在的主线任务就是EXTI线0~15**

而我们EXTI线0~15是外部的，其余是内部的。



#### 2.EXTI的主要特性

`F1/F4/F7系列`

每条EXTI线都可以单独配置：选择类型（中断或者事件）、触发方式（上升沿，下降沿或者双边沿触发）、支持软件触发、开启/屏蔽、有挂起状态位



`H7系列`

由其它外设对 EXTI 产生的事件分为可配置事件和直接事件。

可配置事件：简单概括，基本和F1/F4/F7系列类似

直接事件：固定上升沿触发、不支持软件触发、无挂起状态位（由其它外设提供）





#### **3.EXTI工作原理**    （F1/F4/F7系列）

首先我们需要知道我们的逻辑运算符所对应的数字电路图形

![有“脑”的数字电路](https://ts1.cn.mm.bing.net/th/id/R-C.71efefd475cba5923eede183f1a26a34?rik=n5fah8mKTc59%2bg&riu=http%3a%2f%2frapheal-wordpress.stor.sinaapp.com%2fuploads%2f2015%2f10%2f7.%e5%85%b6%e4%bb%96%e9%80%bb%e8%be%91%e9%97%a8.png&ehk=c0tTZ%2bD3HdaTTFyR6fxm7A6OQyXstfX7%2bXUInTmNTf8%3d&risl=&pid=ImgRaw&r=0)



现在我们来看看工作原理图：

![EXTI工作原理](https://s2.loli.net/2023/11/11/F3yUs4VbJW52qmh.png)



①，边沿检测																											了解寄存器：

②，软件触发																											1，EXTI_FTSR

③，中断屏蔽/清除																								   2，EXTI_RTSR

④，事件屏蔽																											3，EXTI_IMR

​																																   4，EXTI_PR



对于我们需要了解的这四个寄存器我们只需要知道对应位，置1打开置0关闭就行了，我们查看STM32F10xxx中文数据参考手册就可以发现。



#### 4.EXTI与IO的映射关系

![AFIO](https://s2.loli.net/2023/11/11/O84VCie9M1FILBy.png)





![SYSCFG](https://s2.loli.net/2023/11/11/YU4smk3O8Ve2JMq.png)



![EXTI与IO口对应关系](https://s2.loli.net/2023/11/11/ZX9nkMz8phwI1m7.png)

也就是说EXTI1对应PA1，PB1······EXTI2对应PA2，PB2······



## 五.如何使用中断



![中断](https://s2.loli.net/2023/11/11/2kGjfov9dWzD4pe.png)

从上图我们可以了解到关于外部中断和外设中断的使用方法。

下面我们正式开始讲解如何使用外部中断，以下是设置步骤

![image-20231113185326795](https://s2.loli.net/2023/11/13/47YI6CXMafojRiu.png)



在进入中断服务函数后会

![image-20231113195541466](https://s2.loli.net/2023/11/13/MUj7IKnQ6Ry9eDd.png)





## 六.编程实战—外部中断点亮LED灯



#### 1.外部中断初始化

首先，外部中断，一定是来自外部输入，由此可见可以是我们的按键。所以我们如何初始化我们的GPIO结构体呢？

```c
void EXTI_Init(void)
{
    /* 定义 GPIO 初始化结构体 */
    GPIO_InitTypeDef GPIO_InitStructure = {0};
    /* 使能 GPIOA 时钟 */
    __HAL_RCC_GPIOA_CLK_ENABLE();
    /* 配置PA4，PA5两个IO口 */
    GPIO_InitStructure.Pin   = GPIO_PIN_4|GPIO_PIN_5;
    /* 设置工作模式 */
    GPIO_InitStructure.Mode  = GPIO_MODE_IT_FALLING;             //外部中断，下降沿触发检测
    /* 设置上下拉模式 */
    GPIO_InitStructure.Pull  = GPIO_NOPULL;                     //不上下拉
    /* 设置速度模式 */
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;            //低速模式
    /* 初始化 GPIO */
    HAL_GPIO_Init(GPIOA, &GPIO_InitStructure);
    HAL_NVIC_SetPriority(EXTI4_IRQn, 2, 0);
    HAL_NVIC_EnableIRQ(EXTI4_IRQn);
    HAL_NVIC_SetPriority(EXTI9_5_IRQn, 3, 0);
    HAL_NVIC_EnableIRQ(EXTI9_5_IRQn);
}
```

按照上面图的配置步骤，我们所使用的是PA4和PA5所对应的按键，所以

**第一步**是定义GPIO初始化结构体并使能GPIOA时钟

**第二步**配置PA4，PA5两个IO口

**第三步**设置工作模式，肯定是外部中断模式

```c
/* 外部中断，上升和下降双沿触发检测 */
#define GPIO_MODE_IT_RISING_FALLING (0x10310000u)
#define GPIO_MODE_IT_RISING (0x10120000U) 			/*外部事件，上升沿触发检测 */
#define GPIO_MODE_IT_FALLING (0x10220000U) 		/*外部事件，下降沿触发检测 */
```

通过第二节的学习我们可以知道按键按下是低电平，也就是下降沿触发

![image-20231109162232500](https://s2.loli.net/2023/11/09/lgjdtQhDopPMuFz.png)

**第四步**设置上下拉，上次课我们讲过是不上下拉。

**第五步**，初始化GPIO结构体。

到这里我们我们完成了一半。还有什么呢？

外部中断，首先我们应该设置中断优先级对吧，在上面我们讲过我们已经分组为4，所以只有抢占优先级而没有响应优先级，设置响应优先级是没有用的。所以我们只需要设置抢占优先级。又由于我们设置了两个按键，所以是两个外部中断，我们要分别设置两个按键的抢占优先级。

**第六步**设置中断优先级，我们讲过设置中断优先级是这个函数**HAL_NVIC_SetPriority()**,它有三个参数

第一个是中断请求号，按键PA4和PA5根据我们EXTI与IO的映射关系我们可以知道分别对应我们的**EXTI4_IRQn**和**EXTI9_5_IRQn**，我们要设置两次。后两个参数是抢占优先级和响应优先级。

**第七步**是使能中断，参数是中断请求号。



#### 2.设计中断服务函数

**第八步**是设计中断服务函数，我们知道我们的中断服务函数定义在**中断向量表**里，而中断向量表在**.s文件**里。

我们可以找到是**EXTI4_IRQHandler**和**EXTI9_5_IRQHandler**

在进入中断服务函数后会进入中断处理公共函数，由于我们讲过EXTI线0~15：对应GPIO PIN 0~15，所以我们的中断处理公共函数定义在**gpio.c文件**里

```c
void HAL_GPIO_EXTI_IRQHandler(uint16_t GPIO_Pin)
{
  /* EXTI line interrupt detected */
  if (__HAL_GPIO_EXTI_GET_IT(GPIO_Pin) != 0x00u)
  {
    __HAL_GPIO_EXTI_CLEAR_IT(GPIO_Pin);
    HAL_GPIO_EXTI_Callback(GPIO_Pin);
  }
}
```

它的参数为GPIO端口因为EXTI线0~15：对应GPIO PIN 0~15。

所以我们的中断处理公共函数写在我们的中断服务函数里

```c
void EXTI4_IRQHandler(void)
{
    HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_4);
}

void EXTI9_5_IRQHandler(void)
{
    HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_5);
}
```



#### 3.编写中断回调函数

**最后一步**是编写中断回调函数

依然是在**gpio.c文件**里我们可以发现中断回调函数

```c
__weak void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
  /* Prevent unused argument(s) compilation warning */
  UNUSED(GPIO_Pin);
  /* NOTE: This function Should not be modified, when the callback is needed,
           the HAL_GPIO_EXTI_Callback could be implemented in the user file
   */
}
```

**__weak**代表我们的中断回调函数是弱定义函数，可以由我们的用户编写这个函数。

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    //delay_ms(10);正常来说需要消抖，因为我们已经硬件消抖，所以不需要延时消抖。
    if(GPIO_Pin == GPIO_PIN_4)
    {
        if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0)
        {
            LED4_Turn;
        }
    
    }
    if(GPIO_Pin == GPIO_PIN_5)
    {
        if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_5) == 0)
        {
            LED3_Turn;
        }
    
    }
}
```

到这里我们的外部中断点亮LED灯就完成了。

如果不能理解我们的中断。我们可以这样想：

中断程序的处理和把大象放进冰箱类似，可分三步：

- 把冰箱门打开 —— 为中断处理函数的执行准备好环境；（初始化GPIO结构体，设置中断优先级，使能中断）
- 把大象放进去 —— 执行中断处理函数；（通过中断服务函数进入我们的中断处理公共函数）
- 把冰箱门关上 —— 退出中断处理函数，并从堆栈中恢复原有进程的状态。（执行中断回调函数，返回主程序）



#### 4.Keil里的全部代码

**==main.c==**

```c
#include "main.h"


int main()
{
    HAL_Init();
    LED_Init();
    delay_init();
    EXTI_Init();
    
    while(1)
    {
        LED2_ON;
    }
}

```



==**main.h**==

```c
#ifndef __MAIN_H
#define __MAIN_H


#include "stm32f1xx_hal.h"
#include "led.h"
#include "delay.h"
#include "sys.h"
#include "exti.h"

#endif

```



==**exti.c**==

```c
#include "exti.h"

void EXTI_Init(void)
{
    /* 定义 GPIO 初始化结构体 */
    GPIO_InitTypeDef GPIO_InitStructure = {0};
    /* 使能 GPIOA 时钟 */
    __HAL_RCC_GPIOA_CLK_ENABLE();
    /* 配置PA4，PA5两个IO口 */
    GPIO_InitStructure.Pin   = GPIO_PIN_4|GPIO_PIN_5;
    /* 设置工作模式 */
    GPIO_InitStructure.Mode  = GPIO_MODE_IT_FALLING;             //外部中断，下降沿触发检测
    /* 设置上下拉模式 */
    GPIO_InitStructure.Pull  = GPIO_NOPULL;                     //不上下拉
    /* 设置速度模式 */
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;            //低速模式,可以不设置
    /* 初始化 GPIO */
    HAL_GPIO_Init(GPIOA, &GPIO_InitStructure);
    HAL_NVIC_SetPriority(EXTI4_IRQn, 2, 0);
    HAL_NVIC_EnableIRQ(EXTI4_IRQn);
    HAL_NVIC_SetPriority(EXTI9_5_IRQn, 3, 0);
    HAL_NVIC_EnableIRQ(EXTI9_5_IRQn);
    
}

void EXTI4_IRQHandler(void)
{
    HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_4);
}

void EXTI9_5_IRQHandler(void)
{
    HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_5);
}

void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    //delay_ms(10);			//正常来说需要消抖，因为我们已经硬件消抖，所以不需要延时消抖。
    if(GPIO_Pin == GPIO_PIN_4)
    {
        if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0)
        {
            LED4_Turn;
        }
    
    }
    if(GPIO_Pin == GPIO_PIN_5)
    {
        if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_5) == 0)
        {
            LED3_Turn;
        }
    
    }
}

```



==**exti.h**==

```c
#ifndef __EXTI_H
#define __EXTI_H

#include "stm32f1xx_hal.h"
#include "delay.h"
#include "led.h"

void EXTI_Init(void);

#endif

```



==**led.c**==

```c
#include "led.h"

void LED_Init(void) {
    /* 定义 GPIO 初始化结构体 */
    GPIO_InitTypeDef GPIO_InitStructure = {0};
    /* 使能 GPIOA 时钟 */
    __HAL_RCC_GPIOA_CLK_ENABLE();
    /* 配置PA0，PA1，PA2三个IO口 */
    GPIO_InitStructure.Pin   = GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2 | GPIO_PIN_3;
    /* 设置工作模式 */
    GPIO_InitStructure.Mode  = GPIO_MODE_OUTPUT_PP;         //推挽输出模式
    /* 设置上下拉模式 */
    GPIO_InitStructure.Pull  = GPIO_NOPULL;                 //既不上拉也不下拉
    /* 设置速度模式 */
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;         //低速模式
    /* 初始化 GPIO */
    HAL_GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    /* 设置输出状态 */
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET);     //给PA0设置高电平
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET);     //给PA1设置高电平
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, GPIO_PIN_SET);     //给PA2设置高电平
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_SET);     //给PA3设置高电平
}

```



==**led.h**==

```c
#ifndef __LED_H
#define __LED_H

#include "stm32f1xx_hal.h"

#define LED1_ON HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET)
#define LED2_ON HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_RESET)
#define LED3_ON HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, GPIO_PIN_RESET)
#define LED4_ON HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_RESET)
#define LED_ALL_ON HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, GPIO_PIN_RESET)

#define LED1_OFF HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET)
#define LED2_OFF HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET)
#define LED3_OFF HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, GPIO_PIN_SET)
#define LED4_OFF HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_SET)
#define LED_ALL_OFF HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, GPIO_PIN_SET)

#define LED1_Turn HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_0)
#define LED2_Turn HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_1)
#define LED3_Turn HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_2)
#define LED4_Turn HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_3)
#define LED_ALL_Turn HAL_GPIO_HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3)

#define LED1_Read HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0)
#define LED2_Read HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_1)
#define LED3_Read HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2)
#define LED4_Read HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_3)
#define LED_ALL_Read HAL_GPIO_HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3)

void LED_Init(void);

#endif

```

---

## 相关笔记

**STM32 HAL 库学习路径**

- [[STM32先导]]
- [[STM32最小系统]]
- [[STM32文件讲解与使用方法]]
- [[STM32-GPIO_1]]
- [[STM32-GPIO_2]]
- [[STM32-TIMER]]
- [[STM32-USART]]
- [[STM32-IIC]]
- [[TB6612]]
- [[STM32标准库学习记录]]
- [[PID算法学习记录]]
- [[stm32与openmv串口通信]]

