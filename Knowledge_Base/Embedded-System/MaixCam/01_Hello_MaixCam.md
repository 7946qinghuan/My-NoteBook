---
title: 01_Hello_MaixCam
date: 2026-08-01
tags: [Embedded, MaixCam]
aliases: []
---

# Hello MaixCam



## 1.画面显示

拿到MaixCam后，我们先运行一下显示代码，看看摄像头是否正常工作，画面是否正常显示。

```python
# 导入基础库
from maix import display, camera

# 获取摄像头， 并设置分辨率为640*480
cam = camera.Camera(640, 480)
# display 用于显示
disp = display.Display()
while 1:
    disp.show(cam.read())
```

> MaixPy 提供`display`模块，可以将图像显示到屏幕上，同时，在调用`display`模块的`show`方法时，会将图像发送到 MaixVision 显示。
>
> 这里我们用摄像头读取了图像，然后通过`disp.show()`方法将图像显示到屏幕上，同时也会发送到 MaixVision 显示。
>
> 当我们点击了右上角的`暂停`按钮，就会停止发送图像到 MaixVision 显示。