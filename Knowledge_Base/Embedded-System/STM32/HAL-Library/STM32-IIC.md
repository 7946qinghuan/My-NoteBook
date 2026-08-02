---
title: STM32-IIC
date: 2026-08-01
tags: [Embedded, STM32, HAL]
aliases: []
---

# ==STM32-IIC==



## 一.IIC总线协议协议介绍

IIC：Inter Integrated Circuit，集成电路总线，是一种**同步** **串行** **半双工**通信总线。

对于IIC来说

`总线就是传输数据通道`

`协议就是传输数据的规则`



IIC一共有只有两个总线： 一条是双向的串行数据线SDA，一条是串行时钟线SCL。

SDA(Serial data)是数据线，D代表Data也就是数据，Send Data 也就是用来传输数据的

SCL(Serial clock line)是时钟线，C代表Clock 也就是时钟 也就是控制数据发送的时序的



## 二.IIC总线结构图

![image-20240301185551354](https://s2.loli.net/2024/03/01/fXKqrd8ocghbC5y.png)

```
1.由时钟线SCL和数据线SDA组成，并且都接上拉电阻，确保总线空闲状态为高电平
```

```
2.总线支持多设备连接，允许多主机存在，每个设备都有一个唯一的地址
```

```
3.连接到总线上的数目受总线的最大电容400pf限制
```

```
4.数据传输速率：标准模式100k bit/s  快速模式400k bit/s 高速模式3.4Mbit/s
```



## 三.IIC的协议层

I2C 总线在传送数据过程中共有三种类型信号， 它们分别是：**开始信号**、**结束信号**和**应答信号**。



**1**.开始信号：SCL 为高电平时，SDA 由高电平向低电平跳变，开始传送数据。



**2**.结束信号：SCL 为高电平时，SDA 由低电平向高电平跳变，结束传送数据。



**3**.应答信号：接收数据的 IC 在接收到 8bit 数据后，向发送数据的 IC 发出特定的低电平脉冲，表示已收到数据。CPU 向受控单元发出一个信号后，等待受控单元发出一个应答信号，CPU 接收到应答信号后，根据实际情况作出是否继续传递信号的判断。若未收到应答信号，由判断为受控单元出现故障。



**4**.这些信号中，起始信号是必需的，结束信号和应答信号，都可以不要。



## 四.IIC协议时序

![img](https://img-blog.csdn.net/20180514184751564)

### 1.初始(空闲)状态：

因为IIC的SCL 和SDA都需要接上拉电阻，**保证空闲状态的稳定性**

**所以IIC总线在空闲状态下SCL 和SDA都保持高电平**

```c
void IIC_init()       //IIC初始化
{
       IIC_SCL = 1; //首先把时钟线拉高
       delay_us(4);//延时函数
       IIC_SDA = 1; //在SCL为高的情况下把SDA拉高
       delay_us(4); //延时函数
}
```



### 2.开始信号：

**SCL保持高电平，SDA由高电平变为低电平后，延时(>4.7us)，SCL变为低电平。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200411104124610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzNDgwMTMzOTM3,size_16,color_FFFFFF,t_70)

```c
//产生IIC起始信号
//1.先拉高SDA，再拉高SCL，空闲状态
//2.拉低SDA
void IIC_Start(void)         //启动信号
{
       IIC_SDA = 1; //确保SDA线为高电平
       delay_us(5);
       IIC_SCL = 1;  //确保SCL高电平
       delay_us(5);
       IIC_SDA = 0; //在SCL为高时拉低SDA线，即为起始信号
       delay_us(5);
       IIC_SCL = 0;   //钳住I2C总线，准备发送或接收数据 
    
}
```



### 3.停止信号：

**SCL保持高电平。SDA由低电平变为高电平。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407161528542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzNDgwMTMzOTM3,size_16,color_FFFFFF,t_70)

```c
//产生IIC停止信号
//1.先拉低SDA，再拉低SCL
//2.拉高SCL
//3.拉高SDA
//4.停止接收数据
void IIC_Stop(void)
{
	IIC_SCL = 0;
	IIC_SDA = 0;    //STOP:当SCL高时，数据由低变高
 	delay_us(4);
	IIC_SCL = 1; 
	IIC_SDA = 1;    //发送I2C总线结束信号
	delay_us(4);							   	
}
```



在起始条件产生后，总线处于忙状态，由本次数据传输的主从设备独占，其他I2C器件无法访问总线；而在停止条件产生后，本次数据传输的主从设备将释放总线，总线再次处于空闲状态。

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200411104137162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzNDgwMTMzOTM3,size_16,color_FFFFFF,t_70)



### 4.数据有效性：

**IIC信号在数据传输过程中，当SCL=1高电平时，数据线SDA必须保持稳定状态，不允许有电平跳变，只有在时钟线上的信号为低电平期间，数据线上的高电平或低电平状态才允许变化。**

SCL = 1时，数据线SDA的任何电平变换会看做是总线的起始信号或者停止信号。

也就是在IIC传输数据的过程中，SCL时钟线会频繁的转换电平，以保证数据的传输

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407162837546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzNDgwMTMzOTM3,size_16,color_FFFFFF,t_70)

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2Jsb2cvNTg0Mjk3LzIwMTUwMS8xMDE0MzQ1Mjc1MDc1MTUucG5n?x-oss-process=image/format,png)



### 5.应答信号

每当主机向从机发送完一个字节的数据，主机总是需要等待从机给出一个应答信号，以确认从机是否成功接收到了数据

**应答信号：主机SCL拉高，读取从机SDA的电平，为低电平表示产生应答**

- **应答信号为低电平时，规定为有效应答位（ACK，简称应答位），表示接收器已经成功地接收了该字节；**
- **应答信号为高电平时，规定为非应答位（NACK），一般表示接收器接收该字节没有成功。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407171913110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzNDgwMTMzOTM3,size_16,color_FFFFFF,t_70)

每发送一个字节（8个bit）在一个字节传输的8个时钟后的第九个时钟期间，接收器接收数据后必须回一个ACK应答信号给发送器，这样才能进行数据传输。

应答出现在每一次主机完成8个数据位传输后紧跟着的时钟周期，低电平0表示应答，1表示非应答。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200411104200168.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzNDgwMTMzOTM3,size_16,color_FFFFFF,t_70)

```c
//主机产生应答信号ACK
//1.先拉低SCL，再拉低SDA
//2.拉高SCL
//3.拉低SCL
void IIC_Ack(void)
{
   IIC_SCL = 0;   //先拉低SCL，使得SDA数据可以发生改变
   IIC_SDA = 0;   
   delay_us(2);
   IIC_SCL = 1;
   delay_us(5);
   IIC_SCL = 0;
}


//主机不产生应答信号NACK
//1.先拉低SCL，再拉高SDA
//2.拉高SCL
//3.拉低SCL
void IIC_NAck(void)
{
   IIC_SCL = 0;   //先拉低SCL，使得SDA数据可以发生改变
   IIC_SDA = 1;   //拉高SDA，不产生应答信号
   delay_us(2);
   IIC_SCL = 1;
   delay_us(5);
   IIC_SCL =0;
}
```



检查应答信号：

```c
//等待应答信号到来
//返回值：1，接收应答失败
//        0，接收应答成功
char IIC_Wait_Ack(void)
{
	u8 ucErrTime = 0;
	 
	IIC_SDA = 1;delay_us(1);	   
	IIC_SCL = 1;delay_us(1);	 
	while(IIC_SDA)
	{
		ucErrTime++;
		if(ucErrTime > 250)
		{
			IIC_Stop();
			return 1;//fail
		}
	} 
	IIC_SCL = 0;//时钟输出0 	   
	return 0;//succeed
}
```



###  6.发送数据与读数据

数据传送格式：SDA线上的数据在SCL时钟“高”期间必须是稳定的，只有当SCL线上的时钟信号为低时，数据线上的“高”或“低”状态才可以改变。输出到SDA线上的每个字节必须是8位，数据传送时，先传送最高位（MSB），每一个被传送的字节后面都必须跟随一位应答位（即一帧共有9位）。



当一个字节按数据位从高位到低位的顺序传输完后，紧接着从设备将拉低SDA线，回传给主设备一个应答位ACK， 此时才认为一个字节真正的被传输完成 ，如果一段时间内没有收到从机的应答信号，则自动认为从机已正确接收到数据。

![image-20240302174027344](https://s2.loli.net/2024/03/02/kvbCti3hejqUc6S.png)



![img](https://imgconvert.csdnimg.cn/aHR0cDovL3d3dy5lbWJlZGRlZGxpbnV4Lm9yZy5jbi91cGxvYWRzL2FsbGltZy8xMzAzMTcvMTIxOTQ1My5wbmc?x-oss-process=image/format,png)



多数从设备的地址为7位或者10位，一般都用七位。
八位设备地址=7位从机地址+读/写地址，

再给地址添加一个方向位位用来表示接下来数据传输的方向，

- 0表示主设备向从设备(write)写数据

- 1表示主设备向从设备(read)读数据

IIC的每一帧数据由9bit组成，

- 如果是发送数据，则包含 8bit数据+1bit ACK
- 如果是设备地址数据，则8bit包含7bit设备地址 1bit方向

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200411151616581.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzNDgwMTMzOTM3,size_16,color_FFFFFF,t_70)



```
在起始信号后必须传送一个从机的地址(7位) 1~7位为7位接收器件地址，第8位为读写位，用“0”表示主机发送数据(W)，“1”表示主机接收数据 （R）, 第9位为ACK应答位，紧接着的为第一个数据字节，然后是一位应答位，后面继续第2个数据字节。
```



```c
//IIC 发送一个字节
//data: 要发送的数据
void IIC_Send_Byte(uint8_t data) 
{
 	uint8_t t; 
    for (t = 0; t < 8; t++) 
    {
        //IIC_SDA=txd&0x80;   //获取最高位
        //获取数据的最高位，然后右移7位,假设为 1000 0000 右移7位为 0000 0001 
        // 假设为 0000 0000 右移7位为 0000 0000 
        //如果某位为1，则SDA为1，否则相反
        IIC_SDA((data & 0x80) >> 7); /* 高位先发送 */ 
        iic_delay(); 
        IIC_SCL(1); 
        iic_delay(); 
        IIC_SCL(0); 
        data <<= 1; /* 左移 1 位,用于下一次发送 */ 
    } 
    IIC_SDA(1); /* 发送完成, 主机释放 SDA 线 */ 
} 
```



```c
//IIC 读一个字节
//ack: ack=1 时，发送 ack; ack=0 时，发送 nack
uint8_t IIC_Read_Byte(uint8_t ack) 
{ 
     uint8_t i, receive = 0; 
     for(i = 0; i < 8; i++ ) /* 接收 1 个字节数据 */ 
     { 
         receive <<= 1; /* 高位先输出,所以先收到的数据位要左移 */ 
         IIC_SCL(1); 
         iic_delay(); 
         if (IIC_READ_SDA) 
         { 
             receive++; 
         }
         IIC_SCL(0); 
         iic_delay(); 
     } 
     if(!ack) 
     { 
     	iic_nack(); /* 发送 nACK */ 
     } 
     else 
     { 
         iic_ack(); /* 发送 ACK */ 
     }
     return receive; 
}
```



总结：

发送数据：

```
1.主机首先产生START信号
2.然后紧跟着发送一个从机地址，这个地址共有7位，紧接着的第8位是数据方 向位(R/W)，0表示主机发送数据(写)，1表示主机接收数据(读)
3.主机发送地址时，总线上的每个从机都将这7位地址码与自己的地址进行比较，若相同，则认为自己正在被主机寻址，根据R/T位将自己确定为   发送器和接收器
4.这时候主机等待从机的应答信号(A)
5.当主机收到应答信号时，发送要访问从机的那个地址， 继续等待从机的应答信号
6.当主机收到应答信号时，发送N个字节的数据，继续等待从机的N次应答信号，
7.主机产生停止信号，结束传送过程。
```



读数据：

```
1.主机首先产生START信号
2.然后紧跟着发送一个从机地址，注意此时该地址的第8位为0，表明是向从机写命令，
3.这时候主机等待从机的应答信号(ACK)
4.当主机收到应答信号时，发送要访问的地址，继续等待从机的应答信号，
5.当主机收到应答信号后，主机要改变通信模式(主机将由发送变为接收，从机将由接收变为发送)所以主机重新发送一个开始start信号，然后   紧跟着发送一个从机地址，注意此时该地址的第8位为1，表明将主机设 置成接收模式开始读取数据，
6.这时候主机等待从机的应答信号，当主机收到应答信号时，就可以接收1个字节的数据，当接收完成后，主机发送非应答信号，表示不在接收   数据
7.主机进而产生停止信号，结束传送过程。
```





### 五.DHT11

[[玩转传感器——DHT11温湿度传感器（STM32版）_温湿度传感器接线图-CSDN博客](https://blog.csdn.net/weixin_43002939/article/details/124487920?ops_request_misc=%7B%22request%5Fid%22%3A%22170943440116800188525006%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=170943440116800188525006&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-124487920-null-null.142^v99^pc_search_result_base4&utm_term=DHT11&spm=1018.2226.3001.4187)]:直接观看此篇博客

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
- [[TB6612]]
- [[STM32标准库学习记录]]
- [[PID算法学习记录]]
- [[stm32与openmv串口通信]]

