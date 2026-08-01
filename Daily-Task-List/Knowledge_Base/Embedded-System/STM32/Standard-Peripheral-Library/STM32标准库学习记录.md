# STM32标准库学习记录



## *一.标准库建工程*

[手把手教你STM32入门教程（标准库）_stm32教程-CSDN博客](https://blog.csdn.net/yunsheng233/article/details/131403745)



## *二.常见问题汇总*

### 问题一：

如果使用V5编译器不能编译，则代表没有V5编译器，下载该文件夹

![image-20240410201321119](https://s2.loli.net/2024/04/10/7aIyuUlbdzCkocD.png)

然后：

![image-20240410201553917](https://s2.loli.net/2024/04/10/exoZwb4jBD21M75.png)

添加该文件夹就行了。

![image-20240410201739884](https://s2.loli.net/2024/04/10/yuo5TYbfnhx7qwZ.png)

添加好以后，就会多一个这个。



### 问题二：

如果错误原因是找不到文件，则解决方案如下：

点击魔法棒，然后

![image-20240410202258473](https://s2.loli.net/2024/04/10/zH7UEpXCjKV5cGh.png)

接着：

![image-20240410202735916](https://s2.loli.net/2024/04/10/X4fuUZJsegPDv1t.png)

以后每多一个功能实现的`.c`文件和`.h`文件，都存放在自己创建的文件夹里面，然后在魔法里面添加`.c`文件。



## 三.什么是GPIO？

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



## 四.关键结构体简介

```c
typedef struct
{
  uint16_t GPIO_Pin;				/* 引脚号 */ 

  GPIOSpeed_TypeDef GPIO_Speed; 	/* 速度设置 */

  GPIOMode_TypeDef GPIO_Mode;		/* 模式设置 */
}GPIO_InitTypeDef;
```



### **①Pin：**

```c
#define GPIO_Pin_0                 ((uint16_t)0x0001)  /*!< Pin 0 selected */
#define GPIO_Pin_1                 ((uint16_t)0x0002)  /*!< Pin 1 selected */
#define GPIO_Pin_2                 ((uint16_t)0x0004)  /*!< Pin 2 selected */
#define GPIO_Pin_3                 ((uint16_t)0x0008)  /*!< Pin 3 selected */
#define GPIO_Pin_4                 ((uint16_t)0x0010)  /*!< Pin 4 selected */
#define GPIO_Pin_5                 ((uint16_t)0x0020)  /*!< Pin 5 selected */
#define GPIO_Pin_6                 ((uint16_t)0x0040)  /*!< Pin 6 selected */
#define GPIO_Pin_7                 ((uint16_t)0x0080)  /*!< Pin 7 selected */
#define GPIO_Pin_8                 ((uint16_t)0x0100)  /*!< Pin 8 selected */
#define GPIO_Pin_9                 ((uint16_t)0x0200)  /*!< Pin 9 selected */
#define GPIO_Pin_10                ((uint16_t)0x0400)  /*!< Pin 10 selected */
#define GPIO_Pin_11                ((uint16_t)0x0800)  /*!< Pin 11 selected */
#define GPIO_Pin_12                ((uint16_t)0x1000)  /*!< Pin 12 selected */
#define GPIO_Pin_13                ((uint16_t)0x2000)  /*!< Pin 13 selected */
#define GPIO_Pin_14                ((uint16_t)0x4000)  /*!< Pin 14 selected */
#define GPIO_Pin_15                ((uint16_t)0x8000)  /*!< Pin 15 selected */
#define GPIO_Pin_All               ((uint16_t)0xFFFF)  /*!< All pins selected */
```

**成员 Pin 表示引脚号，范围：GPIO_PIN_0 到 GPIO_PIN_15。**



### **②Mode：**

```c
typedef enum
{ GPIO_Mode_AIN = 0x0,                  //模拟输入
  GPIO_Mode_IN_FLOATING = 0x04,         //浮空输入
  GPIO_Mode_IPD = 0x28,                 //下拉输入
  GPIO_Mode_IPU = 0x48,                 //上拉输入
  GPIO_Mode_Out_OD = 0x14,              //开漏输出
  GPIO_Mode_Out_PP = 0x10,              //推挽输出
  GPIO_Mode_AF_OD = 0x1C,               //复用开漏输出
  GPIO_Mode_AF_PP = 0x18                //复用推挽输出
}GPIOMode_TypeDef;
```

**成员 Mode 是 GPIO 的模式选择，有以上选择项：**



### **③Speed：**

**成员 Speed 用于配置 GPIO 的速度**

```c
typedef enum
{ 
  GPIO_Speed_10MHz = 1,		/* 低速 */
  GPIO_Speed_2MHz, 			/* 中速 */
  GPIO_Speed_50MHz			/* 高速 */
}GPIOSpeed_TypeDef;
```



## 五.常用的GPIO函数

```c
void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct)
```

GPIO结构体初始化函数：

第一个参数是端口号；

第二个参数是我们创建的GPIO结构体。



```c
void GPIO_ResetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
```

GPIO引脚置低函数：

第一个参数是端口号；

第二个参数是引脚号。



```c
void GPIO_SetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
```

GPIO引脚置高函数：

第一个参数是端口号；

第二个参数是引脚号。



```c
void GPIO_WriteBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, BitAction BitVal)
```

GPIO引脚写入函数：

第一个参数是端口号；

第二个参数是引脚号；

第三个参数是引脚电平，可填以下两个参数：

```c
Bit_RESET //等价于0
Bit_SET   //等价于1
```



```c
uint8_t GPIO_ReadInputDataBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
```

GPIO引脚读入函数：

第一个参数是端口号；

第二个参数是引脚号；

返回值为读取的引脚电平：`0或1`（一般此函数用在按键输入，按下为低电平0，否则没有按下为高电平1）



## 六.LED的使用

首先添加我们存放用户自己编写的`.c`和`.h`文件的Basic文件夹的路径（老师命名的BSP）

![image-20240416174352230](https://s2.loli.net/2024/04/16/9DuLInyBKZ8mvl1.png)

然后添加我们编写的`.c`文件，每编写一个文件，就要在魔方里对应的文件夹里面添加一次对应的`.c`文件

![image-20240416174831829](https://s2.loli.net/2024/04/16/nQZMIwO6lApmh8D.png)



### （1）led.c的编写

```c
#include "led.h"//包含对应的.h文件

/* LED初始化(函数名字随意) */

//我们这里假设LED灯的端口为GPIOA，引脚为0，1，2，3
void LED_Config(void)
{
    //定义GPIO结构体
    GPIO_InitTypeDef GPIO_InitStructure;
    //使能挂载在APB2上的GPIOA时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    //结构体参数初始化
    /* LED的模式选择推挽输出(可高可低) */
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0|GPIO_Pin_1|GPIO_Pin_2|GPIO_Pin_3;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    //初始化GPIO结构体
    GPIO_Init(GPIOA, &GPIO_InitStructure);
     
    //初始化LED灯全灭
    GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);
    GPIO_WriteBit(GPIOA, GPIO_Pin_1, Bit_SET);
    GPIO_WriteBit(GPIOA, GPIO_Pin_2, Bit_SET);
    GPIO_WriteBit(GPIOA, GPIO_Pin_3, Bit_SET);
}

/* 下面的注释代表GPIO结构体可以用指针 */

//void LED_Config(void)
//{
//    //定义GPIO结构体
//    GPIO_InitTypeDef *GPIO_InitStructure;
//    //使能挂载在APB2上的GPIOA时钟
//    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
//    
//    //结构体参数初始化
//    /*  */
//    GPIO_InitStructure->GPIO_Mode = GPIO_Mode_Out_PP;
//    GPIO_InitStructure->GPIO_Pin = GPIO_Pin_0|GPIO_Pin_1|GPIO_Pin_2|GPIO_Pin_3;
//    GPIO_InitStructure->GPIO_Speed = GPIO_Speed_50MHz;
//    //初始化GPIO结构体
//    GPIO_Init(GPIOA, GPIO_InitStructure);
//     
//    //初始化LED灯全灭
//    GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);
//    GPIO_WriteBit(GPIOA, GPIO_Pin_1, Bit_SET);
//    GPIO_WriteBit(GPIOA, GPIO_Pin_2, Bit_SET);
//    GPIO_WriteBit(GPIOA, GPIO_Pin_3, Bit_SET);
//}

//参数1：选择LED灯
//参数2：LED灯的状态，0为亮，1为灭
//void LED_Show(uint8_t led, uint8_t state)
//{
//    //左移运算符操作LED灯，实质是操作寄存器的值
//    GPIO_WriteBit(GPIOA, GPIO_Pin_0<<(led-1), state);
//}

```



### （2）led.h的编写

```c
/*如下为LED驱动的头文件*/
#ifndef __LED_H_ 
//防重复引用，如果没有定义过_LED_H_，则编译下句
#define __LED_H_ 
//此符号唯一， 表示只要引用过一次，即我们的#include，则定义符号_LED_H_/

#include "main.h"

//为了简化对LED的操作，我们可以使用define简化我们的操作
#define LED1_ON GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_RESET);
#define LED2_ON GPIO_WriteBit(GPIOA, GPIO_Pin_1, Bit_RESET);
#define LED3_ON GPIO_WriteBit(GPIOA, GPIO_Pin_2, Bit_RESET);
#define LED4_ON GPIO_WriteBit(GPIOA, GPIO_Pin_3, Bit_RESET);

#define LED1_OFF GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);
#define LED2_OFF GPIO_WriteBit(GPIOA, GPIO_Pin_1, Bit_SET);
#define LED3_OFF GPIO_WriteBit(GPIOA, GPIO_Pin_2, Bit_SET);
#define LED4_OFF GPIO_WriteBit(GPIOA, GPIO_Pin_3, Bit_SET);
void LED_Config(void);

#endif

//在#endif一定要空一行否则会有Warning:
//warning: no newline at end of file [-Wnewline-eof] #endif  /* __LED_H */
```



### （3）main.c的编写

使用延时函数需要在魔方里的Basic文件夹里面添加`delay.c`，同时需要把`delay.c`和`delay.h`放在Basic（老师命名的BSP）文件夹里面

#### `delay.c`

```c
#include "delay.h"
////////////////////////////////////////////////////////////////////////////////// 	 
//如果需要使用OS,则包括下面的头文件即可.
#if SYSTEM_SUPPORT_OS
#include "includes.h"					//ucos 使用	  
#endif
//////////////////////////////////////////////////////////////////////////////////	 
//本程序只供学习使用，未经作者许可，不得用于其它任何用途
//ALIENTEK STM32开发板
//使用SysTick的普通计数模式对延迟进行管理（适合STM32F10x系列）
//包括delay_us,delay_ms
//正点原子@ALIENTEK
//技术论坛:www.openedv.com
//创建日期:2010/1/1
//版本：V1.8
//版权所有，盗版必究。
//Copyright(C) 广州市星翼电子科技有限公司 2009-2019
//All rights reserved
//********************************************************************************
//V1.2修改说明
//修正了中断中调用出现死循环的错误
//防止延时不准确,采用do while结构!
//V1.3修改说明
//增加了对UCOSII延时的支持.
//如果使用ucosII,delay_init会自动设置SYSTICK的值,使之与ucos的TICKS_PER_SEC对应.
//delay_ms和delay_us也进行了针对ucos的改造.
//delay_us可以在ucos下使用,而且准确度很高,更重要的是没有占用额外的定时器.
//delay_ms在ucos下,可以当成OSTimeDly来用,在未启动ucos时,它采用delay_us实现,从而准确延时
//可以用来初始化外设,在启动了ucos之后delay_ms根据延时的长短,选择OSTimeDly实现或者delay_us实现.
//V1.4修改说明 20110929
//修改了使用ucos,但是ucos未启动的时候,delay_ms中中断无法响应的bug.
//V1.5修改说明 20120902
//在delay_us加入ucos上锁，防止由于ucos打断delay_us的执行，可能导致的延时不准。
//V1.6修改说明 20150109
//在delay_ms加入OSLockNesting判断。
//V1.7修改说明 20150319
//修改OS支持方式,以支持任意OS(不限于UCOSII和UCOSIII,理论上任意OS都可以支持)
//添加:delay_osrunning/delay_ostickspersec/delay_osintnesting三个宏定义
//添加:delay_osschedlock/delay_osschedunlock/delay_ostimedly三个函数
//V1.8修改说明 20150519
//修正UCOSIII支持时的2个bug：
//delay_tickspersec改为：delay_ostickspersec
//delay_intnesting改为：delay_osintnesting
//////////////////////////////////////////////////////////////////////////////////  

static u8  fac_us=0;							//us延时倍乘数			   
static u16 fac_ms=0;							//ms延时倍乘数,在ucos下,代表每个节拍的ms数
	
	
#if SYSTEM_SUPPORT_OS							//如果SYSTEM_SUPPORT_OS定义了,说明要支持OS了(不限于UCOS).
//当delay_us/delay_ms需要支持OS的时候需要三个与OS相关的宏定义和函数来支持
//首先是3个宏定义:
//    delay_osrunning:用于表示OS当前是否正在运行,以决定是否可以使用相关函数
//delay_ostickspersec:用于表示OS设定的时钟节拍,delay_init将根据这个参数来初始哈systick
// delay_osintnesting:用于表示OS中断嵌套级别,因为中断里面不可以调度,delay_ms使用该参数来决定如何运行
//然后是3个函数:
//  delay_osschedlock:用于锁定OS任务调度,禁止调度
//delay_osschedunlock:用于解锁OS任务调度,重新开启调度
//    delay_ostimedly:用于OS延时,可以引起任务调度.

//本例程仅作UCOSII和UCOSIII的支持,其他OS,请自行参考着移植
//支持UCOSII
#ifdef 	OS_CRITICAL_METHOD						//OS_CRITICAL_METHOD定义了,说明要支持UCOSII				
#define delay_osrunning		OSRunning			//OS是否运行标记,0,不运行;1,在运行
#define delay_ostickspersec	OS_TICKS_PER_SEC	//OS时钟节拍,即每秒调度次数
#define delay_osintnesting 	OSIntNesting		//中断嵌套级别,即中断嵌套次数
#endif

//支持UCOSIII
#ifdef 	CPU_CFG_CRITICAL_METHOD					//CPU_CFG_CRITICAL_METHOD定义了,说明要支持UCOSIII	
#define delay_osrunning		OSRunning			//OS是否运行标记,0,不运行;1,在运行
#define delay_ostickspersec	OSCfg_TickRate_Hz	//OS时钟节拍,即每秒调度次数
#define delay_osintnesting 	OSIntNestingCtr		//中断嵌套级别,即中断嵌套次数
#endif


//us级延时时,关闭任务调度(防止打断us级延迟)
void delay_osschedlock(void)
{
#ifdef CPU_CFG_CRITICAL_METHOD   				//使用UCOSIII
	OS_ERR err; 
	OSSchedLock(&err);							//UCOSIII的方式,禁止调度，防止打断us延时
#else											//否则UCOSII
	OSSchedLock();								//UCOSII的方式,禁止调度，防止打断us延时
#endif
}

//us级延时时,恢复任务调度
void delay_osschedunlock(void)
{	
#ifdef CPU_CFG_CRITICAL_METHOD   				//使用UCOSIII
	OS_ERR err; 
	OSSchedUnlock(&err);						//UCOSIII的方式,恢复调度
#else											//否则UCOSII
	OSSchedUnlock();							//UCOSII的方式,恢复调度
#endif
}

//调用OS自带的延时函数延时
//ticks:延时的节拍数
void delay_ostimedly(u32 ticks)
{
#ifdef CPU_CFG_CRITICAL_METHOD
	OS_ERR err; 
	OSTimeDly(ticks,OS_OPT_TIME_PERIODIC,&err);	//UCOSIII延时采用周期模式
#else
	OSTimeDly(ticks);							//UCOSII延时
#endif 
}
 
//systick中断服务函数,使用ucos时用到
void SysTick_Handler(void)
{	
	if(delay_osrunning==1)						//OS开始跑了,才执行正常的调度处理
	{
		OSIntEnter();							//进入中断
		OSTimeTick();       					//调用ucos的时钟服务程序               
		OSIntExit();       	 					//触发任务切换软中断
	}
}
#endif

			   
//初始化延迟函数
//当使用OS的时候,此函数会初始化OS的时钟节拍
//SYSTICK的时钟固定为HCLK时钟的1/8
//SYSCLK:系统时钟
void delay_init()
{
#if SYSTEM_SUPPORT_OS  							//如果需要支持OS.
	u32 reload;
#endif
	SysTick_CLKSourceConfig(SysTick_CLKSource_HCLK_Div8);	//选择外部时钟  HCLK/8
	fac_us=SystemCoreClock/8000000;				//为系统时钟的1/8  
#if SYSTEM_SUPPORT_OS  							//如果需要支持OS.
	reload=SystemCoreClock/8000000;				//每秒钟的计数次数 单位为K	   
	reload*=1000000/delay_ostickspersec;		//根据delay_ostickspersec设定溢出时间
												//reload为24位寄存器,最大值:16777216,在72M下,约合1.86s左右	
	fac_ms=1000/delay_ostickspersec;			//代表OS可以延时的最少单位	   

	SysTick->CTRL|=SysTick_CTRL_TICKINT_Msk;   	//开启SYSTICK中断
	SysTick->LOAD=reload; 						//每1/delay_ostickspersec秒中断一次	
	SysTick->CTRL|=SysTick_CTRL_ENABLE_Msk;   	//开启SYSTICK    

#else
	fac_ms=(u16)fac_us*1000;					//非OS下,代表每个ms需要的systick时钟数   
#endif
}								    

#if SYSTEM_SUPPORT_OS  							//如果需要支持OS.
//延时nus
//nus为要延时的us数.		    								   
void delay_us(u32 nus)
{		
	u32 ticks;
	u32 told,tnow,tcnt=0;
	u32 reload=SysTick->LOAD;					//LOAD的值	    	 
	ticks=nus*fac_us; 							//需要的节拍数	  		 
	tcnt=0;
	delay_osschedlock();						//阻止OS调度，防止打断us延时
	told=SysTick->VAL;        					//刚进入时的计数器值
	while(1)
	{
		tnow=SysTick->VAL;	
		if(tnow!=told)
		{	    
			if(tnow<told)tcnt+=told-tnow;		//这里注意一下SYSTICK是一个递减的计数器就可以了.
			else tcnt+=reload-tnow+told;	    
			told=tnow;
			if(tcnt>=ticks)break;				//时间超过/等于要延迟的时间,则退出.
		}  
	};
	delay_osschedunlock();						//恢复OS调度									    
}
//延时nms
//nms:要延时的ms数
void delay_ms(u16 nms)
{	
	if(delay_osrunning&&delay_osintnesting==0)	//如果OS已经在跑了,并且不是在中断里面(中断里面不能任务调度)	    
	{		 
		if(nms>=fac_ms)							//延时的时间大于OS的最少时间周期 
		{ 
   			delay_ostimedly(nms/fac_ms);		//OS延时
		}
		nms%=fac_ms;							//OS已经无法提供这么小的延时了,采用普通方式延时    
	}
	delay_us((u32)(nms*1000));					//普通方式延时  
}
#else //不用OS时
//延时nus
//nus为要延时的us数.		    								   
void delay_us(u32 nus)
{		
	u32 temp;	    	 
	SysTick->LOAD=nus*fac_us; 					//时间加载	  		 
	SysTick->VAL=0x00;        					//清空计数器
	SysTick->CTRL|=SysTick_CTRL_ENABLE_Msk ;	//开始倒数	  
	do
	{
		temp=SysTick->CTRL;
	}while((temp&0x01)&&!(temp&(1<<16)));		//等待时间到达   
	SysTick->CTRL&=~SysTick_CTRL_ENABLE_Msk;	//关闭计数器
	SysTick->VAL =0X00;      					 //清空计数器	 
}
//延时nms
//注意nms的范围
//SysTick->LOAD为24位寄存器,所以,最大延时为:
//nms<=0xffffff*8*1000/SYSCLK
//SYSCLK单位为Hz,nms单位为ms
//对72M条件下,nms<=1864 
void delay_ms(u16 nms)
{	 		  	  
	u32 temp;		   
	SysTick->LOAD=(u32)nms*fac_ms;				//时间加载(SysTick->LOAD为24bit)
	SysTick->VAL =0x00;							//清空计数器
	SysTick->CTRL|=SysTick_CTRL_ENABLE_Msk ;	//开始倒数  
	do
	{
		temp=SysTick->CTRL;
	}while((temp&0x01)&&!(temp&(1<<16)));		//等待时间到达   
	SysTick->CTRL&=~SysTick_CTRL_ENABLE_Msk;	//关闭计数器
	SysTick->VAL =0X00;       					//清空计数器	  	    
} 
#endif 

```

#### `delay.h`

```c
#ifndef __DELAY_H
#define __DELAY_H 
#include "stm32f10x.h"
	 
void delay_init(void);
void delay_ms(u16 nms);
void delay_us(u32 nus);

#endif
```

#### `main.c`

```c
#include "main.h"

int main(void)
{
    /* LED初始化 */
    LED_Config();
    /* 延时函数初始化 */
    delay_init();
    while(1)
    {
        /* LED灯闪烁方法一： */
//        GPIO_SetBits(GPIOA, GPIO_Pin_0);
//        delay_ms(500);
//        GPIO_ResetBits(GPIOA, GPIO_Pin_0);
//        delay_ms(500);
        
        /* LED灯闪烁方法二： */
//        GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_RESET);
//        ddelay_ms(500);
//        GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);
//        delay_ms(500);
        
        /* LED灯闪烁方法三： */
        LED1_ON;
        delay_ms(500);
        LED1_OFF;
        delay_ms(500);
        
    }
}
```



### （4）main.h的编写

```c
#ifndef __MAIN_H
#define __MAIN_H

#include "stm32f10x.h"                  // Device header
#include "delay.h"

//每写一个.c和.h文件，就需要在main.h里面声明一下
#include "led.h"

#endif
```



## 七.KEY的使用

### （1）key.c的编写

```c
#include "key.h"

void KEY_Config(void)
{
    GPIO_InitTypeDef KEY;
    //因为假设是GPIOA的4，5，6，7，所以打开GPIOA时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    //设置4个引脚
    KEY.GPIO_Pin = KEY1_Pin|KEY2_Pin|KEY3_Pin|KEY4_Pin;
    //上拉输入，按下为低电平，未按下为高电平
    KEY.GPIO_Mode = GPIO_Mode_IPU;
    //可以设置也可以不设置
    KEY.GPIO_Speed = GPIO_Speed_50MHz;
    //初始化GPIO结构体
    GPIO_Init(GPIOA, &KEY);
}

u8 led1_state =1 ;
/* u8 led2_state = 1;
......
*/
void KEY_Scan()
{
    if(KEY1 == 0)
    {
        delay_ms(20);
        while(KEY1 = 0);
        /* 法一： */
//        led1_state++;
//        led1_state %= 2;
//        GPIO_WriteBit(GPIOA, GPIO_Pin_0, led1_state);
        /* 法二： */
        if(led1_state == 1)
        {
            LED1_ON;
            led1_state = 0;
        }
        else
        {
            LED1_OFF;
            led1_state = 1;
        }
    }
    /*
    if(KEY2 == 0)
    {
        delay_ms(20);
        while(KEY2 = 0);
        //法一
//        led2_state++;
//        led2_state %= 2;
//        GPIO_WriteBit(GPIOA, GPIO_Pin_1, led2_state);
        //法二：
        if(led2_state == 1)
        {
            LED2_ON;
            led2_state = 0;
        }
        else
        {
            LED2_OFF;
            led2_state = 1;
        }
    }
    ......
    */
}
```



### （2）key.h的编写

```c
#ifndef __KEY_H
#define __KEY_H

//假设按键为GPIOA的4，5，6，7
#define KEY1_Port GPIOA
#define KEY2_Port GPIOA
#define KEY3_Port GPIOA
#define KEY4_Port GPIOA

#define KEY1_Pin GPIO_Pin_4
#define KEY2_Pin GPIO_Pin_5
#define KEY3_Pin GPIO_Pin_6
#define KEY4_Pin GPIO_Pin_7

#define KEY1 GPIO_ReadInputDataBit(KEY1_Port, KEY1_Pin)
#define KEY2 GPIO_ReadInputDataBit(KEY2_Port, KEY2_Pin)
#define KEY3 GPIO_ReadInputDataBit(KEY3_Port, KEY3_Pin)
#define KEY4 GPIO_ReadInputDataBit(KEY4_Port, KEY4_Pin)

void KEY_Config(void);
void KEY_Scan();

#endif
```



### （3）main.c的编写

```c
#include "main.h"

int main(void)
{
    //LED初始化
    LED_Config();
    //KEY初始化
    KEY_Config();
    while(1)
    {
        KEY_Scan();
    }
}
```



### （4）main.h的编写

```c
#ifndef __MAIN_H
#define __MAIN_H

#include "stm32f10x.h"                  // Device header
#include "delay.h"

//每写一个.c和.h文件，就需要在main.h里面声明一下
#include "led.h"
#include "key.h"

#endif
```



## 八.外部中断

### 1.什么是中断？

![什么是中断](https://s2.loli.net/2023/11/10/vnpAfsuMgcjlQzr.png)

简单理解的话，这就是中断。

以下是对于中断的详细介绍：



![什么是中断](https://img-blog.csdnimg.cn/img_convert/daf264a3945cfadb190e1e062b4afa68.png)



### 2.**中断的作用和意义**

作用：

1.实时控制：在确定时间内对相应事件作出响应，如：温度监控。

2.故障处理：检测到故障，需要第一时间处理，如：电梯门夹人了。

3.数据传输：不确定数据何时会来，如：串口数据接收。

意义：

中断的意义：**高效**处理紧急程序，不会一直占用CPU资源。



