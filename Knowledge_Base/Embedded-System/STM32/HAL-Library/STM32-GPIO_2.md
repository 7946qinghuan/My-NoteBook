---
title: STM32-GPIO_2
date: 2026-08-01
tags: [Embedded, STM32, HAL]
aliases: []
---

# ==STM32第三次培训—GPIO==

上一节课我们学习了STM32的第一个外设GPIO，那么这节课我们继续学习GPIO外设之**按键**。



[TOC]



## 一.分析原理图上的按键

![image-20231109140021106](https://s2.loli.net/2023/11/09/4ZqSQyAw73YPsCT.png)

首先我们分析这一张原理图上的按键，我们可以发现对于**按键WK_UP**，它所接的GPIO端口为PA0，所以我们在初始化时应该使能GPIOA时钟，引脚为GPIO_PIN_4，工作模式为输入模式，速度可以设置也可以不设置。

最后我们来分析一下上下拉模式，首先，我们需要知道**上拉和下拉电阻**：

**1.定义：** 上拉电阻是把一个信号通过一个电阻接到电源(Vcc)，下拉电阻是一个信号通过一个电阻接到地(GND)。



**2.作用：**

上拉电阻：

①提升电路的驱动能力（提高电压，提高输出电路的驱动能力）

②将引脚电压钳制在某个值

==③将不确定的信号钳制在高电平==

下拉电阻：

①默认情况下将信号稳定在0V，而接上按键输入3.3V或5V时将信号钳制在高电平。让信号在高低电平之间转换。

==②将不确定的信号钳制在低电平==



**3.总结：**

上拉（Pull Up ）或下拉（Pull Down）电阻（两者统称为“**拉电阻**”）最基本的作用是：将状态不确定的信号线通过一个电阻将其箝位至高电平（上拉）或低电平（下拉）



**我们知道上下拉电阻后，我们便可以把下拉模式看成是下拉电阻，而上拉模式看成上拉电阻。**

所以，现在我们可以分析PA端口在按键不按下时是高阻态（开路），而按键按下是高电平，但是我们现在只有一种状态，所以我们使用下拉电阻，将不确定的信号钳制在低电平，那么现在我们就有了两种状态高电平1和低电平0，此时我们就可以判断按键是否按下了。



同理，PE2，PE3，PE4，我们选择上拉模式。



![image-20231109140156427](https://s2.loli.net/2023/11/09/k7gpxDfNuGznLZy.png)

我们现在分析我们的原理图。在我们的原理图上，我们按键不按下为高电平，而按下为低电平，所以我们已经有了两种状态，那么我们就把不需要设置上下拉了。（电容的作用是硬件消抖，我们后续做了解，目前知道就行）



关于上下拉电阻的原理，大家可以课下看视频在巩固一下。

[上拉电阻](https://www.bilibili.com/video/BV1W34y1579U/?spm_id_from=333.999.0.0)

[下拉电阻](https://www.bilibili.com/video/BV1ZU4y1Q7eo/?spm_id_from=333.999.0.0&vd_source=2daccc2d29c21699d84b67d8045701fe)



## 二.按键初始化在Keil里的实现

```c
void KEY_Init(void)
{
    /* 定义 GPIO 初始化结构体 */
    GPIO_InitTypeDef GPIO_InitStructure = {0};
    /* 使能 GPIOA 时钟 */
    __HAL_RCC_GPIOA_CLK_ENABLE();
    /* 配置PA4，PA5，PA6，PA7四个IO口 */
    GPIO_InitStructure.Pin   = GPIO_PIN_4 | GPIO_PIN_5 | GPIO_PIN_6 | GPIO_PIN_7;
    /* 设置工作模式 */
    GPIO_InitStructure.Mode  = GPIO_MODE_INPUT;             //输入模式
    /* 设置上下拉模式 */
    GPIO_InitStructure.Pull  = GPIO_NOPULL;                 //不上下拉
    /* 设置速度模式 */
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;        //低速模式
    /* 初始化 GPIO */
    HAL_GPIO_Init(GPIOA, &GPIO_InitStructure);
}
```



## 三.编程实战——通过按键控制LED灯的亮灭

### 1.按键的抖动与消抖——按键扫描函数

首先我们需要知道按键按下时，实际情况与理想情况并不完全一样

![image-20231109162232500](https://s2.loli.net/2023/11/09/lgjdtQhDopPMuFz.png)

​						   					  **独立按键抖动波形图**



通常的按键所用开关为机械弹性开关，当机械触点断开、闭合时，由于机械触点的弹性作用，一个按键开关在闭合时不会马上稳定地接通，在断开时也不会一下子断开。因而在闭合及断开的瞬间均伴随有一连串的抖动。（**抖动的时间为5-10ms**）



我们通过图片可以发现在按键按下和松开时会有一段不稳定的信号，我们无法判断它的电平，这便是抖动。但是我们更注重的是我们按下时是稳定的低电平而不是松开时的高电平，所以我们需要消除按键按下时这段不稳定的信号，此时就需要**消抖**，而消抖分为**软件消抖和硬件消抖**，我们所以用的是硬件消抖。

我们两个都讲一下：

**软件消抖就是通过延时跳过那段抖动**

```c
/* 按键扫描函数(软件消抖版) */
uint8_t key_scan_1(void)
{
    if(KEY1 == 0)               //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0
    {
        delay_ms(10);           //* 软件消抖 */
        if(KEY1 == 0)           //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0
        {
            while(KEY1 == 0);   //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0   /* 等待按键松开 */
            return 1;           /* 按键按下了 */
        }
    }
    return 0;   /* 按键没有按下 */
}
```



**硬件消抖就是将电容并联在按键的两端，利用电容的放电的延时特性。将产生抖动的电平通过电容吸收掉。从而达到消抖的作用**

电容经过电阻充电，开关未使用的默认状态是高电平。当开关闭合时，它慢慢将电容消耗至地电位，以此减弱所有小抖动的影响。

![电容基础2——充放电时间常数 - 知乎](https://pic2.zhimg.com/v2-6d5179c97761e6200a124b66427ec485_720w.jpg?source=172ae18b)

我们详细分析一下：

【PA7管脚分析】
①如上原理图所示，最开始状态时按键未按下时，电容肯定先是已经被充满电然后开路，又因为电容C15左极板与接地，所以左右极板电位0/3.3V，又因为没有按下按键，所以电路处于开路，单片机管脚PA7处于高电平3.3V（电路开路与VCC相连/也与电容右极板相连）；
②当按键按下时，有一段时间的机械抖动，此时按键KEY4右边节点的电位是3.3V（也是单片机管脚PA7的电位为3.3V），又因为按键与电容形成回路，所以电容会进行放电（电容是一个非线性元件，放电需要时间），但是机械抖动的时间和电容放电的时间不是完全一致，所以当电容放电放一部分时间（假设可能放到2.9V时）机械抖动就已经结束，那么管脚的电平却是始终都是表现为高电平，所以CPU识别还是高电平未变，当按键彻底按下稳定接触后，很快就会将放电完毕将电容短路（左右极板的电位变成0v/0v），然后按键KEY4右边节点电位接地（也就是管脚PA7因为接地电势为0v），所以此时单片机管脚PA7为低电平，这就是放电延时；
③当按键要松开时，按键有个松开时的抖动时间，只有你有松开的可能，按键这条支路就会开路而电容开始充电，电容就开始充电，但是充电需要时间在抖动的时间内电容右极板不会瞬间达到3.3V的电位，也就是按键KEY4右边的节点电位不会一下子达到高电平（可能在这个抖动时间内电容才充电充了0.5V），所以该节点依然保持低电平（也就是CPU识别的管脚PA7为低电平），当按键彻底松开稳定后，按键那条支路断开电容也充满电（两条支路都是开路），按键KEY4右边节点和管脚的电位就是高电平（与直流电源VCC相连）这就是充电延时；

```c
/* 按键扫描函数(硬件消抖版) */
uint8_t key_scan_2(void)
{
    if(KEY1 == 0)               //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0
    {
            while(KEY1 == 0);   //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0   /* 等待按键松开 */
            return 1;           /* 按键按下了 */
    }
    return 0;   /* 按键没有按下 */
}

```



### 2.按键控制LED——按键功能函数

首先，我们可以简化我们的控制LED亮灭的函数与按键的状态读取函数

```
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
```



```
#define KEY1 HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4)
#define KEY2 HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_5)
#define KEY3 HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_6)
#define KEY4 HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_7)
```



**以上函数需要放在对应的.h文件里**

当然，对于初学者而言，我们还是要多敲代码，不要用上面的方式来偷懒，当自己真正理解以后，在用上面的方法简化代码。



好的，现在我们可以开始编写我们的**按键功能函数**了

```c
void KEY_Fun(uint8_t state)
{
	if(key_scan_2())
		LED1_ON;
	else
		LED_OFF;
}

```



**以上对于初学者已经够用了，以下是提升部分。**



### 3.按键的短按与连按

```c
/* 连按与短按 */
KEY_STATUS KEY_Scan(uint8_t mode) {
    /* static是静态变量的关键词，我们需要知道静态变量的值不使用时不改变，使用时改变 */
    static uint8_t KEY_UP = 1;  //按键按松开标志
    if (mode == 1)              // mode = 1，支持连按；mode = 0，按一次执行一次
    {
        KEY_UP = 1;
    }
    if (KEY_UP && (!KEY1 || !KEY2 || !KEY3 || !KEY4))//如果KEY_UP的值不为0（按键按下标志），并且按键1-4其中任意一个按下
    {
        delay_ms(15);           //LED灯闪烁频率
        KEY_UP = 0;
        if (!KEY1)              //等价于if(KEY1 == 0)如果是按键1按下
            return KEY1_DOWN;   // 按键1按下
        else if (!KEY2)         //等价于if(KEY2 == 0)如果是按键2按下
            return KEY2_DOWN;   // 按键2按下
        else if (!KEY3)         //等价于if(KEY3 == 0)如果是按键3按下
            return KEY3_DOWN;   // 按键3按下
        else if (!KEY4)         //等价于if(KEY4 == 0)如果是按键4按下
            return KEY4_DOWN;   // 按键4按下
    }
    else if (KEY1 && KEY2 && KEY3 && KEY4 ) //如果没有按键按下，KEY_UP为1
        KEY_UP = 1;
    return KEY_NULL; // 没有按键按下
}

void KEY_FUNC(uint8_t mode) {
    switch(KEY_Scan(mode)) 
    {
        case KEY1_DOWN : LED1_Turn;break;
        case KEY2_DOWN : LED2_Turn;break;
        case KEY3_DOWN : LED3_Turn;break;
        case KEY4_DOWN : LED4_Turn;break;
        case KEY_NULL : break;              //此处未用default关键词，因为全部情况已列出
    }
}

```



```c
typedef enum KEY_STATUS
{
    KEY_NULL = 0,
    KEY1_DOWN, 
    KEY2_DOWN, 
    KEY3_DOWN, 
    KEY4_DOWN
}KEY_STATUS;

```

关于按键的状态变量在key.h里面声明



当然，以上代码仍然有缺陷，我们通过实验可以发现当我们连按后，LED灯的状态可能依然为亮的状态，那我们如何解决

```c
/* 连按与短按 */
KEY_STATUS KEY_Scan(uint8_t mode) {
    /* static是静态变量的关键词，我们需要知道静态变量的值不使用时不改变，使用时改变 */
    static uint8_t KEY_UP = 1;  //按键按松开标志
    if (mode == 1)              // mode = 1，支持连按；mode = 0，按一次执行一次
    {
        KEY_UP = 1;
    }
    if (KEY_UP && (!KEY1 || !KEY2 || !KEY3 || !KEY4))//如果KEY_UP的值不为0（按键按下标志），并且按键1-4其中任意一个按下
    {
        delay_ms(15);           //闪烁频率
        KEY_UP = 0;
        if (!KEY1)              //等价于if(KEY1 == 0)如果是按键1按下
        {
            key.key1_time++;
            return KEY1_DOWN;   // 按键1按下
        }
        else if (!KEY2)         //等价于if(KEY2 == 0)如果是按键2按下
        {
            key.key2_time++;
            return KEY2_DOWN;   // 按键2按下
        }
        else if (!KEY3)         //等价于if(KEY3 == 0)如果是按键3按下
        {
            key.key3_time++;
            return KEY3_DOWN;   // 按键3按下

        }
        else if (!KEY4)         //等价于if(KEY4 == 0)如果是按键4按下
        {
            key.key4_time++;
            return KEY4_DOWN;   // 按键4按下
        }
    }
    else if (KEY1 && KEY2 && KEY3 && KEY4 ) //如果没有按键按下，KEY_UP为1
    {
        key.key1_time = 0;
        key.key2_time = 0;
        key.key3_time = 0;
        key.key4_time = 0;
        KEY_UP = 1;
    }
    return KEY_NULL; // 没有按键按下
}

void KEY_FUNC(uint8_t mode) {
    switch(KEY_Scan(mode)) 
    {
        case KEY1_DOWN : LED1_Turn;break;
        case KEY2_DOWN : LED2_Turn;break;
        case KEY3_DOWN : LED3_Turn;break;
        case KEY4_DOWN : LED4_Turn;break;
        case KEY_NULL : break;              //此处未用default关键词，因为全部情况已列出
    }
    if((KEY1 == 1) && (LED1_Read == 0) && (key.key1_time > 2)) //2*15=30ms 所以连按大于30ms则清除LED灯状态
        LED1_OFF;
    if((KEY2 == 1) && (LED2_Read == 0) && (key.key2_time > 2))
        LED2_OFF;
    if((KEY3 == 1) && (LED3_Read == 0) && (key.key3_time > 2))
        LED3_OFF;
    if((KEY4 == 1) && (LED4_Read == 0) && (key.key4_time > 2))
        LED4_OFF;
}

```



### 4.Keil里的全部代码



**==led.c==**

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



**==led.h==**

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



**==key.c==**

```c
#include "key.h"

void KEY_Init(void){
    /* 定义 GPIO 初始化结构体 */
    GPIO_InitTypeDef GPIO_InitStructure = {0};
    /* 使能 GPIOA 时钟 */
    __HAL_RCC_GPIOA_CLK_ENABLE();
    /* 配置PA4，PA5，PA6，PA7四个IO口 */
    GPIO_InitStructure.Pin   = GPIO_PIN_4 | GPIO_PIN_5 | GPIO_PIN_6 | GPIO_PIN_7;
    /* 设置工作模式 */
    GPIO_InitStructure.Mode  = GPIO_MODE_INPUT;             //输入模式
    /* 设置上下拉模式 */
    GPIO_InitStructure.Pull  = GPIO_NOPULL;                 //不上下拉
    /* 设置速度模式 */
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;        //低速模式
    /* 初始化 GPIO */
    HAL_GPIO_Init(GPIOA, &GPIO_InitStructure);
}


/* 按键扫描函数(软件消抖版) */
uint8_t key_scan_1(void){
    if(KEY1 == 0)               //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0
    {
        delay_ms(10);           //* 软件消抖 */
        if(KEY1 == 0)           //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0
        {
            while(KEY1 == 0);   //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0   /* 等待按键松开 */
            return 1;           /* 按键按下了 */
        }
    }
    return 0;   /* 按键没有按下 */
}

/* 按键扫描函数(硬件消抖版) */
uint8_t key_scan_2(void){
    if(KEY1 == 0)               //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0
    {
            while(KEY1 == 0);   //HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4) == 0   /* 等待按键松开 */
            return 1;           /* 按键按下了 */
    }
    return 0;   /* 按键没有按下 */
}

/* 所有按键的扫描函数（无需消抖） */
uint8_t key_scan_all(void){
    if(KEY1 == 0)
    {
        while(KEY1 == 0);           /* 等待按键1松开 */
        return KEY1_DOWN;           /* 按键1按下了 */
    }
    else if(KEY2 == 0)
    {
        while(KEY2 == 0);           /* 等待按键1松开 */
        return KEY2_DOWN;           /* 按键1按下了 */
    }
    else if(KEY3 == 0)
    {
        while(KEY3 == 0);           /* 等待按键1松开 */
        return KEY3_DOWN;           /* 按键1按下了 */
    }
    else if(KEY4 == 0)
    {
        while(KEY4 == 0);           /* 等待按键1松开 */
        return KEY4_DOWN;           /* 按键1按下了 */
    }
    else
        return KEY_NULL;
}

```



**提升部分：**

```c
static KEY_TIME key = {0};

/* 连按与短按 */
KEY_STATUS KEY_Scan(uint8_t mode) {
    /* static是静态变量的关键词，我们需要知道静态变量的值不使用时不改变，使用时改变 */
    static uint8_t KEY_UP = 1;  //按键按松开标志
    if (mode == 1)              // mode = 1，支持连按；mode = 0，按一次执行一次
    {
        KEY_UP = 1;
    }
    if (KEY_UP && (!KEY1 || !KEY2 || !KEY3 || !KEY4))//如果KEY_UP的值不为0（按键按下标志），并且按键1-4其中任意一个按下
    {
        delay_ms(15);           //闪烁频率
        KEY_UP = 0;
        if (!KEY1)              //等价于if(KEY1 == 0)如果是按键1按下
        {
            key.key1_time++;
            return KEY1_DOWN;   // 按键1按下
        }
        else if (!KEY2)         //等价于if(KEY2 == 0)如果是按键2按下
        {
            key.key2_time++;
            return KEY2_DOWN;   // 按键2按下
        }
        else if (!KEY3)         //等价于if(KEY3 == 0)如果是按键3按下
        {
            key.key3_time++;
            return KEY3_DOWN;   // 按键3按下

        }
        else if (!KEY4)         //等价于if(KEY4 == 0)如果是按键4按下
        {
            key.key4_time++;
            return KEY4_DOWN;   // 按键4按下
        }
    }
    else if (KEY1 && KEY2 && KEY3 && KEY4 ) //如果没有按键按下，KEY_UP为1
    {
        key.key1_time = 0;
        key.key2_time = 0;
        key.key3_time = 0;
        key.key4_time = 0;
        KEY_UP = 1;
    }
    return KEY_NULL; // 没有按键按下
}

void KEY_FUNC(uint8_t mode) {
    switch(KEY_Scan(mode)) 
    {
        case KEY1_DOWN : LED1_Turn;break;
        case KEY2_DOWN : LED2_Turn;break;
        case KEY3_DOWN : LED3_Turn;break;
        case KEY4_DOWN : LED4_Turn;break;
        case KEY_NULL : break;              //此处未用default关键词，因为全部情况已列出
    }
    if((KEY1 == 1) && (LED1_Read == 0) && (key.key1_time > 2)) //2*15=30ms 所以长按大于30ms则清除LED灯状态
        LED1_OFF;
    if((KEY2 == 1) && (LED2_Read == 0) && (key.key2_time > 2))
        LED2_OFF;
    if((KEY3 == 1) && (LED3_Read == 0) && (key.key3_time > 2))
        LED3_OFF;
    if((KEY4 == 1) && (LED4_Read == 0) && (key.key4_time > 2))
        LED4_OFF;
}

```



**==key.h==**

```c
#ifndef __KEY_H
#define __KEY_H

#include "stm32f1xx_hal.h"
#include "delay.h"
#include "led.h"

#define KEY1 HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_4)
#define KEY2 HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_5)
#define KEY3 HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_6)
#define KEY4 HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_7)

typedef enum KEY_STATUS
{
    KEY_NULL = 0,
    KEY1_DOWN, 
    KEY2_DOWN, 
    KEY3_DOWN, 
    KEY4_DOWN
}KEY_STATUS;

typedef struct KEY_TIME
{
    int key1_time;
    int key2_time;
    int key3_time;
    int key4_time;
    
}KEY_TIME;

void KEY_Init(void);
uint8_t key_scan_1(void);
uint8_t key_scan_2(void);
uint8_t key_scan_all(void);
KEY_STATUS KEY_Scan(uint8_t mode);
void KEY_FUNC(uint8_t mode);

#endif

```



**==main.c==**

```c
#include "main.h"

extern KEY_TIME key;

int main()
{
    HAL_Init();
    
    LED_Init();
    KEY_Init();
    delay_init();
    
    while(1)
    {
        
    }
}

```



**==main.h==**

```c
#ifndef __MAIN_H
#define __MAIN_H


#include "stm32f1xx_hal.h"
#include "led.h"
#include "key.h"
#include "delay.h"
#include "sys.h"


#endif

```

