---
title: PID算法学习记录
date: 2026-08-01
tags: [Embedded, STM32]
aliases: []
---

# PID算法学习记录

PID:

>Proportional（比例）
>
>P比例控制：基本作用就是控制对象以线性的方式增加，在一个常量比例下，动态输出
>缺点：会产生稳态误差

> Integral（积分）
>
> I积分控制：基本作用就是用来消除稳态误差
> 缺点：会增加超调

> Differential（微分）
>
> D微分控制：基本作用就是减弱超调，加大惯性响应速度

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c37443ab4cd4620a52c9c171803641c.png)

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
- [[stm32与openmv串口通信]]

