---
title: OpenMV中的LED灯操作详解
date: 2026-08-01
tags: [Embedded, OpenMV]
aliases: []
---

# OpenMV中的LED灯操作详解

## 1.导入LED模块与延时模块：

```python
from pyb import LED
import time
```

## 2.基础LED灯：

| LED灯  | 颜色 |
| ------ | ---- |
| LED(1) | 红色 |
| LED(2) | 绿色 |
| LED(3) | 蓝色 |
| LED(4) | 红外 |

暂时不知道LED4是干嘛的。

LED灯相关函数：

首先需要实例化LED灯

```python
led = LED(1)  # 这里假设使用红色LED灯
```



| 函数         | 介绍          |
| ------------ | ------------- |
| led.on()     | 打开LED灯     |
| led.off()    | 关闭LED灯     |
| led.toggle() | 翻转LED灯状态 |



延时相关函数

| 函数               | 介绍      |
| ------------------ | --------- |
| time.sleep(1)      | 延时1s    |
| time.sleep_ms(500) | 延时500ms |
| time.sleep_us(10)  | 延时10us  |



代码示例：

实现LED灯的闪烁：

```python
from pyb import LED
import time

while True:
    # 红色LED灯闪烁
    led = LED(1)
    led.on()
    time.sleep_ms(500)
    led.off()
    time.sleep_ms(500)
```



## 3.LED灯进阶操作：

首先我们得知道OpenMV装载的不是普通LED灯，是RGB三色灯，所以我们由此产生一个想法，是否可以组合LED灯颜色，从而实现

基础三个颜色之外的颜色。

通过实践是可以的：

```python
from pyb import LED
import time

while True:
    # 这里我们实例化红色和绿色LED灯
    led1 = LED(1)
    led2 = LED(2)
    # 然后我们同时打开和关闭两个颜色的LED灯
    led1.on()
    led2.on()
    time.sleep_ms(500)
    led1.off()
    led2.off()
    time.sleep_ms(500)
    # 由此我们便实现了黄色lED灯闪烁
```



这里给出我实践过的颜色：

| 组合                     | 颜色   |
| ------------------------ | ------ |
| LED(1) + LED(2)          | 黄色   |
| LED(1) + LED(3)          | 紫色   |
| LED(2) + LED(3)          | 青蓝色 |
| LED(1) + LED(2) + LED(3) | 白色   |

### 进阶1：

然后我们便可以实现流水灯循环点亮7个颜色：

```python
from pyb import LED
import time

def Led_Use():
    # 实例化三个LED灯
    led1 = LED(1)
    led2 = LED(2)
    led3 = LED(3)
    
    # 红色
    led1.on()
    time.sleep_ms(500)
    led1.off()
    
    # 绿色
    led2.on()
    time.sleep_ms(500)
    led2.off()
    
    # 蓝色
    led3.on()
    time.sleep_ms(500)
    led3.off()
    
    # 黄色
    led1.on()
    led2.on()
    time.sleep_ms(500)
    led1.off()
    led2.off()
    
    # 紫色
    led1.on()
    led3.on()
    time.sleep_ms(500)
    led1.off()
    led3.off()
    
    # 淡蓝色
    led2.on()
    led3.on()
    time.sleep_ms(500)
    led2.off()
    led3.off()
    
    # 白色
    led1.on()
    led2.on()
    led3.on()
    time.sleep_ms(500)
    led1.off()
    led2.off()
    led3.off()
    
while True:
    Led_Use()
```



### 进阶2

为了简化我们对LED灯操作，我们可以为每个LED灯创建自己的函数使用

```python
# 0亮1灭
def Red_Use(x):
    led1 = LED(1)
    if x == 0:
        led1.on()
    else:
        led1.off()

def Green_Use(x):
    led1 = LED(2)
    if x == 0:
        led1.on()
    else:
        led1.off()

def Blue_Use(x):
    led1 = LED(1)
    if x == 0:
        led1.on()
    else:
        led1.off()

def Yellow_Use(x):
    led1 = LED(1)
    led2 = LED(2)
    if x == 0:
        led1.on()
        led2.on()
    else:
        led1.off()
        led2.off()

def Purple_Use(x):
    led1 = LED(1)
    led2 = LED(3)
    if x == 0:
        led1.on()
        led2.on()
    else:
        led1.off()
        led2.off()

def Cyan_Use(x):
    led1 = LED(2)
    led2 = LED(3)
    if x == 0:
        led1.on()
        led2.on()
    else:
        led1.off()
        led2.off()

def White_Use(x):
    led1 = LED(1)
    led2 = LED(2)
    led3 = LED(3)
    if x == 0:
        led1.on()
        led2.on()
        led3.on()
    else:
        led1.off()
        led2.off()
        led3.off()
```



### 进阶3

使用函数确实可以实现要求，但是我们可以通过创建一个LED类来帮助我们简化代码的同时也可以在其他python文件里调用

```python
from pyb import LED  
import time  

class LED_Use:  
    def Red_Use(self, x):  
        led1 = LED(1)  
        if x == 0:  
            led1.on()  
        else:  
            led1.off()  
  
    def Green_Use(self, x):  
        led2 = LED(2)  
        if x == 0:  
            led2.on()  
        else:  
            led2.off()  
  
    def Blue_Use(self, x):  
        led3 = LED(3)  
        if x == 0:  
            led3.on()  
        else:  
            led3.off()  
  
    def Yellow_Use(self, x):  
        led1 = LED(1)  
        led2 = LED(2)  
        if x == 0:  
            led1.on()  
            led2.on()  
        else:  
            led1.off()  
            led2.off()  
  
    def Purple_Use(self, x):  
        led1 = LED(1)  
        led3 = LED(3)  
        if x == 0:  
            led1.on()  
            led3.on()  
        else:  
            led1.off()  
            led3.off()  
  
    def Cyan_Use(self, x):  
        led2 = LED(2)  
        led3 = LED(3)  
        if x == 0:  
            led2.on()  
            led3.on()  
        else:  
            led2.off()  
            led3.off()  
  
    def White_Use(self, x):  
        led1 = LED(1)  
        led2 = LED(2)  
        led3 = LED(3)  
        if x == 0:  
            led1.on()  
            led2.on()  
            led3.on()  
        else:  
            led1.off()  
            led2.off()  
            led3.off() 
            
# 创建LED_Use对象  
led_controller = LED_Use()  

# 无限循环来测试LED  
while True:  
    # 点亮红色LED  
    led_controller.Red_Use(0)  
    time.sleep(1)  # 等待一秒  
    # 熄灭红色LED  
    led_controller.Red_Use(1)  
    time.sleep(1)  # 等待一秒  
    # 您可以根据需要添加其他颜色的LED控制逻辑
```



### 进阶4

我们还可以简化代码，我们可以知道这里基础有三个LED灯，我们把它们看作成三位二进制数

| 二进制 | 颜色   |
| ------ | ------ |
| 000    | 白色   |
| 001    | 黄色   |
| 010    | 紫色   |
| 100    | 青蓝色 |
| 011    | 红色   |
| 101    | 绿色   |
| 110    | 蓝色   |
| 111    | 全灭   |

当然，为了简化操作，我还创建了一个可以根据单词选择颜色的子函数

```python
from pyb import LED
import time

class LED_Use:
    def __init__(self):
        # 在初始化时创建LED对象
        self.red_led = LED(1)
        self.green_led = LED(2)
        self.blue_led = LED(3)

    def switch_led(self, led, state):
        # 切换单个LED的状态
        if state == 0:
            led.on()
        else:
            led.off()

    def set_color(self, red, green, blue):
        # 设置LED的颜色
        # 参数应为0或1，分别表示LED的亮或灭
        self.switch_led(self.red_led, red)
        self.switch_led(self.green_led, green)
        self.switch_led(self.blue_led, blue)
    # 两种方法最好不要一起用，一起用的话需要在它们之间关闭所有LED灯
    def choose_color(self, color, state):
        # 设置组合颜色的LED状态
        # 例如，设置红色和绿色LED同时亮或灭
        if color == 'red':
            self.switch_led(self.red_led, state)
        elif color == 'green':
            self.switch_led(self.green_led, state)
        elif color == 'blue':
            self.switch_led(self.blue_led, state)
        elif color == 'white':
            self.switch_led(self.red_led, state)
            self.switch_led(self.green_led, state)
            self.switch_led(self.blue_led, state)
        elif color == 'cyan':
            self.switch_led(self.green_led, state)
            self.switch_led(self.blue_led, state)
        elif color == 'yellow':
            self.switch_led(self.red_led, state)
            self.switch_led(self.green_led, state)
        elif color == 'purple':
            self.switch_led(self.red_led, state)
            self.switch_led(self.blue_led, state)

# 创建LED_Use对象
led_controller = LED_Use()

# 测试代码
while True:
    # 点亮红色LED灯
    led_controller.set_color(0, 1, 1)
    time.sleep(1)  # 等待一秒
	
    # 熄灭所有LED灯
    led_controller.set_color(1, 1, 1)
    time.sleep(1)  # 等待一秒

    # 点亮白色LED灯
    led_controller.set_color(0, 0, 0)
    time.sleep(1)  # 等待一秒
	
    # 熄灭所有LED灯
    led_controller.set_color(1, 1, 1)
    time.sleep(1)  # 等待一秒
	
    # 点亮青蓝色LED灯
    led_controller.choose_color('cyan', 0)
    time.sleep(1)  # 等待一秒

```

---

## 相关笔记

**OpenMV 基础**

- [[OpenMV基础知识]]
- [[OpenMV IDE界面介绍]]
- [[OpenMV中的定时器]]
- [[卡尔曼滤波算法]]

