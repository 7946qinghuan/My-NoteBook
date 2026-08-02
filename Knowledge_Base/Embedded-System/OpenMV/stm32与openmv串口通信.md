---
title: stm32与openmv串口通信
date: 2026-08-01
tags: [Embedded, OpenMV]
aliases: []
---

# STM32与OpenMV串口通信



## 一.OPenMV上的串口

### UART类 – 双向串行通信总线

UART执行标准UART/USART双向串行通信协议。其物理层包括两条线：RX和TX。通信单元为8位或9位宽的字符（勿与字符串字符混淆）。

UART对象可通过下列方式创建和初始化:

```python
from pyb import UART

uart = UART(3, 9600, timeout_char=1000)                         # i使用给定波特率初始化
uart.init(9600, bits=8, parity=None, stop=1, timeout_char=1000) # 使用给定参数初始化
```

波特率按照自己的需求设置

数据位可为7、8、9。

奇偶校验位可为None、0（偶）、1（奇）。

停止位可为1或2。

>*注意:* 奇偶性为None时，仅支持位数为8和9。启用奇偶性时，仅支持位数为7和8。

方法详解：

```
UART.init(baudrate, bits=8, parity=None, stop=1, \*, timeout=1000, flow=0, timeout_char=0, read_buf_len=64)
```

使用给定参数初始化UART总线:

> - `baudrate` 为时钟频率。
> - `bits` 为每个字符的位数，7、8或9。
> - `parity` 为奇偶校验， `None` ，0（偶）或1（奇）。
> - `stop` 为停止位的数量，1或2
> - `flow` 设置流控制类型。可为0、 `UART.RTS`, `UART.CTS` 或 `UART.RTS | UART.CTS`.
> - `timeout` 为等待读取/写入首个字符的超时时长（以毫秒为单位）。
> - `timeout_char` 为读取或写入时字符间等待的超时时长（以毫秒为单位）。
> - `read_buf_len` 为读取缓冲区的字符长度（0为禁用）。

























































































### usart.c

```c
/* USER CODE BEGIN 1 */
/*重写printf函数*/
/*方便我直接使用printf来打印数据*/ 
int fputc(int ch,FILE *stream)
{
	HAL_UART_Transmit(&huart1,(uint8_t *)&ch,1,0xffff);
	return 1; 
}

void Split_Str(char *split_str[], char str[], const char* ch)
{
	if (str == NULL) {				/*差分的字符串为空 直接return*/
		return;
	}
	
	int chSize = strlen(ch);		/*获取分隔字符串长度*/
	char *head = str;				/*头指针指向str第一个位置*/
	while (*head == *ch) { 			/*防止一开始就有分隔符出现*/
		head += chSize;
	}
	
	char *tail = head;				/*尾指针初始化指向头指针位置*/
	int cnt = 0; 					/*记录第几个字符串(数据)位置*/
	
	while (*tail != '\0') { 		/*只要尾指针不为'\0'(字符串没有结束)*/	
		if (*tail == *ch) { 		/*当前位置的字符为分隔字符串的第一个字符*/
			char temp = *tail;		/*保存当前尾指针所指向的字符*/
			*tail = '\0'; 			/*把当前尾指针所指向位置字符改成'\0' 表示结束*/
			split_str[cnt] = head; 	/*当前保存数据为头指针到分隔符位置字符*/
			cnt++;					/*数据位置后移1位*/
			*tail = temp;		 	/*把尾指针的'\0'恢复为原来的值*/
			head = tail + chSize;	/*更新头结点位置为分隔符下一个字符*/
			tail += chSize ;		/*跳过分隔字符串*/
		} else {					/*当前位置的字符不是分隔符*/
			tail++;					/*尾指针向后移动1*/
		}	
	}
	
	/*处理最后一个数据可以没有分隔符*/
	if (*tail == '\0') {		/*如果尾指针为'\0'*/
		split_str[cnt] = head;	/*当前头指针位置到'\0'*/
		return;	
	}
}
/* USER CODE END 1 */
这段代码是一个函数，名为Split_Str。该函数的功能是将一个字符串根据另一个字符串（分隔符）进行分割，分割后的每个子串（数据）存储到一个字符串指针数组中。下面是对代码中的各行进行的解释：

1.定义了名为Split_Str的函数，该函数接收三个参数，分别为指向字符串指针数组的指针、源字符串和分割字符。
2.如果要分割的字符串为空，则直接返回。
3.获取分割字符的长度。
4.头指针（初始化为源字符串的起始地址）向右移动，跳过源字符串开头的分割字符。
5.尾指针初始化为和头指针相同的位置，用于扫描源字符串。
6.cnt用于记录已经找到几个数据。
7.在循环中，只要尾指针所指位置不是字符串结束符’\0’，就会一直进行扫描。
8.如果当前尾指针所指位置符合分割字符的第一个字符，则意味着一个数据已经结束，尾指针所指位置的字符需要修改为字符串结束符’\0’。
9.通过指针数组split_str传递头指针和尾指针之间的字符数据。
10.将头指针指向下一个数据的起始位置，将尾指针跨过一个分割字符。
11.如果当前尾指针所指位置不符合分割字符的第一个字符，则只是尾指针向右移动了一个位置。
12.如果尾指针所指的位置是’\0’，则最后一个数据会被忽略，因此需要特别处理。
程序结束。
```

### usart.h

```c
/* USER CODE BEGIN Includes */
#include "stdio.h"
#include "string.h"
/* USER CODE END Includes */


/* USER CODE BEGIN Private defines */
#define MAX_SIZE 1024
#define DATANUM 4 /*拆分之后的数据个*/
/* USER CODE END Private defines */

/* USER CODE BEGIN Prototypes */
void MX_USART1_UART_Init(void);
void Split_Str(char *split_str[], char str[], const char* ch);

/*串口的结构体*/ 
typedef struct
{
	char data;			 //接收数据
	uint16_t cnt;		 //接受数据计数
	char buff[MAX_SIZE]; //接受数据的数组缓存
	uint8_t rx_Flag;	 //接收完毕标志位
} UART;

typedef struct {
	char str_buff[MAX_SIZE];/*用于拆分的原始数据*/
	char *show_Str[DATANUM];/*拆分后的数据*/
} Split_Str_Handl;

extern UART rx;//使其他文件(main.c)能够使用这个结构体
/* USER CODE END Prototypes */
```

### main.c

```c

/* USER CODE BEGIN PV */
UART rx;

static UART rx1 = {0, 0, 0, 0}; 			//串口结构体
static Split_Str_Handl my_Split = {0, 0};	//拆分字符创结构体
int x,y,length,a=0,w;
static int intData[DATANUM] = {0};
void Data_Proccess(void);
//uint8_t  CO2AskBuffer[1]={2};

/* USER CODE END PV */

/* USER CODE BEGIN 2 */
	HAL_UART_Receive_IT(&huart1, (uint8_t *)&rx1.data, 1); //开启接收中断
//HAL_UART_Receive_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size)
//功能：串口中断接收，以中断方式接收指定长度数据。
//大致过程是，设置数据存放位置，接收数据长度，然后使能串口接收中断。接收到数据时，会触发串口中断。
//再然后，串口中断函数处理，直到接收到指定长度数据，而后关闭中断，进入中断接收回调函数，不再触发接收中断。(只触发一次中断)
//UART_HandleTypeDef *huart      UATR的别名    如 :   UART_HandleTypeDef huart1;   别名就是huart1  
//*pData      接收到的数据存放地址
//Size    接收的字节数
/* USER CODE END 2 */


 while (1)
  {
		Data_Proccess();
  }
/* USER CODE BEGIN 4 */
void Data_Proccess(void)
{
	if (rx1.rx_Flag == 1) {
		/*把rx1.buff[0] ~ rx1.buff[Cnt - 2]的内容转存到str_buff*/
		
//		uint8_t index;
//		uint8_t y = 0;
		
		Split_Str(my_Split.show_Str, my_Split.str_buff, ",");
		OLED_Clear();
		
		intData[0] = atoi((const char*)my_Split.show_Str[0]);
		intData[1] = atoi((const char*)my_Split.show_Str[1]);
		intData[2] = atoi((const char*)my_Split.show_Str[2]);
		intData[3] = atoi((const char*)my_Split.show_Str[3]);
		
		/*OLED显示*/
	  x=intData[0];
		y=intData[1];
		length=intData[2];
		w=intData[3];
		
		OLED_ShowNum(0, 0, x, 4, 16);
		OLED_ShowNum(0, 2, intData[1], 4, 16);
		OLED_ShowNum(0, 4, intData[2], 4, 16);
		OLED_ShowNum(0, 6, intData[3], 4, 16);
		
		printf("%d %d %d %d\r\n",intData[0], intData[1],intData[2],intData[3]);
		/*清空数据 方便下次存储*/
		memset(my_Split.str_buff, 0x00, sizeof(my_Split.str_buff));
		/*处理完一帧数据接收标志位清零可以接收下一次数据*/
		rx1.rx_Flag = 0;
	}
}


/*串口回调函数*/
//HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart);  
//功能：HAL库的中断进行完之后，并不会直接退出，而是会进入中断回调函数中，用户可以在其中设置代码，串口中断接收完成之后，会进入该函数，该函数为空函数，用户需自行修改
//参数：UART_HandleTypeDef *huart      UATR的别名    如 :   UART_HandleTypeDef huart1;   别名就是huart1 
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	if (huart->Instance == USART1) //如果串口1接收到数据
	{
		rx1.buff[rx1.cnt++] = rx1.data; //把数据存储到接收数据缓存里
		//如果倒数第一个是'#'针尾
		if (rx1.buff[rx1.cnt - 1] == '#' && rx1.rx_Flag == 0)
		{	
			memmove(my_Split.str_buff, rx1.buff, rx1.cnt - 1);
			//打印接收到的内容
			printf("内容:%s\r\n", my_Split.str_buff);  
			rx1.rx_Flag = 1;
			/*把本次接收的数据清零 计数也清零*/
			rx1.cnt = 0;
			memset(rx1.buff, 0x00, strlen(rx1.buff));
		}
		HAL_UART_Receive_IT(&huart1, (uint8_t *)&rx1.data, 1); //更新串口接收中断
	}
}


/* USER CODE END 4 */

```

### main.h

```c
/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include "sys.h"
#include "delay.h"
#include "stdio.h"
#include "string.h"
#include "stdlib.h"
#include "OLED.h"
/* USER CODE END Includes */

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
- [[STM32-TIMER]]
- [[STM32-USART]]
- [[STM32-IIC]]
- [[TB6612]]
- [[STM32标准库学习记录]]
- [[PID算法学习记录]]

