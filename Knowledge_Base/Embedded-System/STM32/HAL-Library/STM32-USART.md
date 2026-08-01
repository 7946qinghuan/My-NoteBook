# ==STM32第五次培训—USART==



[TOC]



## 一.串口通信概述

### （一）串口

串口是串行接口 (Serial Interface)的简称，它是指数据一位一位地顺序传送，其特点是通信线路简单，只要一对传输线就可以实现双向通信（可以直接利用电话线作为传输线），从而大大降低了成本，特别适用于远距离通信，但传送速度较慢。一条信息的各位数据被逐位按顺序传送的通讯方式称为串行通讯。串行通讯的特点是：数据位的传送，按位顺序进行，最少只需一根传输线即可完成；成本低但传送速度慢。串行通讯的距离可以从几米到几千米；根据信息的传送方向，串行通讯可以进一步分为单工、半双工和全双工三种。

### （二）协议

所谓协议，就是通信双方约定好的规定，通信双方只有遵守这个规定才能够完成任务。举个栗子就是周幽王烽火戏诸侯，双方约定好以烽火为信号进行通信，但是愚蠢的周幽王为博美人褒姒一笑破坏了这个规定，最后付出的代价是惨重的。可见，通信双方只有遵守协议才能够完成通信。

### （三）时序

时序就是协议的实际化，它实质上是一些列的脉冲信号，通信双方将信息按照预先定好的规定（协议）转换成一系列的脉冲信号，通过总线发送给接收方，接收方再将接收到的数据按照规定进行解析，从而得到发送方发送过来的数据。

### （四）上位机

上位机和下位机其实是一个相对的概念，上位机指的是可以直接发出操控命令的计算机，一般指PC机，能够显示各种信号变化（液压，水位，温度等），能够将信息直接传递给人。下位机是直接控制设备获取设备状况的计算机，一般是PLC/单片机single chip microcomputer/slave computer/lower computer之类的，下位机需要PC机来对其进行控制。

关于USART

stm32有丰富的通讯外设，USART（Universal Synchronous Asynchronous Receiver Transmitter）、SPI（Serial Peripheral interface）、I2c（Inter-Integrated Circuit）、CAN（Controller Area Network），因为stm32有完整的且强大的固件库，这使得配置串口的难度大大降低了，和用软件IO口模拟通信时序相比，硬件的支持可以大大提高通信的速率、大大降低出错的概率，从而提高了通信的质量和效率。用IO口模拟USART难度较大，它对延时要求比较苛刻，且出错的概率较大，所以一般很少用IO口模拟USART。IO口模拟I2c比较常见，由于I2c的最高通信速度只有3.4M/s，单片机的IO口速度可以完美驾驭。由于SPI多用于一些较高速的通信，例如LCD、OLED、TFT显示器的写入，EEPROM (Electrically Erasable Programmable read only memory)的写入和读取，用IO口模拟效果不是很理想，所以建议使用硬件自带接口。



## 二.数据通信的基本概念

### 1.串行/并行通信

![通信](https://s2.loli.net/2023/11/16/9JCcNRnzdfK7AtH.png)



总结：

串行通信：单对单，逐个发送。

并行通信：多对多，同时发送。



### 2.单工/半双工/全双工通信

![通信](https://s2.loli.net/2023/11/16/RofsYFkNCj12PIl.png)

总结：

单工通信：单方向传输，一条输入输出通道。

半双工通信：双方向传输，一条输入输出通道，错时进行

全双工通信：双方向传输，两条输入输出通道，同时进行



### 3.同步/异步通信

![同步/异步通信](https://s2.loli.net/2023/11/16/ciPSFdux84JbtYm.png)

总结：

同步通信：有时钟线，数据由时钟信号得来。

异步通信：无时钟线，数据通过数据帧来获取。



### 4.波特率

![波特率](https://s2.loli.net/2023/11/16/GIzoDJ4Z5Ssi9Ff.png)



基础知识前瞻：

①比特数：二进制的位数。

②码元与码元数：

码元：码元是承载信息量的基本信号单位，在数字通信中常用**时间间隔相同的符号**来表示一个二进制数字，这样的时间间隔内的信号称为（二进制）**码元**。而这个间隔被称为**码元长度**。值得注意的是当码元的离散状态有大于2个时（如M大于2个）时，此时码元为 M 进制码元。

举例：假定基带信号为 10101100011011101

- 直接传送。也就是上面每位二进制数都是一个码元，这种方式被称为二进制码元。发送的过程就是：1、0、1、0……，传多少个数字就要用多少个码元。每个码元的信息量是 1bit（用自信息量的公式计算即可）。

- 如果两两一组，发送的过程就是：10、10、11……，两个二进制数为一个码元，这种方式被称为四进制码元。每个码元的信息量是 2bit。

- 将上面的信号3个一组，分为 101、011、000、110、111、010……，这被称为八进制码元，每个码元为 3bit。

- 类比下去，n 个二进制数一组，就能构成 M 进制码元，其中 M=2 ^n^ 。

码元数：码元数就是信号经过调制（转换为我们认为定义的2进制数），编码（每个码元能够携带的二进制数）的个数，即码元数M=2 ^n^ 。



### 5.常见的串行通信接口

![串行通信接口](https://s2.loli.net/2023/11/16/B6xFvYbHcsNzUil.png)





## 三.STM32F103C8T6之USART简介

### 1.UART与USART简介

**U**niversal **s**ynchronous **a**synchronous **r**eceiver **t**ransmitter，通用同步异步收发器

**U**niversal **a**synchronous **r**eceiver **t**ransmitter，通用异步收发器

USART/UART都可以与外部设备进行全双工异步通信

USART，我们常用的也是异步通信

![USART简介](https://s2.loli.net/2023/11/16/g2WkflN5Z6jIois.png)



从图上可以发现我们STM32F103C8T6有3个串口，它们分别是：

| STM32F103C8T6串口号 | TXD  | RXD   |
| ------------------- | ---- | ----- |
| 1                   | PA 9 | PA 10 |
| 2                   | PA 2 | PA 3  |
| 2                   | PB10 | PB11  |



### 2.USART框图介绍

![框图](https://s2.loli.net/2023/11/16/wFMsEyl8bTcP9G5.png)

在黄色部分中：

我们只需要知道Tx是发送数据，Rx是接收数据 

RTS和CTS都是硬件数据流，是关于同步通信的，而我们目前学习的是异步通信，我们暂且不做了解

SCLK是同步时钟，同理是关于同步通信的，我们暂且不做了解



在红色部分中：

我们只需要了解我们接受数据和发送数据是通过操作DR寄存器就行了。



在绿色部分中：

整个绿色部分是我们传统的波特率发送器，USART_BRR是我们的波特率摄制寄存器，TE是发送使能位，RE是接受使能位，只有当使能位置1时，才能实现对应的功能。中间部分是用来存放我们设置的波特率的值，而波特率的值由外面蓝色紫框部分决定。



在蓝色部分中：

发送器控制和接收器控制需要我们的蓝色紫框部分提供的波特率，而接收器控制又需要唤醒单元去唤醒它。



橙色部分就是各种寄存器



紫色部分就是我们计算波特率值的公式。



关于USART的框图对于初学者非常困难，所以我们一般看简化版本就够了。

![简化框图](https://s2.loli.net/2023/11/16/SX7oRfB1bW4n5ye.png)





## 四.设置USART/UART波特率（F1/F4）

首先我们来到我们USART框图的波特率部分

![波特率](https://s2.loli.net/2023/11/16/P3RWlkhz7amObM5.png)



首先我们可以波特率计算公式为：




$$
baud = \frac{f_{ck}}{16∗USARTDIV}
$$
![芯片手册](https://s2.loli.net/2023/11/16/JYrS6sPnGDKHgtd.png)

​	

其中f~ck~是串口的时钟，如：USART1的时钟是PCLK2,其他串口都是PCLK1

通过上面的芯片数据手册我们可以看到我们的USART1挂载在APB2总线时钟上，其对应的最高稳定值是72Mhz；而USART2和USART3挂载在APB1总线时钟上，其对应的最高稳定值是36Mhz。



有了计算波特律的公式，我们通过设置不同的波特率可以求出USARTDIV，而我们的USARTDIV
$$
USARTDIV = {DIV\_Mantissa} + (DIV\_Fraction/16)
$$


![image-20231116195127946](https://s2.loli.net/2023/11/16/LIWHdp3nGZEyXVS.png)

其中把USARTDIV的整数部分写入位[15:4]， USARTDIV的小数部分写入位[3:0]

由此我们便可以进行计算：

![计算公式](https://s2.loli.net/2023/11/16/tnP7VAgOBWeQq21.png)

```c
uint16_t mantissa; 
uint16_t fraction; 
mantissa = 39; 
fraction = 0.0625*16+0.5=0x01;          /* USARTDIV = DIV_Mantissa + (DIV_Fraction/16) */
USART1->BRR = (mantissa << 4) + fraction;

```



![F1波特率公式推演](https://s2.loli.net/2023/11/16/aJINGnLSKTM7dvq.png)

我们拓展一下：

其中F4是同理的

![image-20231116202847202](https://s2.loli.net/2023/11/16/axyPH1wKi8I73gv.png)




$$
baud =\frac{f_{ck}}{8∗(2−OVER8)∗USARTDIV}
$$
![image-20231116203136797](https://s2.loli.net/2023/11/16/5cmowMKbyECVWGL.png)

我们一般采用16倍过采样，所以OVER8等于0。



## 五.USART寄存器的介绍

### 1.控制寄存器1（CR1）

| 该寄存器需要完成的配置：    |
| --------------------------- |
| 位13：使能USART             |
| 位12：配置8个数据位         |
| 位10：禁止检验控制          |
| 位5：使能接收缓冲区非空中断 |
| 位3：使能发送               |
| 位2：使能接收               |

`请参考：STM32F10xxx参考手册_V10（中文版）.pdf，25.6.4节`



### 2.控制寄存器2（CR2）

![image-20231117163129946](https://s2.loli.net/2023/11/17/tZov3KPysFUw1gE.png)

该寄存器需要完成的配置：配置1个停止位

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，25.6.5节`



### 3.控制寄存器3（CR3）

![image-20231117163335336](https://s2.loli.net/2023/11/17/izrFIU9RySGJ6ht.png)

该寄存器需要完成的配置：配置不选择半双工模式

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，25.6.6节`



### 4.数据寄存器（DR）

![image-20231117163522334](https://s2.loli.net/2023/11/17/mBlafE7Uj3MgRLC.png)

设置好控制和波特率寄存器后，往该寄存器写入数据即可发送，接收数据则读该寄存器

`摘自：STM32F10xxx参考手册_V10（中文版）.pdf，25.6.2节`



### 5.状态寄存器（SR）

![image-20231117163751539](https://s2.loli.net/2023/11/17/Q9XAZ8cRpad7frW.png)

根据TC位可以知道能否发数据，根据RXNE位知道是否收到数据



![image-20231117164003939](https://s2.loli.net/2023/11/17/4ViFIZld13SgLJw.png)



## 六.外设初始化MSP回调机制及中断回调机制

### 1.问题提出

在STM32的HAL库使用中，会发现库函数大都被设计成了一对：

```c
HAL_PPP/PPPP_Init
```

```c
HAL_PPP/PPPP_MspInit
```

而且HAL_PPP/PPPP_MspInit函数的defination前面还会有__weak关键字

上面的PPP/PPPP代表常见外设的名称为3个字符或者4个字符

怎么理解这个设计呢？



### 2.问题分析

#### 2.1 结论

首先说结论：

 HAL_PPP/PPPP_Init 是与具体芯片（无论是STM32F4/F1/F7）**无关**的设置

 HAL_PPP/PPPP_MspInit 是与具体芯片**相关**的配置（如STM32F429IGTx）

这样的设计是将不变的东西以库函数HAL_PPP/PPPP_Init的形式固定下来，而将需要用户根据

芯片进行编写的部分抽象成函数HAL_PPP/PPPP_MspInit的形式，用户只需要编写这部分函数

即可，这样做减少了用户的代码编写量

**总结：不同的芯片上的外设及其数量不同，所以用一个函数HAL_PPP/PPPP_MspInit来解决使用不同芯片时带来的代码移植问题或使用问题。**



**__weak**关键字的使用是定义一个弱函数，这个函数的函数体通常是空的

方便用户重写一个自己的函数HAL_PPP/PPPP_MspInit，来覆盖之前库函数中定义的函数带有

__weak关键字的HAL_PPP/PPPP_MspInit函数，编译器在编译的时候，如果检查到有重名的

（但不含__weak关键字）HAL_PPP/PPPP_MspInit的函数，此时就会默认编译这个用户写的函数

**总结：带有__week关键词的函数可以由我们用户重定义。**



#### 2.2 实例分析

以串口通信为例进行分析：

在编写串口通信的代码的时候，常使用usart.c&usart.h组合，在usart.c中

定义了HAL_UART_MspInit作为回调函数：

```c
/* 串口MSP回调函数 */
void HAL_UART_MspInit(UART_HandleTypeDef *huart)
{
    GPIO_InitTypeDef GPIO_InitStruct;
    if(huart->Instance == USART1)                		   /* 如果是串口1，进行串口1 MSP初始化 */
    {
        /* 串口1时钟使能 */
        __HAL_RCC_USART1_CLK_ENABLE();
        /* GPIOA时钟使能 */
        __HAL_RCC_GPIOA_CLK_ENABLE();
        
        //TX端
        GPIO_InitStruct.Pin = GPIO_PIN_9;                  /* PA9 */
        GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;            /* 推挽式复用输出 */
        GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;      /* 高速 */
        HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);            /* 初始化串口1的TX引脚 */
        
        //RX端
        GPIO_InitStruct.Pin = GPIO_PIN_10;                 /* PA10 */
        GPIO_InitStruct.Mode = GPIO_MODE_AF_INPUT;         /* 复用输入 */
        GPIO_InitStruct.Pull = GPIO_PULLUP;                /* TX线开始默认发送高电平所以上拉 */
        HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);            /* 初始化串口1的RX引脚 */
        
        HAL_NVIC_SetPriority(USART1_IRQn, 2, 0);           /* 抢占优先级2,子优先级0 */
        HAL_NVIC_EnableIRQ(USART1_IRQn);                   /* 使能USART1中断通道 */
    }
}
```

同时编写我们的串口初始化的接口：

在这之前我们需要定义我们的串口1句柄

```c
UART_HandleTypeDef huart1;  /* UART句柄(Uart1_handle) */
```



```c
void USART_Init(uint32_t bound) // bound为波特率
```



```c
/* 串口1初始化函数 */
void USART_Init(uint32_t baudrate)
{
    //选择串口1
    huart1.Instance = USART1;
    //设置波特率
    huart1.Init.BaudRate = baudrate;
    //选择数据位，8
    huart1.Init.WordLength = UART_WORDLENGTH_8B;
    //选择停止位，1
    huart1.Init.StopBits = UART_STOPBITS_1;
    //选择奇偶校验位，不选择
    huart1.Init.Parity = UART_PARITY_NONE;
    //选择硬件流控，无
    huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;
    //数据方式，收发
    huart1.Init.Mode = UART_MODE_TX_RX;
    //默认16倍过采样，F1系列可以不用设置
    huart1.Init.OverSampling = UART_OVERSAMPLING_16;
    //初始化串口1
    HAL_UART_Init(&huart1);
    
    /* 开启串口接受中断进入 */
    //HAL_UART_Receive_IT(&huart1, (uint8_t*)g_rx_buffer, 1);
}
```

这样在main函数中，首先调用函数uart_init()

注意：

```c
HAL_UART_Receive_IT(&huart1, (uint8_t*)g_rx_buffer, 1);
```

此句函数我们一般放在主函数里面

然后USART_Init()函数就会去调用HAL_UART_Init()，这个函数就是HAL库中的函数

![img](https://img-blog.csdnimg.cn/20190923214133449.png)

跳转到文件stm32f4xx_hal_uart.c，找到函数HAL_UART_Init的定义：

```c
/**
  * @brief  Initializes the UART mode according to the specified parameters in
  *         the UART_InitTypeDef and create the associated handle.
  * @param  huart: pointer to a UART_HandleTypeDef structure that contains
  *                the configuration information for the specified UART module.
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_UART_Init(UART_HandleTypeDef *huart)
{
  /* Check the UART handle allocation */
  if(huart == NULL)
  {
    return HAL_ERROR;
  }
 
  /* Check the parameters */
  if(huart->Init.HwFlowCtl != UART_HWCONTROL_NONE)
  { 
    /* The hardware flow control is available only for USART1, USART2, USART3 and USART6 */
    assert_param(IS_UART_HWFLOW_INSTANCE(huart->Instance));
    assert_param(IS_UART_HARDWARE_FLOW_CONTROL(huart->Init.HwFlowCtl));
  }
  else
  {
    assert_param(IS_UART_INSTANCE(huart->Instance));
  }
  assert_param(IS_UART_WORD_LENGTH(huart->Init.WordLength));
  assert_param(IS_UART_OVERSAMPLING(huart->Init.OverSampling));
  
  if(huart->gState == HAL_UART_STATE_RESET)
  {  
    /* Allocate lock resource and initialize it */
    huart->Lock = HAL_UNLOCKED;
    /* Init the low level hardware */
    HAL_UART_MspInit(huart);
  }
 
  huart->gState = HAL_UART_STATE_BUSY;
 
  /* Disable the peripheral */
  __HAL_UART_DISABLE(huart);
  
  /* Set the UART Communication parameters */
  UART_SetConfig(huart);
  
  /* In asynchronous mode, the following bits must be kept cleared: 
     - LINEN and CLKEN bits in the USART_CR2 register,
     - SCEN, HDSEL and IREN  bits in the USART_CR3 register.*/
  huart->Instance->CR2 &= ~(USART_CR2_LINEN | USART_CR2_CLKEN);
  huart->Instance->CR3 &= ~(USART_CR3_SCEN | USART_CR3_HDSEL | USART_CR3_IREN);
  
  /* Enable the peripheral */
  __HAL_UART_ENABLE(huart);
  
  /* Initialize the UART state */
  huart->ErrorCode = HAL_UART_ERROR_NONE;
  huart->gState= HAL_UART_STATE_READY;
  huart->RxState= HAL_UART_STATE_READY;
  
  return HAL_OK;
}
```

可以看到函数HAL_UART_Init中调用了函数HAL_UART_MspInit

在库文件中本身是有一个同名的使用__weak关键字定义的函数，

```c
/**
  * @brief  UART MSP Init.
  * @param  huart: pointer to a UART_HandleTypeDef structure that contains
  *                the configuration information for the specified UART module.
  * @retval None
  */
 __weak void HAL_UART_MspInit(UART_HandleTypeDef *huart)
{
   /* Prevent unused argument(s) compilation warning */
  UNUSED(huart);
  /* NOTE: This function Should not be modified, when the callback is needed,
           the HAL_UART_MspInit could be implemented in the user file
   */ 
}
```

由于我们在usart.c里面重新编写了这个函数，所以编译器在编译的时候就不会再编译这个HAL库自带的函数HAL_UART_MspInit

而是编译引入的库函数HAL_UART_MspInit



MSP回调机制顺序总结：

```c
USART_Init() --> HAL_UART_Init() --> HAL_UART_MspInit()
```



### 3.中断回调机制详解

![image-20231126134259311](https://s2.loli.net/2023/11/26/VeZqoraJknHKAEU.png)

首先通过实例分析我们可以确定的是

```c
 HAL_PPP/PPPP_Init 
```

是用来设置外设参数，

而

```
 HAL_PPP/PPPP_MspInit 
```

是用来初始化配置外设的，可以由用户重定义。



那么我们的中断回调机制是什么呢？

HAL库中的中断处理机制与固件库中不同，他是经过公共中断处理函数，自动调用中断处理回调函数。用户想要再中断中实现的逻辑代码则要放在回调函数中，而公共中断处理函数会帮你检测是否有中断发生，并帮你清除中断标志位。

![image-20231126143651534](https://s2.loli.net/2023/11/26/4HkNYbOcKfVFBMx.png)



HAL_PPP_IRQHandler();公共中断处理函数，它会自动调用中断处理回调函数HAL_PPP_xxxCallback()
而公共中断处理函数需要用户写在中断服务处理函数中，同时公共中断处理函数会帮你清除中断标志,并且自动调用回调函数。

由于公共中断处理函数会帮你清除中断标志，所以需要在中断服务处理函数中重新开启串口接收中断。

```c
/* 串口1中断服务函数 */
void USART1_IRQHandler(void)
{
    HAL_UART_IRQHandler(&huart1);
    HAL_UART_Receive_IT(&huart1, (uint8_t*)g_rx_buffer, 1);
}
```

串口1中断服务函数定义在startup_stm32f103xb.s文件中



```c
/* 串口数据接收完成回调函数 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if(huart->Instance == USART1)
    {
        g_usart1_rx_flag = 1;	/* 串口数据接收完毕标志 */
    }
    
}
```



总结：在中断服务函数中执行顺序如下

![image-20231126144804753](https://s2.loli.net/2023/11/26/ehsprm6HBSNlVF4.png)



## 七.USART异步通信配置步骤总结

![image-20231130105421810](https://s2.loli.net/2023/11/30/DafUkX8ItydR4MK.png)



### 1.关键函数介绍：

#### ①.HAL_UART_Init 函数

 要使用一个外设首先要对它进行初始化，所以先看串口的初始化函数，其声明如下： 

```c
HAL_StatusTypeDef HAL_UART_Init(UART_HandleTypeDef *huart); 
```

⚫ 函数描述： 用于初始化异步模式的收发器。 

⚫ 函数形参：

形参1：串口的句柄，结构体类型是 UART_HandleTypeDef，其定义如下：

```c
typedef struct
{
 USART_TypeDef *Instance; 			/* UART 寄存器基地址 */
 UART_InitTypeDef Init; 			/* UART 通信参数 */
 uint8_t *pTxBuffPtr; 				/* 指向 UART 发送缓冲区 */
 uint16_t TxXferSize; 				/* UART 发送数据的大小 */
 __IO uint16_t TxXferCount; 		/* UART 发送数据的个数 */
 uint8_t *pRxBuffPtr; 				/* 指向 UART 接收缓冲区 */
 uint16_t RxXferSize;			    /* UART 接收数据大小 */
 __IO uint16_t RxXferCount; 		/* UART 接收数据的个数 */
 DMA_HandleTypeDef *hdmatx; 		/* UART 发送参数设置（DMA） */
 DMA_HandleTypeDef *hdmarx; 		/* UART 接收参数设置（DMA） */
 HAL_LockTypeDef Lock;			    /* 锁定对象 */
 __IO HAL_UART_StateTypeDef gState; /* UART 发送状态结构体 */
 __IO HAL_UART_StateTypeDef RxState; /* UART 接收状态结构体 */
 __IO uint32_t ErrorCode; 			/* UART 操作错误信息 */
}UART_HandleTypeDef; 
```

 1）Instance：指向 UART 寄存器基地址。实际上这个基地址 HAL 库已经定义好了，可以选择范围：USART1~ USART3。

 2）Init：UART 初始化结构体，用于配置通讯参数，如波特率、数据位数、停止位等等。下面我 们再详细讲解这个结构体。

 3）pTxBuffPtr，TxXferSize，TxXferCount：分别是指向发送数据缓冲区的指针，发送数据的大小，发送数据的个数。

 4）pRxBuffPtr，RxXferSize，RxXferCount：分别是指向接收数据缓冲区的指针，接受数据的大小，接收数据的个数； 

 5）hdmatx，hdmarx：配置串口发送接收数据的DMA具体参数。 

 6）Lock：对资源操作增加操作锁保护功能，可选 HAL_UNLOCKED 或者 HAL_LOCKED 两个参数。如果 gState 的值等于 HAL_UART_STATE_RESET，则可认为串口未被初始化，此时， 分配锁资源，并且调用 HAL_UART_MspInit 函数来对串口的 GPIO 和时钟进行初始化。

 7）gState，RxState：分别是 UART 的发送状态、工作状态的结构体和 UART 接受状态的结构 体。HAL_UART_StateTypeDef 是一个枚举类型，列出串口在工作过程中的状态值，有些值只 适用于 gState，如 HAL_UART_STATE_BUSY。

 8）ErrorCode：串口错误操作信息。主要用于存放串口操作的错误信息。 



下面，我们来了解 UART_InitTypeDef 这个结构体类型，该结构体用于配置 UART 的各个通信参数，包括波特率，停止位等，具体说明如下：

```c
typedef struct
{
 uint32_t BaudRate; 	/* 波特率 */
 uint32_t WordLength; 	/* 字长 */
 uint32_t StopBits; 	/* 停止位 */
 uint32_t Parity; 		/* 校验位 */
 uint32_t Mode;		    /* UART 模式 */
 uint32_t HwFlowCtl; 	/* 硬件流设置 */
 uint32_t OverSampling; /* 过采样设置 */
}UART_InitTypeDef;
```

1）BaudRate：波特率设置。一般设置为 2400、9600、19200、115200。
2）WordLength：数据帧字长，可选 8 位或 9 位。这里我们设置为 8 位字长数据格式。
3）StopBits：停止位设置，可选 0.5 个、1 个、1.5 个和 2 个停止位，一般我们选择 1 个停止位。
4）Parity：奇偶校验控制选择，我们设定为无奇偶校验位。
5）Mode：UART 模式选择，可以设置为只收模式，只发模式，或者收发模式。这里我们设置为
全双工收发模式。
6）HwFlowCtl：硬件流控制选择，我们设置为无硬件流控制。
7）OverSampling：过采样选择，选择 8 倍过采样或者 16 过采样，一般选择 16 过采样。

⚫ 函数返回值：

HAL_StatusTypeDef 枚举类型的值，有 4 个，分别是 HAL_OK 表示成功，HAL_ERROR 表
示错误，HAL_BUSY 表示忙碌，HAL_TIMEOUT 超时。后续遇到该结构体也是一样的。

```c
typedef enum
{
  HAL_OK       = 0x00U,
  HAL_ERROR    = 0x01U,
  HAL_BUSY     = 0x02U,
  HAL_TIMEOUT  = 0x03U
} HAL_StatusTypeDef;
```



#### 串口查询模式：

#### ②.HAL_UART_Transmit 函数

其声明如下：

```c
HAL_StatusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout);
```

⚫ 函数描述： 以阻塞的方式（阻塞模式）发送指定字节的数据。

⚫ 函数形参： 

形参1：**huart** 是指向**UART_HandleTypeDef**结构的**指针**，该结构包含指定UART模块的配置信息，也就是串口句柄。

形参2：**pData** 是指向**数据缓冲区**的**指针**（u8或u16数据元素），也就是要发送的数据地址。

形参3：**Size** 是要**发送**的数据元素的数量（u8或u16），以字节为单位。

形参4：**Timeout** 是超时时间，单位是**ms**。

⚫ 函数返回值： HAL_StatusTypeDef 枚举类型的值，返回此时的HAL状态。



#### ③.HAL_UART_Receive函数

其声明如下：

```c
HAL_StatusTypeDef HAL_UART_Receive(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout)
```

⚫ 函数描述： 以阻塞的方式（阻塞模式）接收指定字节的数据。

⚫ 函数形参： 

形参1：**huart** 是指向**UART_HandleTypeDef**结构的**指针**，该结构包含指定UART模块的配置信息，也就是串口句柄。

形参2：**pData** 是指向**数据缓冲区**的**指针**（u8或u16数据元素），也就是要接收的数据地址。

形参3：**Size**是 要**接收**的数据元素的数量（u8或u16），以字节为单位。

形参4：**Timeout** 是超时时间，单位是**ms**。

⚫ 函数返回值： HAL_StatusTypeDef 枚举类型的值，返回此时的HAL状态。



注意：

```c
(uint8_t*)：
    1字节     uint8_t
    2字节     uint16_t
    4字节     uint32_t
    8字节     uint64_t
```



#### 串口中断模式：

#### ④. HAL_UART_Transmit_IT函数

HAL_UART_Receive_IT 函数是开启串口发送中断函数。其声明如下：

```c
HAL_StatusTypeDef HAL_UART_Transmit_IT(UART_HandleTypeDef *huart, const uint8_t *pData, uint16_t Size)
```

⚫ 函数描述： 用于开启以中断的方式（非阻塞模式）发送指定字节。数据发送在中断处理函数里面实现。

⚫ 函数形参： 

形参1：**huart** 是指向**UART_HandleTypeDef**结构的**指针**，该结构包含指定UART模块的配置信息，也就是串口句柄。

形参2：**pData**是 指向数据缓冲区的指针（u8或u16数据元素），也就是要发送的数据地址。 

形参3：**Size** 是要**发送**的数据元素的数量（u8或u16），以字节为单位。

⚫ 函数返回值： HAL_StatusTypeDef 枚举类型的值，返回此时的HAL状态。



#### ⑤.HAL_UART_Receive_IT 函数

HAL_UART_Receive_IT 函数是开启串口接收中断函数。其声明如下：

```c
 HAL_StatusTypeDef HAL_UART_Receive_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size);
```

⚫ 函数描述： 用于开启以中断的方式（非阻塞模式）接收指定字节。数据接收在中断处理函数里面实现。 

⚫ 函数形参： 

形参1：**huart** 是指向**UART_HandleTypeDef**结构的**指针**，该结构包含指定UART模块的配置信息，也就是串口句柄。

形参2：**pData**是 指向数据缓冲区的指针（u8或u16数据元素），也就是要接收的数据地址。 

形参3：**Size** 是要**接收**的数据元素的数量（u8或u16），以字节为单位。

⚫ 函数返回值： HAL_StatusTypeDef 枚举类型的值，返回此时的HAL状态。



#### ⑥.HAL_UART_IRQHandler 函数

HAL_UART_IRQHandler 函数是 HAL 库中断处理公共函数。其声明如下：

```c
void HAL_UART_IRQHandler(UART_HandleTypeDef *huart);
```

⚫ 函数描述：

该函数是 HAL 库中断处理公共函数，在串口中断服务函数中被调用。

⚫ 函数形参：

形参1：UART_HandleTypeDef 结构体指针类型的串口句柄。

⚫ 函数返回值：
无

⚫ 注意事项：

 该函数是 HAL 库已经定义好，用户一般不能随意修改。如果用户要在中断中实现自己的逻辑代码，可以直接在函数 HAL_UART_IRQHandler 的前面或者后面添加新代码，也可以直接在 HAL_UART_IRQHandler 调用的各种回调函数里面执行，这些回调都是弱定义的，方便用户直接在其它文件里面重定义。串口回调函数主要有下面几个：

```c
__weak void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_TxHalfCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_RxHalfCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_ErrorCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_AbortCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_AbortTransmitCpltCallback(UART_HandleTypeDef *huart)
__weak void HAL_UART_AbortReceiveCpltCallback(UART_HandleTypeDef *huart)
```



### 2.usart.c与usart.h函数的编写

首先需要在HALLIB中添加如下驱动文件

![image-20231203142433683](https://s2.loli.net/2023/12/03/eKOl3EYwo7jDSMI.png)



#### 2.1==usart.c==的编写

##### ①首先我们需要重写print函数与scanf函数

```c
/* 重写printf函数 */
/*方便我直接使用printf来打印数据*/
int fputc(int ch, FILE *stream)
{
    (void)stream;//由于我们没有使用stream这个参数
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xffff);
    return 1;
}

/* 重写getchar/scanf函数 */
/* 还是用getchar吧，scnaf有bug，不知道为什么需要点击两次才能发送数据 */
int fgetc(FILE *stream)
{
    (void)stream;
    uint8_t ch = 0;
    HAL_UART_Receive(&huart1, &ch, 1, 0xffff);
    return ch;
}
```

huart1为我们定义的串口1句柄



##### ②定义串口相关变量

```c
uint8_t g_rx_buffer[1];             /* HAL库使用的串口接收数据缓冲区 */

uint8_t g_usart1_rx_flag = 0;       /* 串口接收到数据标志，0代表未接受到，1代表接收到 */

UART_HandleTypeDef huart1;  /* UART句柄(uart1_handle) */
```



##### ③串口1初始化函数的编写

```c
/* 串口1初始化函数 */
void USART_Init(uint32_t baudrate)
{
    //选择串口1
    huart1.Instance = USART1;
    //设置波特率
    huart1.Init.BaudRate = baudrate;
    //选择数据位，8
    huart1.Init.WordLength = UART_WORDLENGTH_8B;
    //选择停止位，1
    huart1.Init.StopBits = UART_STOPBITS_1;
    //选择奇偶校验位，不选择
    huart1.Init.Parity = UART_PARITY_NONE;
    //选择硬件流控，无
    huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;
    //数据方式，收发
    huart1.Init.Mode = UART_MODE_TX_RX;
    //默认16倍过采样
    huart1.Init.OverSampling = UART_OVERSAMPLING_16;
    //初始化串口1
    HAL_UART_Init(&huart1);
    
    /* 开启串口接受中断进入 */
    //HAL_UART_Receive_IT(&huart1, (uint8_t*)g_rx_buffer, 1);
}
```



```c
HAL_UART_Receive_IT(&huart1, (uint8_t*)g_rx_buffer, 1);
```

开启串口接受中断进入一般放在主函数里



##### ④串口MSP回调函数的编写

```c
/* 串口MSP回调函数 */
void HAL_UART_MspInit(UART_HandleTypeDef *huart)
{
    GPIO_InitTypeDef GPIO_InitStruct;
    if(huart->Instance == USART1)                /* 如果是串口1，进行串口1 MSP初始化 */
    {
        /* 串口1时钟使能 */
        __HAL_RCC_USART1_CLK_ENABLE();
        /* GPIOA时钟使能 */
        __HAL_RCC_GPIOA_CLK_ENABLE();
        
        //TX端
        GPIO_InitStruct.Pin = GPIO_PIN_9;                  /* PA9 */
        GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;            /* 推挽式复用输出 */
        GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;      /* 高速 */
        HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);            /* 初始化串口1的TX引脚 */
        
        //RX端
        GPIO_InitStruct.Pin = GPIO_PIN_10;                 /* PA10 */
        GPIO_InitStruct.Mode = GPIO_MODE_AF_INPUT;         /* 复用输入 */
        GPIO_InitStruct.Pull = GPIO_PULLUP;                /* TX线开始默认发送高电平所以上拉 */
        HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);            /* 初始化串口1的RX引脚 */
        
        HAL_NVIC_SetPriority(USART1_IRQn, 2, 0);           /* 抢占优先级2|子优先级0 */
        HAL_NVIC_EnableIRQ(USART1_IRQn);                   /* 使能USART1中断通道 */
    }
}
```



##### ⑤串口1中断服务函数与串口数据接收完成回调函数的编写

```c
/* 串口1中断服务函数 */
void USART1_IRQHandler(void)
{
    HAL_UART_IRQHandler(&huart1);
    HAL_UART_Receive_IT(&huart1, (uint8_t*)g_rx_buffer, 1);
}

/* 串口数据接收完成回调函数 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if(huart->Instance == USART1)
    {
        g_usart1_rx_flag = 1;
    }
    
}
```



#### 2.2==uasrt.h==的编写

```c
#ifndef __USART_H
#define __USART_H

#include "main.h"

extern UART_HandleTypeDef huart1;               /* HAL UART句柄 */
extern uint8_t g_rx_buffer[1];                  /* HAL库使用的串口接收数据缓冲区 */
extern uint8_t g_usart1_rx_flag;                /* 串口接收到数据标志 */

void USART_Init(uint32_t baudrate);                /* 串口初始化函数 */

#endif

```



### 3.主函数==main.c==里的调用

##### 3.1中断法

```c
#include "main.h"
#include "stdio.h"
#include "string.h"

int main(void)
{
    HAL_Init();
    Stm32_Clock_Init();                         /* 配置时钟树，必须配置，因为USART1挂载在APB2总线时钟上 */
    LED_Init();
    KEY_Init();
    delay_init();
    EXTI_Init();
    USART_Init(115200);                         /* 波特率设为115200 */
    HAL_UART_Receive_IT(&huart1, (uint8_t*)g_rx_buffer, 1);//开启串口接受中断进入
    printf("请输入一个英文字符:\r\n\r\n");
    while(1)
    {
        if(g_usart1_rx_flag == 1)
        {
            printf("您输入的字符为:\r\n");
            HAL_UART_Transmit(&huart1, (uint8_t*)g_rx_buffer, 1, 1000);
            printf("\r\n");
            g_usart1_rx_flag = 0;
        }
        else
        {
            delay_ms(10);
        }
    }
}

```

由于重写printf函数后打印中文会有warning，所以我们在魔法棒里需要

![image-20231203144532890](https://s2.loli.net/2023/12/03/5yGlHUDeTtPiY9w.png)

添加

```c
-Wno-invalid-source-encoding
```



同时因为我们使用了重定义的printf函数，所以我们需要勾选

![image-20231203145220875](https://s2.loli.net/2023/12/03/gcJDiLaA7ZH4ztU.png)



若勾选此选项出现以下报错

```c
1.Undefined symbol __use_two_region_memory

2.Undefined symbol __initial_sp
```

在startup_stm32f103xb.s文件里把相应的那两项给注释然后编译，在取消注释在编译就可以了。

![image-20231203150350554](https://s2.loli.net/2023/12/03/IVBqDQuHdpl13e8.png)

若我们没有勾选Use MicroLIB则需要这样写

```c
#if 1
#include <stdio.h>
 
/* 告知连接器不从C库链接使用半主机的函数 */
#pragma import(__use_no_semihosting)
 
/* 定义 _sys_exit() 以避免使用半主机模式 */
void _sys_exit(int x)
{
    x = x;
}
 
/* 标准库需要的支持类型 */
struct __FILE
{
    int handle;
};
 
FILE __stdout;

/* 重写printf函数 */
/*方便我直接使用printf来打印数据*/
int fputc(int ch, FILE *stream)
{
    (void)stream;
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xffff);
    return 1;
}

/* 重写getchar/scanf函数 */
/* 还是用getchar吧，scnaf有bug，不知道为什么需要点击两次才能发送数据 */
int fgetc(FILE *stream)
{
    (void)stream;
    uint8_t ch = 0;
    HAL_UART_Receive(&huart1, &ch, 1, 0xffff);
    return ch;
}

#endif

```

强调不使用半主机（no semihosting）模式



拓展知识：

半主机模式解释：

**一、什么是半主机模式？**
简单的说，就是我们嵌入式程序中，类似printf的接口是会与我们PC进行通讯，以方便我们借助我们调试板的仿真器在PC上用开发工具进行调试。

**二、为什么要禁用半主机模式？**
在嵌入式的编程中你是避免不了使用printf、fopen、fclose等函数的但是因为嵌入式的程序中并没有对这些函数的底层实现，使得设备运行时会进入软件中断BAEB处，这时就需要__use_no_semihosting_swi这 个声明，使程序遇到这些文件操作函数时不停在此中断处。

MDK上开启半主机模式-需要SWO线（换言之，需要使用JTAG接线），而我们程序模式开启的半主机模式，所以，我们需要禁止半主机模式。当目标板脱离仿真器（jlink/ulink）单独运行时，不能使用半主机模式。否则进入软件中断BAEB处，无法再执行下去。

**三、如何禁止半主机模式？**

> pragma import(__use_no_semihosting_swi)


这条语句可以关闭半主机模式，只需要在任意一个C文件中加入即可。

> 还有在使用keil编程的过程中还会遇到..\OBJ\USART.axf: Error: L6915E: Library reports error: __use_no_semihosting was requested, but _ttywrch was referenced


说的大概的意思就是关掉了半主机模式，但是函数__ttywrch被要求了，这时要把函数重写一遍，当然出现其他的函数被要求的时候，可以参考上面的函数进行编写，只要放到任意一个.c源文件之中即可。



##### 3.2查询法

```c
#include "main.h"
#include "stdio.h"
#include "string.h"

int main(void)
{
    HAL_Init();
    Stm32_Clock_Init();                         /* 配置时钟树 */
    LED_Init();
    KEY_Init();
    delay_init();
    EXTI_Init();
    USART_Init(115200);                         /* 波特率设为115200 */
    
    char buffer[3];
    int i = 0;
    printf("请输入一个字符串:\r\n\r\n");
    while(1)
    {
        //fflush(stdin);//清空数据缓存区
        if(i == 3)
        {
            printf("%s\r\n", buffer);
            memset(buffer, 0x00, sizeof(buffer));
            i = 0;
        }
        while(i<3)
        {
            buffer[i] = getchar();
            i++;
        }
    }
}

```



##### 3.3main.h的编写

```c
#ifndef __MAIN_H
#define __MAIN_H


#include "stm32f1xx_hal.h"
#include "stdio.h"
#include "delay.h"
#include "sys.h"
#include "usart.h"

#endif

```

