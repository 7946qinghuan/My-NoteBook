---
title: STM32-GPIO_1
date: 2026-08-01
tags: [Embedded, STM32, HAL]
aliases: []
---

# ==STM32第二次培训—GPIO==



[TOC]



## 一.什么是GPIO？

我们STM32学习的第一个外设就是GPIO，那么什么是GPIO呢？

**General-purpose input/output**

通用型之输入输出的简称



**作用：**负责采集外部器件的信息或者控制外部器件工作，即输入输出

常见输入就是按键，常见输出就是LED灯



**特点：**

`1.可配置为8种输入输出模式。`

输出模式：可控制端口输出高低电平，用以驱动LED、控制蜂鸣器、模拟通信协议输出时序等 。

输入模式：可读取端口的高低电平或电压，用于读取按键输入、外接模块电平信号输入、ADC电压采集、模拟通信协议接收数据等。

`
2.引脚电平：0V~3.3V(工作电压2V~3.6V),部分引脚可容忍5V。`

`3.每个IO口都可以做中断。`

`4.单个IO口最大电流25mA。`



**GPIO的八种模式：**

| 模式名称     | 性质     | 特征                                               |
| ------------ | -------- | -------------------------------------------------- |
| 浮空输入     | 数字输入 | 可读取引脚电平。若引脚悬空。则电平不稳             |
| 上拉输入     | 数字输入 | 可读取引脚电平，内部连接上拉电阻，悬空时默认高电平 |
| 下拉输入     | 数字输入 | 可读取引脚电平，内部连接下拉电阻，悬空时默认低电平 |
| 模拟输入     | 模拟输入 | GPIO无效，引脚直接接入内部ADC                      |
| 开漏输出     | 数字输出 | 可输出引脚电平，高电平为高阻态，低电平接VSS        |
| 推挽输出     | 数字输出 | 可输出引脚电平，高电平接VDD，低电平接VSS           |
| 复用开漏输出 | 数字输出 | 由片上外设控制，高电平为高阻态，低电平接VSS        |
| 复用推挽输出 | 数字输出 | 由片上外设控制，高电平接VDD，低电平接VSS           |

详解：（[STM32学习—GPIO的八种工作模式_gpio八种工作模式_Renhui₋的博客-CSDN博客](https://blog.csdn.net/weixin_46245859/article/details/132426484)）

通用输出为开漏输出和推挽输出：

**开漏输出：**

- **当输出寄存器输出高电平，则引脚输出高阻态(电路分析时高阻态可做开路理解)**

- **当输出寄存器输出低电平，则引脚输出低电平**

**推挽输出:**

- **当输出寄存器输出高电平，则引脚也输出高电平**
- **当输出寄存器输出低电平，则引脚也输出低电平**







## 二.通用外设驱动模型（四步法）

**①初始化**

包括：**时钟设置，参数设置**，`IO设置，中断设置（开中断，设NVIC）【可选】`

**②读函数（可选）**

从外设读取数据（可选）

**③写函数（可选）**

往外设写入数据（可选）

**④中断服务函数（可选）**

根据中断标志，处理外设各种中断事务（可选）



## 三.GPIO的配置步骤

1.使能时钟

```c
__HAL_RCC_GPIOx_CLK_ENABLE()
```

其中x代表A，B，C等，就是我们芯片上引脚的字母编号



2.设置工作模式

```
__HAL_GPIO_Init()
```



3.设置输出状态（可选）

```
HAL_GPIO_WritePin()
```

给对应的引脚写一个状态，高电平或者低电平

```
HAL_GPIO_TogglePin()
```

让对应的引脚翻转状态



4.读取输入状态（可选）

```
HAL_GPIO_ReadPin()
```



![image-20231031102652661](https://s2.loli.net/2023/10/31/Q6WbZVSLftxqTDj.png)



## 四.关键结构体简介

![image-20231031103140082](https://s2.loli.net/2023/10/31/WPRQSBV4nOip8Eu.png)

```c
typedef struct
{
 uint32_t Pin; 		/* 引脚号 */
 uint32_t Mode; 	/* 模式设置 */
 uint32_t Pull; 	/* 上拉下拉设置 */
 uint32_t Speed; 	/* 速度设置 */
} GPIO_InitTypeDef;
```



**①Pin：**

![image-20231031103319642](https://s2.loli.net/2023/10/31/uAYgfLexCayvWtk.png)

**成员 Pin 表示引脚号，范围：GPIO_PIN_0 到 GPIO_PIN_15，另外还有 GPIO_PIN_All 和 GPIO_PIN_MASK 可选。**



**②Mode：**

![image-20231031103448260](https://s2.loli.net/2023/10/31/r8oBPi3eFzGwAYS.png)

**成员 Mode 是 GPIO 的模式选择，有以上选择项：**

我们用中文在简化以下：

```c
#define GPIO_MODE_INPUT (0x00000000U) 				/* 输入模式 */
#define GPIO_MODE_OUTPUT_PP (0x00000001U) 			/* 推挽输出 */
#define GPIO_MODE_OUTPUT_OD (0x00000011U) 			/* 开漏输出 */
#define GPIO_MODE_AF_PP (0x00000002U) 				/* 推挽式复用 */
#define GPIO_MODE_AF_OD (0x00000012U) 				/* 开漏式复用 */
#define GPIO_MODE_AF_INPUT GPIO_MODE_INPUT
#define GPIO_MODE_ANALOG (0x00000003U) 				/* 模拟模式 */
#define GPIO_MODE_IT_RISING (0x10110000u) 			/* 外部中断，上升沿触发检测 */
#define GPIO_MODE_IT_FALLING (0x10210000u) 			/* 外部中断，下降沿触发检测 */

/* 外部中断，上升和下降双沿触发检测 */
#define GPIO_MODE_IT_RISING_FALLING (0x10310000u)
#define GPIO_MODE_EVT_RISING (0x10120000U) 			/*外部事件，上升沿触发检测 */
#define GPIO_MODE_EVT_FALLING (0x10220000U) 		/*外部事件，下降沿触发检测 */

/* 外部事件，上升和下降双沿触发检测 */
#define GPIO_MODE_EVT_RISING_FALLING (0x10320000U)
```



**③Pull：**

![image-20231031103623110](https://s2.loli.net/2023/10/31/HtErnmSwFJADjlz.png)

**成员 Pull 用于配置上下拉电阻**

```c
#define GPIO_NOPULL (0x00000000U) 	/* 无上下拉 */
#define GPIO_PULLUP (0x00000001U) 	/* 上拉 */
#define GPIO_PULLDOWN (0x00000002U) /* 下拉 */
```



**④Speed：**

![image-20231031103708125](https://s2.loli.net/2023/10/31/9mw3RTkjzXFlQV7.png)

**成员 Speed 用于配置 GPIO 的速度**

```c
#define GPIO_SPEED_FREQ_LOW (0x00000002U) 		/* 低速 */
#define GPIO_SPEED_FREQ_MEDIUM (0x00000001U) 	/* 中速 */
#define GPIO_SPEED_FREQ_HIGH (0x00000003U) 		/* 高速 */
```



## 五.常用的三个GPIO函数

```c
HAL_GPIO_ReadPin()		/* GPIO 口的读引脚函数 */
HAL_GPIO_WritePin()     /* GPIO 口的写引脚函数 */
HAL_GPIO_TogglePin()	/* GPIO 口的电平翻转函数 */
```

**引脚分为高电平和低电平，高电平为1，低电平为0。**



## **六.GPIO编程实战——点亮一个LED灯**

首先我们需要查看我们的原理图：

![image-20231031110857814](https://s2.loli.net/2023/10/31/KiDIHlhFUoJzTOS.png)

我们发现：

| LED1接我们的PA0引脚     |
| ----------------------- |
| **LED2接我们的PA1引脚** |
| **LED3接我们的PA2引脚** |
| **LED4接我们的PA3引脚** |

所以如何点亮我们的LED灯呢？

首先回到我们的GPIO的配置步骤

### **第一步：**

**使能时钟：**

我们发现我们接的是GPIOA

所以我们应该这样写

```c
__HAL_RCC_GPIOA_CLK_ENABLE()
```



### **第二步：**

**选择引脚：**PA0，所以PIN是0。

**设置上下拉：**不用选择，即使选择了也无效。

**设置工作模式：**

首先观看我们的原理图

我们的LED灯是输出，那我们应该设置那种输出模式呢？

我们通用的输出模式是推挽输出和开漏输出，我们又应该选择哪种呢？我们都分析一下：

1.首先是推挽输出，它既可以输出高电平也可以输出低电平。当我们的PA0是高电平时LED灯不亮，因为LED灯两端都是高电平，LED灯没有导通；当我们的PA0是低电平时，LED灯亮，因为LED灯两端有压差，LED灯导通。我们发现当我们使用推挽输出时，LED灯有两种状态，所以可以使用推挽输出。

2.然后是开漏输出，输出高电平时是高阻态也就是开路，输出低电平时是低电平。当我们的PA0是高电平时LED灯不亮，因为此时相当于开路，LED灯不导通；当我们的PA0是低电平时，LED灯亮，分析方法和推挽输出相同。所以我们也可以使用开漏输出。

这里我们选择推挽输出吧！

**设置速度：**我们点一个灯不需要很快的速度。



### 第三步：

**设置输出状态：**

我们肯定是输出低电平，所以

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, RESET)
```



我们简单讲完了步骤，我们在Keil里在详细讲一遍，并一起敲写代码。



### **Keil里的全部代码：**

**==main.c==**

```c
#include "main.h"

int main()
{
    HAL_Init();
    
    LED_Init();
    
    while(1)
    {
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET);//点亮PA0端口的LED灯
        
    }
}

```



**==main.h==**

```c
#ifndef __MAIN_H
#define __MAIN_H


#include "stm32f1xx_hal.h"
#include "led.h"

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
    /* 配置PA0，PA1，PA2，PA3四个IO口 */
    GPIO_InitStructure.Pin   = GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2 | GPIO_PIN_3;
    /* 设置工作模式 */
    GPIO_InitStructure.Mode  = GPIO_MODE_OUTPUT_PP;			//推挽输出模式
    /* 设置上下拉模式 */
    GPIO_InitStructure.Pull  = GPIO_NOPULL;					//既不上拉也不下拉
    /* 设置速度模式 */
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_LOW;			//低速模式
    /* 初始化 GPIO */
    HAL_GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    /* 设置输出状态 */
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET);	//给PA0设置低电平
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET);	//给PA1设置低电平
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, GPIO_PIN_SET);	//给PA2设置低电平
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_SET);	//给PA3设置低电平
}

```



**==led.h==**

```c
#ifndef __LED_H
#define __LED_H

#include "main.h"


void LED_Init(void);

#endif
```



## 七.led灯亮灭的延时操作

首先我们需要在魔法棒里

![image-20231031171251022](https://s2.loli.net/2023/10/31/ax274i9RgBYTA3y.png)

然后在魔方里

![image-20231031171548524](https://s2.loli.net/2023/10/31/Ssgomiuwc8EQ4Rz.png)

配置就完成啦，然后就是完善代码：

==**main.c**==

```c
#include "main.h"


int main()
{
    HAL_Init();
    
    LED_Init();
    
    delay_init();
    
    while(1)
    {
//        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET);//点亮PA0端口的LED灯
//        delay_ms(500);                                       //延时500ms
//        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET);  //熄灭PA0端口的LED灯
//        delay_ms(500);
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_3);                 //翻转PA1端口的电平
        delay_ms(500);                                         //延时500ms
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_3);                 //翻转PA1端口的电平
        delay_ms(500);                                         //延时500ms
    }
}

```



**==main.h==**

```c
#ifndef __MAIN_H
#define __MAIN_H


#include "stm32f1xx_hal.h"
#include "led.h"
#include "delay.h"
#include "sys.h"


#endif

```



