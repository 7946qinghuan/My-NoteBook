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