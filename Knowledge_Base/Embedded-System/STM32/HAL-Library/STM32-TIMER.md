---
title: STM32-TIMER
date: 2026-08-01
tags: [Embedded, STM32, HAL]
aliases: []
---

# ==STM32第六次培训—TIMER==



## 一.定时器概述

### 1.软件定时原理

使用纯软件（CPU死等）的方式实现定时（延时）功能

```c
void delay_us(uint32_t us)
{
    us *= 72;
    while(us--);
}

```

缺点：

1，延时不精准

2，CPU死等，必须延时完才能进行其他操作



### 2.定时器定时原理

使用精准的时基，通过硬件的方式，实现定时功能

定时器核心就是计数器

![image-20231205103842136](https://s2.loli.net/2023/12/05/1pRlJnSdfGcCXM9.png)



### 3.STM32定时器的分类

![image-20231205104205705](https://s2.loli.net/2023/12/05/V4uOQrLPvC2Debt.png)

目前我们所培训的是标红部分



### 4.STM32F1系列定时器特性表

![芯片手册](https://s2.loli.net/2023/11/16/JYrS6sPnGDKHgtd.png)



![image-20231205105610107](https://s2.loli.net/2023/12/05/obP6eY9Ir4Q1fwc.png)

| 定时器类型 | 主要功能                                                     |
| ---------- | ------------------------------------------------------------ |
| 基本定时器 | 没有输入输出通道，常用作时基，即定时功能                     |
| 通用定时器 | 具有多路独立通道，可用于输入捕获/输出比较，也可用作时基      |
| 高级定时器 | 除具备通用定时器所有功能外，还具备带死区控制的互补信号输出、刹车输入等功能（可用于电机控制、数字电源设计等） |



![时钟树.png](https://s2.loli.net/2023/10/24/oJzeZFpy1txCUTc.png)



对于我们的F1系列的定时器，我们可以发现定时器1和8挂载在APB2总线上，定时器2-7挂载在APB1总线上，同时，当APB2预分频系数为1时频率为72MHz，否则乘2；当APB1预分频系数为1时频率为36MHz，否则乘2



![image-20231205220543818](https://s2.loli.net/2023/12/05/UZajs9odWbrtwpg.png)



对于我们这款STM32F103C8T6芯片，我们选择APB1预分频系数为2，APB2预分频系数为1。所以我们所有的定时器都为72MHz。



## 二.基本定时器（F1）

#### 1.基本定时器知识回顾

##### ①.基本定时器分为：

TIM6/TIM7



##### ②.主要特性：

16位递增计数器（计数值：0~65535）

16位预分频器（分频系数：1~65536）

可用于触发DAC

在更新事件（计数器溢出）时，会产生中断/DMA请求



##### ③.总线：

挂载在APB1总线上



#### 2.基本定时器框图

![image-20231205211007323](https://s2.loli.net/2023/12/05/BecWGCKpDZg9usX.png)

##### ①时钟源：  

首先，对于我们的TIM6和TIM7基本定时器，它们的时钟源只能来自内部时钟，也就是来自外设总线APB提供的时钟。



##### ②.控制器：

其次，我们的控制器是用来控制计数器的复位，使能和计数，而触发控制器是当计数器溢出时产生TRGO信号然后产生一次DAC数模转换。



##### ③.计数器（时基单元）：

最后，对于我们的计数器，首先会对来自内部时钟的时钟源通过预分频器（1~65535）进行分频得到计数器的工作频率，而我们计数器的计数模式为递增，每来一个这样的时钟，我们的计数器的值就加一，当我们计数器的值（CNT）等于自动重装载寄存器的值（ARR）就会溢出。此时，便会产生我们的事件（默认产生），中断和DMA输出（默认不产生，可以由我们用户自己配置）。同时，我们的UG位也可以产生软件更新事件。产生了更新事件后便会将自动重装载寄存器和预分频器的值加载到对应的影子寄存器里，所以这两个寄存器实际起到缓冲的作用。



注意：

①.对于我们的预分频器和自动重装载寄存器，它们有各自的影子寄存器，而我们的影子寄存器是实际起作用的寄存器，不可直接访问。

②.自动重装载寄存器的ARPE位可决定ARR是否具有缓冲作用。

**③.我们设置预分频系数时默认加一。**



#### 3.STM32定时器计数模式及溢出条件

| 计数器模式                     | 溢出条件               |
| ------------------------------ | ---------------------- |
| 递增计数模式  （向上计数模式） | CNT==ARR               |
| 递减计数模式  （向下计数模式） | CNT==0                 |
| 中心对齐模式                   | CNT`==`ARR-1、CNT`==`1 |

![image-20231205221000194](https://s2.loli.net/2023/12/05/yvRwHelYSOzEXZM.png)



```c
CK_PSC：定时器的时钟源（未经过分频）

CNT_EN：计数器使能位，当使能为1时，计数器开始计数

CK_CNT：定时器的计数时钟（经过分频）
```



##### ①.递增计数模式实例说明

```c
PSC = 1		//预分频系数为2
ARR = 36
```

![image-20231205221324585](https://s2.loli.net/2023/12/05/1HDvzGIf5Rx62ny.png)

由于预分频系数为2，所以每来两个时钟源才会用一个时钟信号

由于自动重装载值为36，所以当计数器向上计数到36时，计数器上溢出，此时产生更新事件并且更新中断标志置1



##### ②.递减计数模式实例说明

```c
PSC = 1		//预分频系数为2
ARR = 36
```

![image-20231205221533144](https://s2.loli.net/2023/12/05/WyUHAaxOuRkq4Z5.png)

由于预分频系数为2，所以每来两个时钟源才会用一个时钟信号

由于自动重装载值为36，所以当计数器向下计数到0时，计数器下溢出，此时产生更新事件并且更新中断标志置1

   

##### ③.中心对齐模式实例说明

```c
PSC = 0		//预分频系数为1
ARR = 6
```

![image-20231205221646609](https://s2.loli.net/2023/12/05/ihJTrFxZwgy2o6W.png)

由于预分频系数为1，所以每来一个时钟源才会用一个时钟信号

由于自动重装载值为6，所以当计数器向下计数到1或计数器向上计数到5时，此时产生更新事件并且更新中断标志置1



#### 4.定时器中断实验相关寄存器

##### ①.`TIM6 和TIM7控制寄存器1(TIMx_CR1)`

![image-20231206125146914](https://s2.loli.net/2023/12/06/Kz2AjdchDoxywXf.png)

用于设置ARR寄存器是否具有缓冲，使能/关闭计数器

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，15.4.1节`



##### ②.`TIM6和TIM7 DMA/中断使能寄存器(TIMx_DIER)`

![image-20231206125316707](https://s2.loli.net/2023/12/06/p74Pnie9y8uhg2W.png)

用于使能更新中断

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，15.4.3节`



##### ③.`TIM6和TIM7状态寄存器(TIMx_SR)	`

![image-20231206125443006](https://s2.loli.net/2023/12/06/8sPzhc2H9mlAu4G.png)

用于判断是否发生了更新中断，由硬件置1，软件清零

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，15.4.4节`



##### ④.`TIM6和TIM7计数器(TIMx_CNT)`

![image-20231206125614944](https://s2.loli.net/2023/12/06/WzQMK9cp6wIi3GT.png)

计数器实时数值，可用于设置计时器初始值，范围：0~65535

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，15.4.6节`



##### ⑤.` TIM6和TIM7预分频器(TIMx_PSC)`

![image-20231206125738107](https://s2.loli.net/2023/12/06/YJPRte5bFd1qXfr.png)

用于设置预分频系数，范围：0~65535，实际预分频系数等于PSC+1

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，15.4.7节`



##### ⑥. `TIM6和TIM7自动重装载寄存器(TIMx_ARR)`

![image-20231206125923531](https://s2.loli.net/2023/12/06/DOru8jW9QRJ2lh3.png)

用于设置自动重装载值，范围：0~65535

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，15.4.8节`



#### 5.定时器溢出时间计算方法

$$
T_{out} = \frac{(ARR+1)×(PSC+1)}{{F_{t}}}
$$

```cpp
Tout是定时器溢出时间

Ft是定时器的时钟源频率

ARR是自动重装载寄存器的值

PSC是预分频器寄存器的值
```



#### 6.定时器中断实验配置步骤

![image-20231206130513770](https://s2.loli.net/2023/12/06/ncSmXj7kxusPywp.png)



##### ①.相关函数简介

| 函数                            | 主要寄存器    | 主要功能                             |
| ------------------------------- | ------------- | ------------------------------------ |
| HAL_TIM_Base_Init()             | CR1、ARR、PSC | 初始化定时器基础参数                 |
| HAL_TIM_Base_MspInit()          | 无            | 存放NVIC、CLOCK、GPIO初始化代码      |
| HAL_TIM_Base_Start_IT()         | DIER、CR1     | 使能更新中断并启动计数器             |
| HAL_TIM_IRQHandler()            | SR            | 定时器中断处理公用函数，处理各种中断 |
| HAL_TIM_PeriodElapsedCallback() | 无            | 定时器更新中断回调函数，由用户重定义 |



##### ②.关键结构体介绍

```c
typedef struct 
{ 
    TIM_TypeDef *Instance;          /* 外设寄存器基地址 */ 
    TIM_Base_InitTypeDef Init;     	/* 定时器初始化结构体*/
     ...							/* 后面的我们不做了解，因为用不上 */
}TIM_HandleTypeDef;
```



```c
typedef struct 
{ 
    uint32_t Prescaler;               /* 预分频系数 */ 
    uint32_t CounterMode;             /* 计数模式 */ 
    uint32_t Period;                  /* 自动重载值 ARR */ 
    uint32_t ClockDivision;           /* 时钟分频因子 */ 
    uint32_t RepetitionCounter;   	  /* 重复计数器寄存器的值 */ 
    uint32_t AutoReloadPreload; 	  /* 自动重载预装载使能 */
} TIM_Base_InitTypeDef;
```



#### 7.基本定时器的使用

首先需要配置好我们的驱动文件

![image-20231206134301905](https://s2.loli.net/2023/12/06/52zXrcTylwjHW4G.png)



**==timer.c==**

```c
#include "led.h"
#include "timer.h"


TIM_HandleTypeDef g_timx_handle;

/* 定时器中断初始化函数 */
void Timer_Int_init(uint16_t arr, uint16_t psc)
{
    g_timx_handle.Instance = TIM6;
    g_timx_handle.Init.Prescaler = psc;
    g_timx_handle.Init.Period = arr;
    HAL_TIM_Base_Init(&g_timx_handle);//定时器初始化

    HAL_TIM_Base_Start_IT(&g_timx_handle);//使能定时器中断
}

/* 定时器基础MSP初始化函数 */
void HAL_TIM_Base_MspInit(TIM_HandleTypeDef *htim)
{
    if (htim->Instance == TIM6)
    {
        __HAL_RCC_TIM6_CLK_ENABLE();
        HAL_NVIC_SetPriority(TIM6_IRQn, 1, 3);
        HAL_NVIC_EnableIRQ(TIM6_IRQn);
    }
}

/* 定时器6中断服务函数 */
void TIM6_IRQHandler(void)
{
    HAL_TIM_IRQHandler(&g_timx_handle);
}

/* 定时器溢出中断中断回调函数 */
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if (htim->Instance == TIM6)
    {
        LED0_TOGGLE();
    }
}

```



**==timer.h==**

```c
#ifndef __TIMER_H

#define __TIMER_H

#include "main.h"

extern TIM_HandleTypeDef g_timx_handle;

void Timer_Int_Init(uint16_t arr, uint16_t psc);

#endif

```



**==main.c==**

```c
#include "main.h"

int main(void)
{
    HAL_Init();
    Stm32_Clock_Init();                         /* 配置时钟树72MHz */
    LED_Init();
    delay_init();
    Timer_Int_Init(4999, 7199);
    while(1)
    {
        
    }
}

```



**==main.h==**

```C
#ifndef __MAIN_H
#define __MAIN_H


#include "stm32f1xx_hal.h"
#include "led.h"
#include "delay.h"
#include "sys.h"
#include "timer.h"

#endif

```



由于我们STM32F103C8T6芯片没有基本定时器TIM6和TIM7，所以我们选择通用定时器TIM2~4任意一个即可。



### 三.通用定时器（F1）

#### 1.通用定时器知识回顾

##### ①.通用定时器分为：

TIM2/TIM3 /TIM4 /TIM5



##### ②.主要特性：

16位递增、递减、中心对齐计数器（计数值：0~65535）

16位预分频器（分频系数：1~65536）

可用于触发DAC、ADC

在更新事件、触发事件、输入捕获、输出比较时，会产生中断/DMA请求

4个独立通道，可用于：输入捕获、输出比较、输出PWM、单脉冲模式

使用外部信号控制定时器且可实现多个定时器互连（级联）的同步电路

支持编码器和霍尔传感器电路等



##### ③.总线：

挂载在APB1总线上



#### 2.通用定时器框图

![image-20231206161202171](https://s2.loli.net/2023/12/06/JfS96718FxNAr2u.png)

##### ①.时钟源

基本定时器的时钟源只能来自内部时钟，而我们通用定时器的时钟源有四类

![image-20231206164202784](https://s2.loli.net/2023/12/06/oT7PAyrb39EX84n.png)

```basic
①内部时钟(CK_INT):来自外设总线APB提供的时钟
    
②外部时钟模式1:源于外部输入引脚(TIx),来自定时器通道1或者通道2引脚的信号,例如霍尔传感电路检测电机转速
    
③外部时钟模式2:源于外部触发输入(ETR),来自可以复用为TIMx_ETR的IO引脚
    
④内部触发输入(ITRx):源于一个定时器的内部输出（TRGO）到ITRx的输入时钟信号（x:ITR0~3）,用于与芯片内部其它通用/高级定时器级联
```



| 计数器时钟选择类型               | 设置方法                                                   |
| -------------------------------- | ---------------------------------------------------------- |
| 内部时钟(CK_INT)                 | 设置TIMx_SMCR的SMS=000                                     |
| 外部时钟模式1：外部输入引脚(TIx) | 设置TIMx_SMCR的SMS=111                                     |
| 外部时钟模式2：外部触发输入(ETR) | 设置TIMx_SMCR的ECE=1                                       |
| 内部触发输入(ITRx)               | 设置可参考STM32F10xxx参考手册_V10（中文版）.pdf，14.3.15节 |



**外部时钟模式1：**

![image-20231207193339421](https://s2.loli.net/2023/12/07/I3aNw524PiMlrSb.png)



`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，14.3.3节`

从图上我们可以发现，从模式选择外部时钟模式1（ECE设置为0，SMS[2:0]设置为111），同时，我们以通道2单边沿（设置TIMx_SMCR的TS[2:0]为110）输入为例，当信号经过滤波器（具体查看IC2F[3:0]的配置情况）后，进行边缘检测（设置TIMx_CCRE的CC2P为0代表来的是上升沿不用反向，为1代表来的是下降沿需要反向，因为触发输入信号TRGI为上升沿触发），当触发输入上升沿信号来临时，计数器计数一次。



**外部时钟模式2：**

![image-20231207211827744](https://s2.loli.net/2023/12/07/3UHu6TCYV7ISptG.png)

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，14.3.3节`

关于外部时钟模式2我们须要注意的是信号来自我们的IO口复用为ETR引脚输入，然后经过ETR判断外部触发输入信号是上升沿还是下降沿，上升沿寄存器置0，反之置1将下降沿反向（因为触发输入信号ETRF为上升沿触发），其次经过分频器，最后经过滤波后，当触发输入上升沿信号来临时，计数器计数一次。



**内部触发输入（F1）：**

![image-20231207213120532](https://s2.loli.net/2023/12/07/dilzuhJpBQTobFA.png)

![image-20231207213214901](https://s2.loli.net/2023/12/07/D56Bd3uvoSf4F9V.png)



##### ②.控制器

通用定时器的从模式控制器是用来控制计数器的复位，使能，递增和计数，而触发控制器是当计数器溢出时产生TRGO信号然后产生一次DAC数模或ADC模数转换，同时TRGO触发输出信号可以作为内部触发输入信号，用作级联。

编码器接口可接收增量（正交）编码器的信号，根据编码器旋转产生的正交信号脉冲，`自动控制CNT自增或自减`，从而指示编码器的位置、旋转方向和旋转速度。



##### ③.时基单元

参考基本定时器



##### ④.输入捕获

输入捕获可以对输入的信号的上升沿，下降沿或者双边沿进行捕获，通常用于测量输入信号的脉宽、测量 PWM 输入信号的频率及占空比。



##### ⑤.捕获/比较（公共）



##### ⑥.输出比较

输出比较可以通过比较**CNT**（时基单元内的计数器）与**CCR**（捕获比较寄存器）值的关系，来对输出电平进行置1、置0或翻转的操作，用于输出一定频率和占空比的[PWM波](https://so.csdn.net/so/search?q=PWM波&spm=1001.2101.3001.7020)形（用于输出PWM波形）。



#### 3.通用定时器输出比较部分框图介绍

![image-20231207223214913](https://s2.loli.net/2023/12/07/GSFYOpqNkTwIZy9.png)

##### ①.捕获/比较通道1的主电路---输出部分

![image-20231207223510815](https://s2.loli.net/2023/12/07/zWrFDp2HTAUG83L.png)

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，14.3.4节`



##### ②.捕获/比较通道的输出部分（通道1）

![image-20231207224033236](https://s2.loli.net/2023/12/07/LQ89j71qgm3SUzE.png)

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，14.3.4节`



以下为字典部分：

```c
CC1S[1:0]：捕获/比较1 选择 (Capture/Compare 1 selection)
这2位定义通道的方向(输入/输出)，及输入脚的选择：
00：CC1通道被配置为输出；
01：CC1通道被配置为输入，IC1映射在TI1上；
10：CC1通道被配置为输入，IC1映射在TI2上；
11：CC1通道被配置为输入，IC1映射在TRC上。此模式仅工作在内部触发器输入被选中时(由
TIMx_SMCR寄存器的TS位选择)。
注：CC1S仅在通道关闭时(TIMx_CCER寄存器的CC1E=’0’)才是可写的。
```



```c
OC1M[2:0]：输出比较1模式 (Output compare 1 enable)
该3位定义了输出参考信号OC1REF的动作，而OC1REF决定了OC1的值。OC1REF是高电平
有效，而OC1的有效电平取决于CC1P位。
000：冻结。输出比较寄存器TIMx_CCR1与计数器TIMx_CNT间的比较对OC1REF不起作用；
001 ：匹配时设置通道 1 为有效电平。当计数器 TIMx_CNT 的值与捕获 / 比较寄存器 1
(TIMx_CCR1)相同时，强制OC1REF为高。
010 ：匹配时设置通道 1 为无效电平。当计数器 TIMx_CNT 的值与捕获 / 比较寄存器 1
(TIMx_CCR1)相同时，强制OC1REF为低。
011：翻转。当TIMx_CCR1=TIMx_CNT时，翻转OC1REF的电平。
100：强制为无效电平。强制OC1REF为低。
101：强制为有效电平。强制OC1REF为高。
110：PWM模式1－ 在向上计数时，一旦TIMx_CNT<TIMx_CCR1时通道1为有效电平，否则为
无效电平；在向下计数时，一旦TIMx_CNT>TIMx_CCR1时通道1为无效电平(OC1REF=0)，否
则为有效电平(OC1REF=1)。
111：PWM模式2－ 在向上计数时，一旦TIMx_CNT<TIMx_CCR1时通道1为无效电平，否则为
有效电平；在向下计数时，一旦TIMx_CNT>TIMx_CCR1时通道1为有效电平，否则为无效电
平。
注1：一旦LOCK级别设为3(TIMx_BDTR寄存器中的LOCK位)并且CC1S=’00’(该通道配置成输
出)则该位不能被修改。
注2：在PWM模式1或PWM模式2中，只有当比较结果改变了或在输出比较模式中从冻结模式
切换到PWM模式时，OC1REF电平才改变。
```



```c
 OC1CE：输出比较1清0使能 (Output compare 1 clear enable)
0：OC1REF 不受ETRF输入的影响；
1：一旦检测到ETRF输入高电平，清除OC1REF=0。
```



```c
CC1P：输入/捕获1输出极性 (Capture/Compare 1 output polarity)
CC1通道配置为输出：
0：OC1高电平有效
1：OC1低电平有效
CC1通道配置为输入：
该位选择是IC1还是IC1的反相信号作为触发或捕获信号。
0：不反相：捕获发生在IC1的上升沿；当用作外部触发器时，IC1不反相。
1：反相：捕获发生在IC1的下降沿；当用作外部触发器时，IC1反相。
```



```c
CC1E：输入/捕获1输出使能 (Capture/Compare 1 output enable)
CC1通道配置为输出：
0： 关闭－ OC1禁止输出。
1： 开启－ OC1信号输出到对应的输出引脚。
CC1通道配置为输入：
该位决定了计数器的值是否能捕获入TIMx_CCR1寄存器。
0：捕获禁止；
0：捕获使能。
```





#### 4.PWM

[什么是PWM？](https://www.bilibili.com/video/BV1HD4y1k74L/?spm_id_from=333.337.search-card.all.click&vd_source=2daccc2d29c21699d84b67d8045701fe)

##### ①.通用定时器输出PWM原理

```php
假设：递增计数模式

ARR：自动重装载寄存器的值

CCRx：捕获/比较寄存器x的值
```



```tex
当CNT < CCRx，IO输出0

当CNT >= CCRx，IO输出1
```



![image-20231207225322435](https://s2.loli.net/2023/12/07/smUfv4TkdFnc8Be.png)

`总结：PWM波周期或频率由ARR决定，PWM波占空比由CCRx决定`



##### ②.PWM模式

![image-20231207230208648](https://s2.loli.net/2023/12/07/LYfcuCgEKBNxi6q.png)





##### ③.通用定时器PWM输出实验配置步骤

![image-20231207230426284](https://s2.loli.net/2023/12/07/85qh2XfEcbkziRj.png)



###### 1).相关函数简介

| 函数                          | 主要寄存器        | 主要功能                        |
| ----------------------------- | ----------------- | ------------------------------- |
| HAL_TIM_PWM_Init()            | CR1、ARR、PSC     | 初始化定时器基础参数            |
| HAL_TIM_PWM_MspInit()         | 无                | 存放NVIC、CLOCK、GPIO初始化代码 |
| HAL_TIM_PWM_ConfigChannel()   | CCMRx、CCRx、CCER | 配置PWM模式、比较值、输出极性等 |
| HAL_TIM_PWM_Start()           | CCER、CR1         | 使能输出比较并启动计数器        |
| __HAL_TIM_SET_COMPARE()       | CCRx              | 修改比较值                      |
| __HAL_TIM_ENABLE_OCxPRELOAD() | CCER              | 使能通道预装载                  |



###### 2).关键结构体介绍

```c
typedef struct 
{ 
    uint32_t OCMode; 	  		/* 输出比较模式选择 */
    uint32_t Pulse; 	        /* 设置比较值 */
    uint32_t OCPolarity;       	/* 设置输出比较极性 */
    uint32_t OCNPolarity;    	/* 设置互补输出比较极性 */
    uint32_t OCFastMode;   		/* 使能或失能输出比较快速模式 */
    uint32_t OCIdleState;     	/* 空闲状态下OC1输出 */
    uint32_t OCNIdleState;  	/* 空闲状态下OC1N输出 */ 
} TIM_OC_InitTypeDef;
```



```c
typedef struct 
{ 
    uint32_t OCMode; 	  		 /* 输出比较模式选择 */
    uint32_t Pulse; 	         /* 设置比较值 */
    uint32_t OCPolarity;       	 /* 设置输出比较极性 */
	......(高级定时器才会用到)
} TIM_OC_InitTypeDef;
```



##### ④.编程实战：呼吸灯

通过定时器输出的PWM控制LED2，实现类似手机呼吸灯的效果



###### 1)确定PWM波的周期/频率

$$
T_{out} = \frac{(ARR+1)*(PSC+1)}{{F_{t}}}
$$

$$
F_{pwm} = \frac{1}{{T_{out}}}
$$



`50Hz为例，PSC=1439，ARR=999`



###### 2)配置输出比较模式为：

PWM模式1，通道输出极性为：低电平有效

![QQ图片20231209163751](https://s2.loli.net/2023/12/09/3I2UYlk1a9u8HmT.jpg)



###### 3)PWM初始化

![image-20231206134301905](https://s2.loli.net/2023/12/06/52zXrcTylwjHW4G.png)

驱动文件同上

**==pwm.c==**

```c
TIM_HandleTypeDef htimer2_pwm;

/* 通用定时器PWM输出初始化函数 */
void TIMER2_PWM_Int_Init(uint32_t ARR, uint32_t PSC)
{
    TIM_OC_InitTypeDef pwm_oc_chy ={0};
    
    htimer2_pwm.Instance = TIM2;                        /* 通用定时器2 */
    htimer2_pwm.Init.Period = ARR;                      /* 自动重装载值ARR */
    htimer2_pwm.Init.Prescaler = PSC;                   /* 预分频系数PSC */
    htimer2_pwm.Init.CounterMode = TIM_COUNTERMODE_UP;  /* 向上计数模式 */
    
    HAL_TIM_PWM_Init(&htimer2_pwm);                     /* 通用定时器2的PWM初始化 */
    
    pwm_oc_chy.OCMode = TIM_OCMODE_PWM1;                /* PWM模式1 */
    //pwm_oc_chy.Pulse = ARR / 2;                         /* 设置比较值CCRx */
    pwm_oc_chy.OCPolarity = TIM_OCPOLARITY_LOW;         /* 低电平有效 */
    
    HAL_TIM_PWM_ConfigChannel(&htimer2_pwm, &pwm_oc_chy, TIM_CHANNEL_2);/* 通用定时器2的PWM模式及通道初始化 */
    HAL_TIM_PWM_Start(&htimer2_pwm, TIM_CHANNEL_2);     /* 使能输出并启动计数器 */
    
}

/* 通用定时器输出PWM MSP初始化函数 */
void HAL_TIM_PWM_MspInit(TIM_HandleTypeDef *htim)
{
    if(htim->Instance == TIM2)
    {
        GPIO_InitTypeDef led;
        __HAL_RCC_GPIOA_CLK_ENABLE();           /* LED2端口使能 */
        __HAL_RCC_TIM2_CLK_ENABLE();            /* 定时器2使能 */

        led.Pin = GPIO_PIN_1;                   /* PA1端口的LED灯 */
        led.Mode = GPIO_MODE_AF_PP;             /* 推挽复用输出 */
        led.Pull = GPIO_NOPULL;                 /* 不上下拉 */
        led.Speed = GPIO_SPEED_FREQ_HIGH;       /* 高速 */
        
        HAL_GPIO_Init(GPIOA, &led);             /* GPIO端口初始化 */
        
    }
}

```



**==pwm.h==**

```c
#ifndef __PWM_H

#define __PWM_H

#include "main.h"

extern TIM_HandleTypeDef htimer2_pwm;

void TIMER2_PWM_Int_Init(uint32_t ARR, uint32_t PSC);

void BreatheLED(uint32_t delay_ms, int steps);

#endif

```



###### 4)主函数的编写

**==main.c==**

```c
int main(void)
{
    HAL_Init();
    Stm32_Clock_Init();
    delay_init();
    TIMER2_PWM_Int_Init(999, 1439);         /* 50Hz-0.02s(代表一秒变化50次)(20ms一个周期) */
    uint16_t ledpwmval = 0;                 /* 设置比较值 */
    uint8_t dir = 1;                        /* 设置比较值方向 */
    
    while(1)
    {
        delay_ms(1);                        /* PWM每多少ms增加或减少一次 */

        if(dir)
            ledpwmval++;                    /* dir==1 ledpwmval递增LED变亮 */
        else
            ledpwmval--;                    /* dir==0 ledpwmval递减LED变灭 */

        if (ledpwmval > 500)                /* LED最大/最低亮度 */
            dir = 0;                        /* ledrpwmval到达500后，方向为递减 */
        if (ledpwmval == 0)
            dir = 1;                        /* ledrpwmval递减到0后，方向改为递增 */

        /* 修改比较值控制占空比 */
        __HAL_TIM_SET_COMPARE(&htimer2_pwm, TIM_CHANNEL_2, ledpwmval);  /* HAL库函数法 */
        //TIM2->CCR2 = ledpwmval;                                         /* 寄存器法 */
    }
}

```



**==main.h==**

```c
#ifndef __MAIN_H

#define __MAIN_H

#include "stm32f1xx_hal.h"
#include "delay.h"
#include "sys.h"
#include "pwm.h"


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
- [[STM32-EXTI]]
- [[STM32-USART]]
- [[STM32-IIC]]
- [[TB6612]]
- [[STM32标准库学习记录]]
- [[PID算法学习记录]]
- [[stm32与openmv串口通信]]

