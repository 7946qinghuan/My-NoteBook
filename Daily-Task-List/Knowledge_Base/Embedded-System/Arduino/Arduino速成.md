# ==Arduino速成==





## 一.main函数简介：



1.

```c
void setup()
```

这个函数用来放初始化函数



2.

```c
void loop()
```

这个函数相当于`void main()函数里的while(1)`



## 二.信号引脚输入输出

### 1.数字输入输出

①.GPIO配置

```c
void pinMode(uint8_t pin, uint8_t mode);
```

参数1：引脚号

参数2：GPIO模式选择

```c
#define OUTPUT            0x03 
#define PULLUP            0x04
#define INPUT_PULLUP      0x05		//不能接负电平，不能接大于5V的电平
#define PULLDOWN          0x08
#define INPUT_PULLDOWN    0x09
#define OPEN_DRAIN        0x10
#define OUTPUT_OPEN_DRAIN 0x12
#define ANALOG            0xC0
```



a)数字输入

引脚电平读取

```c
int digitalRead(uint8_t pin);
```

参数1：引脚号

返回值，引脚电平高低



b)数字输出

写入引脚电平

```c
void digitalWrite(uint8_t pin, uint8_t val);
```

参数1：引脚号

参数2：电平状态

```c
#define LOW               0x0
#define HIGH              0x1
```



### 2.模拟信号输入输出

a)模拟输入

```c
uint16_t analogRead(uint8_t pin);
```

参数1：引脚号

返回值：获取引脚的ADC值



b)模拟输出

```
void analogWrite(uint8_t pin, uint8_t val)
```

参数1：引脚号

参数2：0~255



## 三.串口通信

#### 1.串口波特率初始化

```c
Serial.begin(speed);
```

参数**speed**是指串口通信波特率



#### 2.串口打印

```c
Serial.print(val);
Serial.println(val); //与print不同点是输出指定数据后回车
```



##### ①.print()

- 功能：串口输出数据，写入字符数据到串口。将数据输出到串口。数据会以ASCII码形式输出。如果想以字节形式输出数据，则需要使用 `write()` 函数。
- 语法：

```c
Serial.print(val)
Serial.print(val, format)
```

- 参数：
  val：需要输出的数据，任意数据类型。
  format：输出的数据格式。BIN(二进制)、OCT(八进制)、DEC(十进制)、HEX(十六进制)。对于浮点数，此参数指定要使用的小数位数（默认输出2位）。

参数val是你要输出的数据，各种类型的数据均可。
下面的示例程序中，演示了使用串口输出数据到计算机：

示例：

```c
Serial.print(55, BIN) 		 	// 输出 "110111"
Serial.print(55, OCT) 			// 输出 "67"
Serial.print(55, DEC) 			// 输出 "55"
Serial.print(55, HEX) 			// 输出 "37
Serial.print(3.1415926, 0) 		// 输出 "3"
Serial.print(3.1415926, 2) 		// 输出 "3.14"
Serial.print(3.1415926, 4)  	// 输出 "3.1416"
Serial.print('N') 				// 输出 "N"
Serial.print("Hello World！") 	// 输出 "Hello World！"
```

- 返回值：返回输出的字节数。



##### ②.println()

- 功能：将数据输出到串口,并回车换行。数据会以ASCII码形式输出。

- 语法：

  ```v
  Serial.println(val)
  Serial.println(val, format)
  ```

- 参数：
  val：需要输出的数据，任意数据类型。
  format：输出的数据格式。和 `Serial.print(val)` 和相同。

- 返回值：返回输出的字节数。



完整示例：

```c
int counter=0; // 计数器

void setup() {
// 初始化串口
  Serial.begin(9600);
}

void loop() {
// 每loop循环一次，计数器变量加1
counter = counter+1;
// 输出变量
Serial.print(counter);
// 输出字符
Serial.print( ':' );
// 输出字符串;
Serial.println("Hellow World");
delay(1000);
}
```



#### 3.串口读取

##### ①.read()

- 功能：读取串口数据，一次读一个字符，读完后删除已读数据。

- 语法：

  ```c
  Serial.read()
  ```

- 参数：无。

- 返回值：返回串口缓存区中第一个可读字节，当没有可读数据时返回-1。



##### ②.readBytes()

- 功能：从接收缓冲区读取指定长度的字符，并将其存人一个数组中。若等待数据时间超过设定的超时时间，则退出该函数。
- 语法：

```c
Serial.readBytes(buffer, length)
```

- 参数：
  buffer，用于存储数据的数组(char[]或者byte[])。
  length，需要读取的字符长度。
- 返回值：读到的字节数；如果没有找到有效的数据，则返回0。



##### ③. peek( )

- 功能：返回1字节的数据，但不会从接收缓冲区删除该数据。与read()函数不同，read()函数读取数据后，会从接收缓冲区删除该数据。

- 语法：

  ```c
  Serial. peek()
  ```

- 语法：`Serial. peek()`

- 参数：无。

- 返回值：进入接收缓冲区的第1字节的数据；如果没有可读数据，则返回-1。