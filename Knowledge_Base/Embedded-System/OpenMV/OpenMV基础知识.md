# 1.感光元件-OpenMV初认识

OpenMV中使用==image.lens_corr(1.8)==来矫正2.8mm焦距的镜头。（鱼眼效果也叫桶型畸变）

但是我们这里使用的`OpenMV-H7-Plus`，一般使用以下语句矫正鱼眼畸变

```python
img = sensor.snapshot().lens_corr(1.6)  # 鱼眼矫正
```



```python
import sensor#引入感光元件的模块

# 设置摄像头
sensor.reset()#初始化感光元件，将摄像头模块重置到默认状态 
sensor.set_pixformat(sensor.RGB565) # 设置摄像头的像素格式为RGB565，这是一种彩色格式，可以捕获红绿蓝三种颜色 
sensor.set_framesize(sensor.QVGA)# 设置摄像头拍摄的照片大小为QVGA（320x240像素）
n = 10
sensor.skip_frames(n)#跳过n张照片，在更改设置后，跳过一些帧，等待感光元件变稳定。

# 一直拍照
while(True):
    img = sensor.snapshot()#拍摄一张照片，img为一个image对象
```

## 1.1 初始化感光元件

`sensor.reset()` 



## 1.2 设置图像的大小

`sensor.set_framesize()`

- ```
  sensor.QQCIF: 88x72
  sensor.QCIF: 176x144
  sensor.CIF: 352x288
  sensor.QQSIF: 88x60
  sensor.QSIF: 176x120
  sensor.SIF: 352x240
  sensor.QQQQVGA: 40x30
  sensor.QQQVGA: 80x60
  sensor.QQVGA: 160x120
  sensor.QVGA: 320x240
  sensor.VGA: 640x480
  sensor.HQQQVGA: 80x40
  sensor.HQQVGA: 160x80
  sensor.HQVGA: 240x160
  sensor.B64X32: 64x32 (用于帧差异 image.find_displacement())
  sensor.B64X64: 64x64 用于帧差异 image.find_displacement())
  sensor.B128X64: 128x64 (用于帧差异 image.find_displacement())
  sensor.B128X128: 128x128 (用于帧差异 image.find_displacement())
  sensor.LCD: 128x160 (用于LCD扩展板)
  sensor.QQVGA2: 128x160 (用于LCD扩展板)
  sensor.WVGA: 720x480 (用于 MT9V034)
  sensor.WVGA2:752x480 (用于 MT9V034)
  sensor.SVGA: 800x600 (仅用于 OV5640 感光元件)
  sensor.XGA: 1024x768 (仅用于 OV5640 感光元件)
  sensor.SXGA: 1280x1024 (仅用于 OV5640 感光元件)
  sensor.UXGA: 1600x1200 (仅用于 OV5640 感光元件)
  sensor.HD: 1280x720 (仅用于 OV5640 感光元件)
  sensor.FHD: 1920x1080 (仅用于 OV5640 感光元件)
  sensor.QHD: 2560x1440 (仅用于 OV5640 感光元件)
  sensor.QXGA: 2048x1536 (仅用于 OV5640 感光元件)
  sensor.WQXGA: 2560x1600 (仅用于 OV5640 感光元件)
  sensor.WQXGA2: 2592x1944 (仅用于 OV5640 感光元件)
  ```

  其实我们所设置的图像大小，就是我们的**分辨率**

  

## 1.3 设置彩色／黑白

- `sensor.set_pixformat() `设置像素模式。
  
  - `sensor.GRAYSCALE`： 灰度，每个像素8bit。
  
  - `sensor.RGB565`：彩色，每个像素16bit。
  
    ```python
    sensor.set_pixformat(sensor.RGB565) # 设置摄像头的像素格式为RGB565，这是一种彩色格式，可以捕获红绿蓝三种颜色
    
    sensor.set_framesize(sensor.QVGA)# 设置摄像头拍摄的照片大小为QVGA（320x240像素）
    ```
  
    

## 1.4 自动增益／白平衡／曝光

- `sensor.set_auto_gain()` 自动增益开启（True）或者关闭（False）。在使用==颜色追踪==时，需要==关闭自动增益==。

  - 它允许摄像头根据场景的光照条件自动调整图像的亮度。

    ```python
    # 开启自动增益控制  
    sensor.set_auto_gain(True)  
      
    # 关闭自动增益控制，并设置固定的增益值  
    sensor.set_auto_gain(False, gain_db=0)
    ```

    

- `sensor.set_auto_whitebal()` 自动白平衡开启（True）或者关闭（False）。在使用==颜色追踪==时，需要==关闭自动白平衡==。

- `sensor.set_auto_exposure(enable[\, exposure_us])`
  
  - enable 打开（True）或关闭（False）自动曝光。默认打开。
  - 如果 enable 为False， 则可以用 exposure_us 设置一个固定的曝光时间（以微秒为单位）。
  - ```python
    # 开启自动曝光  
    sensor.set_auto_exposure(True, exposure_us=0)  
    
    # 关闭自动曝光，并设置固定的曝光时间（以微秒为单位）  
    sensor.set_auto_exposure(False, exposure_us=50000)
    ```
  
    当`sensor.set_auto_exposure(True, exposure_us=0)`被调用时，摄像头将尝试自动调整曝光时间以获得最佳图像。`exposure_us`参数在这种情况下应设置为0，表示使用自动曝光。
  
    

## 1.5 设置窗口ROI

`sensor.set_windowing(roi)`

ROI：Region Of Interest，图像处理中的术语“==感兴趣区==”。就是在要处理的图像中提取出的要处理的区域。

`sensor.set_windowing(roi)`函数用于设置摄像头的窗口化（Windowing）功能。窗口化是一种技术，它允许摄像头仅捕获图像的一个特定区域（Region of Interest, ROI），而不是整个图像。这可以减少处理时间并降低功耗，特别是在你只对图像的一部分感兴趣时。

`roi`是一个四元组，定义了ROI的左上角和右下角的坐标，格式为`(x, y, w, h)`，其中`(x, y)`是ROI左上角的坐标，`w`是ROI的宽度，`h`是ROI的高度。坐标和尺寸通常是以像素为单位的整数。

![img](https://book.openmv.cc/assets/05-01-001.png)

```python
import sensor, image, time  

sensor.reset() # 初始化摄像头  
sensor.set_pixformat(sensor.RGB565) # 设置像素格式 
sensor.set_framesize(sensor.QVGA) # 设置帧大小  
sensor.skip_frames(time = 2000) # 等待摄像头设置生效  

# 定义ROI区域，例如捕获图像中心的一个160x120像素的区域  
roi = (80, 60, 160, 120)  
sensor.set_windowing(roi) # 设置窗口化区域  

while(True):  
    img = sensor.snapshot() # 拍摄ROI区域的照片  
    # 在这里可以对img进行进一步处理或显示  
    # ...
```



## 1.6 设置翻转

```python
sensor.set_hmirror(True)  # 水平方向翻转
sensor.set_vflip(True)  # 垂直方向翻转
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

```python
import sensor, image, time  
  
sensor.reset() # 初始化摄像头  
sensor.set_pixformat(sensor.RGB565) # 设置像素格式  
sensor.set_framesize(sensor.QVGA) # 设置帧大小  
sensor.skip_frames(time = 2000) # 等待摄像头设置生效  
  
# 开启水平方向翻转  
sensor.set_hmirror(True)  
  
# 开启垂直方向翻转  
sensor.set_vflip(True)  
  
while(True):  
    img = sensor.snapshot() # 捕获翻转后的图像  
    # 在这里可以对img进行进一步处理或显示  
    # ...
```



------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 2.图像(image)的基本运算

## 2.1 坐标的表示

![img](https://book.openmv.cc/assets/05-02-001.jpg)



## 2.2 获取/设置像素点

我们可以通过`image.get_pixel(x, y)`方法来获取一个像素点的值。

- `image.get_pixel(x, y)`
  
  - 对于灰度图: 返回(x,y)坐标的灰度值.
  
  - 对于彩色图: 返回(x,y)坐标的(r,g,b)的tuple.
  
    ```python
    import sensor, image, time  
    
    sensor.reset() # 初始化摄像头  
    sensor.set_pixformat(sensor.GRAYSCALE) # 设置像素格式为GRAYSCALE
    # sensor.set_pixformat(sensor.RGB565 )   # 设置像素格式为RGB565
    sensor.set_framesize(sensor.QVGA) # 设置帧大小为QVGA（320x240像素）  
    sensor.skip_frames(time = 2000) # 等待摄像头设置生效  
    
    while(True):  
        img = sensor.snapshot() # 捕获一帧图像  
        pixel_color = img.get_pixel(100, 50) # 获取(100, 50)坐标处的像素颜色值  
        print(pixel_color) # 打印像素颜色值，例如：(12, 34, 56)  
        # 在这里可以对像素颜色值进行进一步处理或分析  
        # ...
    ```
  
    在OpenMV中，图像通常被表示为一个二维数组，其中每个元素都是一个像素。每个像素可以有一个或多个颜色通道的值，具体取决于图像的像素格式。例如，在`RGB565`格式中，每个像素由红色、绿色和蓝色三个通道组成，每个通道的值在0到31（红色或蓝色）、0到63（绿色）之间。
  
    `RGB565`格式中：
  
    - R（红色）：0到31（2^5^ - 1）
    - G（绿色）：0到63（2^6^ - 1）
    - B（蓝色）：0到31（2^5^ - 1）
  
    
  
    `RGB888`格式中：
  
    - R（红色）：0到255（2^8^ - 1）
    - G（绿色）：0到255（2^8^ - 1）
    - B（蓝色）：0到255（2^8^ - 1）
  
    但是通过实践发现在OpenMV中设置像素格式为`RGB565`后，每个颜色通道返回的最大值为255。
  
    原因是：OpenMV的API会自动将`RGB565`格式的颜色通道值转换为`RGB888`格式，然后返回。

同样，我们可以通过`image.set_pixel(x, y, pixel)`方法，来设置一个像素点的值。

- `image.set_pixel(x, y, pixel)`
  
  - 对于灰度图: 设置(x,y)坐标的灰度值。
  
  - 对于彩色图: 设置(x,y)坐标的(r,g,b)的值。
  
    同样，在`RGB565`像素格式下，`pixel` 应该是`RGB565`格式的颜色通道值元组，但是我们提供的确是`RGB888`格式的颜色通道值元组，输入后OpenMV会在内部将这些值自动转换为`RGB565`格式。



为什么不直接选择`RGB888`像素格式作为输入和输出呢？

原因是`RGB565`像素格式其较小的位数需求，在存储和传输方面更为高效，节省空间和带宽。

举例：

```python
import sensor, image, time

sensor.reset() # 初始化摄像头
sensor.set_pixformat(sensor.RGB565) # 设置像素格式为RGB565
sensor.set_framesize(sensor.QVGA) # 设置帧大小
sensor.skip_frames(time = 2000) # 等待摄像头设置生效

while(True):
    img = sensor.snapshot() # 捕获一帧图像
    # 设置像素值，这里使用RGB888格式的值，OpenMV会自动转换
    for i in range(0, 321):
        img.set_pixel(i, 50, (255, 128, 64)) # 设置(0, 50)<-->(320, 50)的像素为红色偏橙
    # 可以在这里添加显示或保存图像的代码
    # ...
```



## 2.3 获取图像的宽度和高度

- `image.width()`
  返回图像的宽度(像素)
- `image.height()`
  返回图像的高度(像素)
- `image.format()`
  灰度图会返回 sensor.GRAYSCALE，彩色图会返回 sensor.RGB565。
- `image.size()`
  返回图像的大小(byte)

```python
import sensor, image, time

sensor.reset() # 初始化摄像头模块
sensor.set_pixformat(sensor.RGB565) # 设置像素格式为RGB565
sensor.set_framesize(sensor.QVGA) # 设置帧大小为QVGA（320x240像素）
sensor.skip_frames(time = 2000) # 等待设置生效并跳过几帧

while(True):
    img = sensor.snapshot() # 捕获一帧图像
    pixformat = img.format() # 获取图像的像素格式
    print("Image width:", img.width()) # 打印图像的宽度
    print("Image height:", img.height()) # 打印图像的高度
   # 可以通过比较pixformat来确定具体的像素格式
    if pixformat == sensor.RGB565:
        print("The image is in RGB565 format.")
    elif pixformat == sensor.RGB888:
        print("The image is in RGB888 format.")
    print("Image size():", img.size(), f"= {img.width()}*{img.height()}*2 bye") # 打印图像的大小
    # 这里可以添加更多处理图像的代码...
    time.sleep(1000) # 暂停一秒钟（1000毫秒）
```



## 2.4 图像的运算

- `image.invert()`

取反，对于二值化的图像，0(黑)变成1(白)，1(白)变成0(黑)。

注：图像可以是另一个image对象，或者是从 (bmp/pgm/ppm)文件读入的image对象。

```python
import sensor, image, time
  
sensor.reset() # 初始化摄像头模块  
sensor.set_pixformat(sensor.RGB565) # 设置像素格式为RGB565  
sensor.set_framesize(sensor.QVGA) # 设置帧大小为QVGA  
sensor.skip_frames(time = 2000) # 等待设置生效并跳过几帧  
  
while(True):  
    original_img = sensor.snapshot() # 捕获原始图像  
    inverted_img = original_img.copy() # 复制图像对象  
    inverted_img.invert() # 反转复制的图像的颜色  
    # 这里可以添加更多处理反转后图像的代码...  
     
    time.sleep(1000) # 暂停一秒钟（1000毫秒）
```

使用以下方法时，两个图像都必须是相同的尺寸和类型（灰度图/彩色图）。

- `image.nand(image)`
  与另一个图片进行与非（NAND）运算。
- `image.nor(image)`
  与另一个图片进行或非（NOR）运算。
- `image.xor(image)`
  与另一个图片进行异或（XOR）运算。
- `image.xnor(image)`
  与另一个图片进行异或非（XNOR）运算。
- `image.difference(image)`
  从这张图片减去另一个图片。比如，对于每个通道的每个像素点，取相减绝对值操作。这个函数，经常用来做移动检测。

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



# 3.使用图像的统计信息

如果我想知道一个区域内的平均颜色或者占面积最大的颜色？

使用统计信息——Statistics！

## 3.1 ROI感兴趣的区域

![img](https://book.openmv.cc/assets/05-03-001.jpg)
roi的格式是`(x, y, w, h)`的tupple.

- x:ROI区域中==左上角的x坐标==（注：从0开始）
- y:ROI区域中==左上角的y坐标==（注：从0开始）
- w:ROI的宽度
- h:ROI的高度

## 3.2 Statistics

```python
image.get_statistics(roi=Auto)
```

其中roi是目标区域。注意，这里的roi，bins之类的参数，一定要**==显式==**地标明，例如：

```
img.get_statistics(roi=(0,0,10,20))
```

==如果是 img.get_statistics((0,0,10,20))，ROI不会起作用。==

对于灰度图片，返回值如下：

- ==statistics.mean()== 返回灰度的**平均数**(0-255) (int)。你也可以通过statistics[0]获得。
- ==statistics.median()== 返回灰度的**中位数**(0-255) (int)。你也可以通过statistics[1]获得。
- ==statistics.mode()== 返回灰度的**众数**(0-255) (int)。你也可以通过statistics[2]获得。
- ==statistics.stdev()== 返回灰度的**标准差**(0-255) (int)。你也可以通过statistics[3]获得。
- ==statistics.min()== 返回灰度的**最小值**(0-255) (int)。你也可以通过statistics[4]获得。
- ==statistics.max()== 返回灰度的**最大值**(0-255) (int)。你也可以通过statistics[5]获得。
- ==statistics.lq()== 返回灰度的**第一四分数**(0-255) (int)。你也可以通过statistics[6]获得。
- ==statistics.uq()== 返回灰度的**第三四分数**(0-255) (int)。你也可以通过statistics[7]获得。

对于RGB565图片，返回值如下：

- l_mean，l_median，l_mode，l_stdev，l_min，l_max，l_lq，l_uq，
- a_mean，a_median，a_mode，a_stdev，a_min，a_max，a_lq，a_uq，
- b_mean，b_median，b_mode，b_stdev，b_min，b_max，b_lq，b_uq，

分别是是==LAB三个通道的==平均数，中位数，众数，标准差，最小值，最大值，第一四分数，第三四分数。



## 3.3 举例

检测左上方的区域中的颜色值。

```python
import sensor, image, time

sensor.reset() # 初始化摄像头
sensor.set_pixformat(sensor.RGB565) # 格式为 RGB565.
sensor.set_framesize(sensor.QVGA)
sensor.skip_frames(10) # 跳过10帧，使新设置生效
sensor.set_auto_whitebal(False)               # Create a clock object to track the FPS.

ROI=(80,30,15,15)

while(True):
    img = sensor.snapshot()         # Take a picture and return the image.
    statistics=img.get_statistics(roi=ROI)
    color_l=statistics.l_mode()
    color_a=statistics.a_mode()
    color_b=statistics.b_mode()
    print(color_l,color_a,color_b)
    img.draw_rectangle(ROI)
```

结果：

![img](https://book.openmv.cc/assets/05-03-002.jpg)

终端

```python
56 66 51
56 66 55
56 66 51
56 66 51
56 66 51
56 66 51
56 66 51
56 66 51
56 66 51
56 66 51
56 66 51
56 66 51
56 66 51
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 4. 画图

视觉系统通常需要给使用者提供一些反馈信息。直接在图像中显示出来，很直观，所以，当找到色块，把这个区域用矩形框标注出来，这样非常直观。

![img](https://book.openmv.cc/assets/05-04-001.jpg)

注意：

- 颜色可以是灰度值(0-255)，或者是彩色值(r, g, b)的tupple。默认是白色。
- ==其中的color关键字必须**显示**的标明**color=**==。例如：

```python
image.draw_line((10,10,20,30), color=(255,0,0))
image.draw_rectangle(rect_tuple, color=(255,0,0))
```

那么如何更改阈值从而寻找目标颜色呢？

## 4.1 更改阈值

那么如何自己更改这个阈值呢？我们怎么知道我们的物体的颜色阈值呢？

- 数字列表项目首先在摄像头中找到目标颜色，在framebuffer中的目标颜色上左击圈出一个矩形
- 在framebuffer下面的坐标图中，选择LAB Color Space。

![img](https://book.openmv.cc/assets/02-019.jpg)

- 三个坐标图分别表示圈出的矩形区域内的颜色的LAB值，选取三个坐标图的最大最小值，即( 0, 60, -50, -10, 0, 30)

![img](https://book.openmv.cc/assets/02-020.jpg)

## 4.2 画线

- `image.draw_line(line_tuple, color=White)` 在图像中画一条直线。
  - line_tuple的格式是(x0, y0, x1, y1)，意思是(x0, y0)到(x1, y1)的直线。
  - 颜色可以是灰度值(0-255)，或者是彩色值(r, g, b)的tupple。默认是白色

## 4.3 画框

- `image.draw_rectangle(rect_tuple, color=White)` 在图像中画一个矩形框。
  - rect_tuple 的格式是 (x, y, w, h)。

## 4.4 画圆

- `image.draw_circle(x, y, radius, color=White)` 在图像中画一个圆。
  - x,y是圆心坐标
  - radius是圆的半径

## 4.5 画十字

- `image.draw_cross(x, y, size=5, color=White) `在图像中画一个十字
  - x,y是坐标
  - size是两侧的尺寸

## 4.6 写字

- `image.draw_string(x, y, text, color=White)` 在图像中写字 8x10的像素
  - x,y是坐标。使用\n, \r, and \r\n会使光标移动到下一行。
  - text是要写的字符串。

## 4.7 例子

```python
import sensor, image, time

sensor.reset() # 初始化摄像头
sensor.set_pixformat(sensor.RGB565) # 格式为 RGB565.
sensor.set_framesize(sensor.QQVGA)  # 160*120
sensor.skip_frames(10) # 跳过10帧，使新设置生效
while(True):
    img = sensor.snapshot()         # Take a picture and return the image.
    img.draw_line((40, 0, 40, 160))
    img.draw_line((120, 0, 120, 160), color=(255,0,0))
    img.draw_rectangle((50, 30, 60, 20), color=(255,0,0))
    img.draw_circle(80, 60, 30)
    img.draw_cross(80,60,size=10)
    img.draw_string(60,0, "hello\r\nworld!\r\n")
```



------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 5.寻找色块

## 5.1` find_blobs()`函数

```python
image.find_blobs(thresholds, roi=Auto, x_stride=2, y_stride=1, invert=False, area_threshold=10, pixels_threshold=10, merge=False, margin=0, threshold_cb=None, merge_cb=None)
```

参数介绍：

- **`thresholds`**：这是一个元组，用于定义颜色或亮度阈值。对于灰度图像，它通常是一个 `(lower, upper)` 的形式，表示要查找的像素值范围。对于彩色图像，它通常是一个包含 6 个值的元组 `(l_h, l_s, l_v, h_h, h_s, h_v)`，其中 `l_h`, `l_s`, `l_v` 是颜色空间的下限（low hue, saturation, value），而 `h_h`, `h_s`, `h_v` 是上限（high hue, saturation, value）。

  - 注意：这个参数是一个列表，可以包含多个颜色。如果你只需要一个颜色，那么在这个列表中只需要有一个颜色值，如果你想要多个颜色阈值，那这个列表就需要多个颜色阈值。
  - 注意：对于彩色图像，在返回的色块对象blob可以调用code方法，来判断是什么颜色的色块。

  ```python
  red = (xxx,xxx,xxx,xxx,xxx,xxx)
  blue = (xxx,xxx,xxx,xxx,xxx,xxx)
  yellow = (xxx,xxx,xxx,xxx,xxx,xxx)
  
  img=sensor.snapshot()
  red_blobs = img.find_blobs([red])
  
  color_blobs = img.find_blobs([red,blue, yellow])
  ```



- **`roi`**：这是一个四元组 `(x, y, w, h)`，定义了查找 blob 的感兴趣区域（Region of Interest, ROI）。如果设置为 `Auto`，则使用整个图像作为 ROI。

  ```python
  left_roi = [0,0,160,240]
  blobs = img.find_blobs([red],roi=left_roi)
  ```

  

- **`x_stride`**：是查找某色块时需要跳过的x像素的数量。找到色块后，直线填充算法将精确像素。 若已知色块较大，可增加 `x_stride` 来提高查找色块的速度。

  ```python
  blobs = img.find_blobs([red],x_stride=10)
  ```

  

- **`y_stride`**：是查找某色块时需要跳过的y像素的数量。找到色块后，直线填充算法将精确像素。 若已知色块较大，可增加 `y_stride` 来提高查找色块的速度。

  ```python
  blobs = img.find_blobs([red],y_stride=5)
  ```

  我们所寻找的色块只要有一部分在跳过后的区域，即使很大部分在跳过区，线条填充算法也能将整个色块包含其中。

  通常`x_stride`与`y_stride`配合使用

  ```python
  import sensor, image, time  
  
  sensor.reset() # 初始化摄像头  
  sensor.set_pixformat(sensor.GRAYSCALE) # 设置图像格式为灰度  
  sensor.set_framesize(sensor.QVGA) # 设置图像分辨率 320*240
  sensor.skip_frames(time = 2000) # 等待摄像头设置生效  
  
  threshold = (200, 255) # 灰度阈值，用于检测白色 blob  
  
  while(True):  
      img = sensor.snapshot() # 捕获一帧图像  
      
      # 使用 x_stride 和 y_stride 查找 blob  
      blobs_with_stride = img.find_blobs([threshold], x_stride=50, y_stride=10)  
      
      # 在找到的每个 blob 上绘制矩形  
      for blob in blobs_with_stride:  
          img.draw_rectangle(blob.rect(), color=(255, 0, 0))  
  ```

  

- **`invert`**：反转阈值，把阈值以外的颜色作为阈值进行查找

- **`area_threshold `**：面积阈值，如果色块被框起来的面积小于这个值，会被过滤掉

- **`pixels_threshold `**：像素个数阈值，如果色块像素数量小于这个值，会被过滤掉

- **`merge `**：合并，如果设置为True，那么合并所有重叠的blob为一个。
  注意：这会合并所有的blob，无论是什么颜色的。如果你想混淆多种颜色的blob，只需要分别调用不同颜色阈值的find_blobs。

```python
all_blobs = img.find_blobs([red,blue,yellow],merge=True)

red_blobs = img.find_blobs([red],merge=True)
blue_blobs = img.find_blobs([blue],merge=True)
yellow_blobs = img.find_blobs([yellow],merge=True)
```

- **`margin`**：边界，如果设置为1，那么两个blobs如果间距1一个像素点，也会被合并。

find_blobs()函数实践：寻找最大色块 

```python
# 定义颜色阈值
#threshold = (100, 95, -128, 127, -10, 127) # (l_r, l_g, l_b, h_r, h_g, h_b)  

import sensor, image, time  
sensor.reset() # 初始化摄像头  
sensor.set_pixformat(sensor.RGB565) # 设置像素格式  
sensor.set_framesize(sensor.QVGA) # 设置帧大小  
sensor.skip_frames(time = 2000) # 等待设置生效  
sensor.set_vflip(True)  # 垂直方向翻转
sensor.set_hmirror(True)  # 水平方向翻转
threshold = (100, 95, -128, 127, -10, 127) # 设置颜色阈值，这里你可以根据需要调整  

# 方法一
def find_largest_blob(img):  
   blobs = img.find_blobs([threshold]) # 在图像中查找色块  
   if not blobs:  
       return None  
   largest_blob = max(blobs, key=lambda x: x.pixels()) # 找出最大的色块  
   return largest_blob  

# 方法二
def find_max(blobs):
    max_size = 0
    max_blob = None
    for blob in blobs:
        if blob.w() * blob.h() > max_size:
            max_blob = blob
            max_size = blob.w() * blob.h()
    return max_blob

while(True):
   img = sensor.snapshot() # 拍摄一张照片
   blobs = img.find_blobs([threshold])
   #   largest_blob = find_largest_blob(img) # 查找最大色块  
   #   if largest_blob:  
   #       img.draw_rectangle(largest_blob.rect(), color=(255, 0, 0)) # 在图像上绘制最大色块的矩形框  
   #   print(img) # 显示图像
   if blobs:
        max_blob = find_max(blobs)
        img.draw_rectangle(max_blob.rect(), color=(255, 0, 0))
```



### 拓展：lambda详解：

`ambda`函数的基本语法如下：

```python
lambda arguments: expression
```

- `arguments` 是函数的参数，可以有一个或多个，用逗号分隔。
- `expression` 是关于这些参数的表达式，函数体只有这一行。

`lambda`函数返回一个函数对象，这个对象可以像普通函数一样被调用。

`lambda` 表达式，又称[匿名函数](https://so.csdn.net/so/search?q=匿名函数&spm=1001.2101.3001.7020)，常用来表示内部仅包含 1 行表达式的函数。如果一个函数的函数体仅有 1 行表达式，则该函数就可以用 `lambda` 表达式来代替。

下面是一些`lambda`函数的示例：

#### 示例1：基本用法

```python
f = lambda x: x * 2  
print(f(5))  # 输出: 10
```

#### 示例2：多个参数

```python
g = lambda x, y: x + y  
print(g(3, 4))  # 输出: 7
```

#### 示例3：在`sorted`函数中使用`lambda`进行排序

```python
words = ['apple', 'banana', 'cherry']  
words_sorted_by_length = sorted(words, key=lambda word: len(word))  
print(words_sorted_by_length)  # 输出: ['apple', 'cherry', 'banana']
```

#### 示例4：在`filter`函数中使用`lambda`进行过滤

```python
numbers = [1, 2, 3, 4, 5, 6]  
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))  
print(even_numbers)  # 输出: [2, 4, 6]
```

#### 示例5：在`map`函数中使用`lambda`进行映射

```python
numbers = [1, 2, 3, 4, 5]  
squared = list(map(lambda x: x**2, numbers))  
print(squared)  # 输出: [1, 4, 9, 16, 25]
```

#### 示例6：在`reduce`函数中使用`lambda`进行累积计算（需要`functools`模块）

```python
from functools import reduce  
  
numbers = [1, 2, 3, 4, 5]  
product = reduce(lambda x, y: x * y, numbers, 1)  
print(product)  # 输出: 120
```

[python——lambda函数_python lambda函数-CSDN博客](https://blog.csdn.net/qq_55858843/article/details/127789066?ops_request_misc=%7B%22request%5Fid%22%3A%22171402763616800180684290%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=171402763616800180684290&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-127789066-null-null.142^v100^pc_search_result_base4&utm_term=lambda&spm=1018.2226.3001.4187)

## 5.2 颜色阈值选择工具

OpenMV 的IDE里加入了阈值选择工具，极大的方便了对于颜色阈值的调试。

首先运行hello world.py让IDE里的framebuffer显示图案。
然后打开 工具 → （机器视觉）Mechine Vision → （阈值编辑器）Threshold Editor

点击 Frame Buffer可以获取IDE中的图像，Image File可以自己选择一个图像文件。

拖动六个滑块，可以实时的看到阈值的结果，我们想要的结果就是，将我们的==目标颜色变成白色，其他颜色全变为黑色==。

## 5.3 blobs是一个列表

`find_blobs`对象返回的是多个blob的列表。（注意区分`blobs`和`blob`，这只是一个名字，用来区分多个色块，和一个色块）。
列表类似与C语言的数组，一个`blobs`列表里包含很多`blob`对象，`blobs`对象就是色块，每个`blobs`对象包含一个色块的信息。

```python
blobs = img.find_blobs([red])
```

`blobs`就是很多色块。

可以用for循环把所有的色块找一遍。

```python
for blob in blobs:
    print(blob.cx())
```

## 5.4 blob色块对象

blob有多个方法：

- **`blob.rect()`** 返回这个色块的外框——矩形元组**`(x, y, w, h)`**，可以直接在`image.draw_rectangle`中使用。

- **`blob.x()`** 返回色块的外框的**`x坐标（int）`**，也可以通过**`blob[0]`**来获取。

- **`blob.y() `**返回色块的外框的**`y坐标（int）`**，也可以通过**`blob[1]`**来获取。

- **`blob.w() `**返回色块的外框的**`宽度w（int）`**，也可以通过**`blob[2]`**来获取。

- **`blob.h() `**返回色块的外框的**`高度h（int）`**，也可以通过**`blob[3]`**来获取。

- **`blob.pixels()`** 返回色块的**`像素数量（int）`**，也可以通过**`blob[4]`**来获取。

- **`blob.cx()`** 返回色块的外框的**`中心x坐标（int）`**，也可以通过**`blob[5]`**来获取。

- **`blob.cy()`** 返回色块的外框的**`中心y坐标（int）`**，也可以通过**`blob[6]`**来获取。

- **`blob.rotation() `**返回色块的**`旋转角度`**（单位为弧度）（float）。如果色块类似一个铅笔，那么这个值为0-180°。如果色块是一个圆，那么这个值是无用的。如果色块完全没有对称性，那么你会得到0~360°，也可以通过**`blob[7]`**来获取。

- **`blob.code()`** 返回一个16bit数字，每一个bit会对应每一个阈值。举个例子：

  ```python
  blobs = img.find_blobs([red, blue, yellow], merge=True)
  ```

​		如果这个色块是红色，那么它的code就是0001，如果是蓝色，那么它的code就是0010。注意：一个blob可能是合并的，如果是红色		和蓝色的blob，那么这个blob就是0011。这个功能可以用于查找颜色代码。也可以通过**`blob[8]`**来获取。

- **`blob.count()`** 如果`merge=True`，那么就会有多个blob被合并到一个blob，这个函数返回的就是这个的数量。如果`merge=False`，那么返回值总是1。也可以通过**`blob[9]`**来获取。
- **`blob.area()`** 返回色块的外框的**`面积`**。应该等于(w * h)
- **`blob.density()`** 返回色块的**`密度`**。这等于色块的像素数除以外框的区域。如果密度较低，那么说明目标锁定的不是很好。
  比如，识别一个红色的圆，返回的blob.pixels()是目标圆的像素点数，blob.area()是圆的外接正方形的面积。密度就是像素数除以面积，结果在0和1之间。