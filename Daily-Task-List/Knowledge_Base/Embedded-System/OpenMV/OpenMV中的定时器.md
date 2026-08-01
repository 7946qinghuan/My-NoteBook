# OpenMV中的定时器

## 一.介绍

每个定时器都包含一个以某一比率计数的计数器。其计数的频率为外设时钟频率（Hz为单位）除以定时器预分频器。 当计数器到达定时器周期时，会触发事件，且计数器重置为0。通过使用回调函数，定时器事件可调用一个Python函数。



一个以固定频率切换LED用法的简单示例:

```python
tim = pyb.Timer(4)              # create a timer object using timer 4  使用定时器4创建一个定时器对象
tim.init(freq=2)                # trigger at 2Hz 以2Hz触发，也就是0.5s触发一次
tim.callback(lambda t:pyb.LED(1).toggle())  # 调用定时器回调函数触发事件
```



## 二.Timer类

### 1.导入Timer库

```python
# 第一种方法：直接导入定时器库
from pyb import Timer  # 导入定时器库
#time = Timer(4)

# 第二种方法导入所有外设库，然后调用定时器库
import pyb
#time = pyb.Timer(4)
```



### 2.定时器初始化

```
Timer.init(\*, freq, prescaler, period)
```

```
注意:您必须指定频率或周期和分频数。\*代表其他参数，需要时再填，不需要则不填
```

初始化参数分别是：

> 1.`freq`（频率）:
>
> 指定定时器的周期性频率。这里的`freq`参数只能填整数，所以这里只能够计数1秒及以下的时间，`T=1/freq`。
>
> >由于freq不能计数1秒以上的时间，所以我们可以通过`prescaler`和`period`来实现。
>
> 
>
> 2.`prescaler`（PSC预分频器值）: 
>
> `[0-0xffff]` –指定要加载到定时器的PSC中的值，定时器时钟源除以（ `prescaler + 1` ）以得出定时器时钟。
>
> >定时器的时钟源通过`Timer.source_freq()`函数可以得知
>
> 
>
> 3.`period`（ARR自动重装载值）:
>
> `[0-0xffff]` 用于定时器1、3、4、6-15。`[0-0x3fffffff]`用于定时器2和5。
>
> >指定要加载到定时器的ARR中的值。该值决定定时器的周期（即当计数器循环时）。定时器将在 `period + 1` 定时器时钟循环后滚动。
>
> 
>
> 4.`mode`（定时器计数模式选择）:
>
> >- `Timer.UP` - 将定时器配置为从0至ARR（默认）
> >- `Timer.DOWN` - 将定时器配置为从ARR至0。
> >- `Timer.CENTER` - 将定时器配置为从0至ARR再到0。
>
> 
>
> 5.`div` :
>
> 可为1或2或4。划分定时器时钟，以确定数字滤波器所使用的采样时钟。
>
> 
>
> 6.`callback`：在回调函数里调用要做的事，比如一个函数，或者点灯等一些的操作。
>
> 
>
> 7.`deadtime` :
>
> 指定`dead`的数量或在免费通道（两通道须为非活动通道）间转换的无效时间。
>
> >`deadtime` :可为一个介于0-1008间的整数，并满足下列要求：0-128按照步骤1进行，128-256按照步骤2进行，256-512按照步骤8进行，512-1008按照步骤16进行。 `deadime` - 测量以div时钟滴答划分的 `source_freq` 滴答。 `Deadtime` - 仅适用于定时器1和8。
>
> 



### 3.定时器的简单用法

```python
import sensor, image, time
import pyb

sensor.reset()
sensor.set_pixformat(sensor.RGB565)
sensor.set_framesize(sensor.QVGA)
sensor.skip_frames(time = 2000)
clock = time.clock()

led = pyb.LED(1)    # 创建一个LED对象
tim = pyb.Timer(4)  # 使用定时器4创建一个定时器对象

def tick(timer):                # 当被调用时，我们会接收一个定时器对象
    print(timer.counter())      # 显示当前定时器的计时值

#tim.init(freq=2)  # 以2Hz触发,也就是0.5s触发一次
#tim.callback(lambda t:led.toggle())  # 回调函数无论放在哪里都会自行按照指定周期调用

# PSC = 24000
# ARR = 10000
# F = (24000*10000)/240000000=1Hz
# T = 1/F = 1s
# mode = tim.DOWN 指定向下计数，默认向上计数
# callback = lambda t:led.toggle() 回调函数里翻转LED灯状态，这是其实并不是初始化，而是在调用定时器回调函数
tim.init(prescaler=23999, period=9999, mode=tim.DOWN, callback=lambda t:led.toggle())  

while(True):
    clock.tick()
    img = sensor.snapshot()
#    tim.callback(lambda t:led.toggle())
#    print(clock.fps())
#    tim.callback(tick)              # set the callback to our tick function 将回调设置为tick函数

```

在上面的实例中我们可以发现在定时器的回调函数里，我们使用匿名函数使LED灯闪烁。当然，我们也可以在回调函数里调用自定义函数。

```python
import sensor, image, time
import pyb

sensor.reset()
sensor.set_pixformat(sensor.RGB565)
sensor.set_framesize(sensor.QVGA)
sensor.skip_frames(time = 2000)
clock = time.clock()

led = pyb.LED(1)    # 创建一个LED对象
tim = pyb.Timer(4)  # 使用定时器4创建一个定时器对象

def tick(timer):                # 当被调用时，我们会接收一个定时器对象
    print(timer.counter())      # 显示当前定时器的计时值

# PSC = 24000
# ARR = 10000
# F = (24000*10000)/240000000=1Hz
# T = 1/F = 1s
# mode = tim.DOWN 指定向下计数，默认向上计数
tim.init(prescaler=23999, period=9999, mode=tim.DOWN)  

while(True):
    clock.tick()
    img = sensor.snapshot()
    tim.callback(tick)              # set the callback to our tick function 将回调设置为tick函数
#    print(clock.fps())
```



### 4.常用定时器函数

#### ①.`Timer.init()`

```python
Timer.init()(\*, freq, prescaler, period)
```

上面已经详细讲过



#### ②.`Timer.deinit()`

```python
Timer.deinit()
```

反初始化定时器。

禁用回调（以及关联的中断请求）。

禁用任何通道回调（以及关联的中断请求）。停用定时器，并禁用定时器外围设备。



#### ③.`Timer.callback(fun)`

```python
Timer.callback(fun)
```

设置定时器触发时所调用的函数。 `fun` 是被传递的参数，即定时器对象。若 `fun` 为 `None` ，则禁用回调。



#### ④.`Timer.channel(channel,mode,...)`

若只有一个通道被传递，则返回一个先前初始化的通道对象（若无先前通道，则 `None` ）。

另外，初始化并返回一个定时器通道对象。

每一通道都可配置来进行脉宽调制、输出比较和输入捕捉。所有通道公用同一基本定时器，即共用同一定时器时钟。

```python
这个函数暂时不做了解
```



#### ⑤.Timer.counter([value])

```python
Timer.counter([value])
```

获取或设置定时器。



#### ⑥.Timer.freq([value])

```python
Timer.freq([value])
```

获取或设置定时器频率（改变分频数与周期）。



#### ⑦.Timer.period([value])

```python
Timer.period([value])
```

获取或设置定时器周期。



#### ⑧.Timer.prescaler([value])

```python
Timer.prescaler([value])
```

获取或设置定时器分频数。



#### ⑨.Timer.source_freq()

```
Timer.source_freq()
```

获取时钟源的频率。



#### ⑩.函数实践：

```python
import sensor, image, time
import pyb

sensor.reset()
sensor.set_pixformat(sensor.RGB565)
sensor.set_framesize(sensor.QVGA)
sensor.skip_frames(time = 2000)
clock = time.clock()

led = pyb.LED(1)    # 创建一个LED对象
tim = pyb.Timer(4)  # 使用定时器4创建一个定时器对象

def tick(timer):                # 当被调用时，我们会接收一个定时器对象
    print(timer.counter())      # 显示当前定时器的计时值

# PSC = 24000
# ARR = 10000
# F = (24000*10000)/240000000=1Hz
# T = 1/F = 1s
# mode = tim.DOWN 指定向下计数，默认向上计数
tim.init(prescaler=23999, period=9999, mode=tim.DOWN)  

while(True):
    clock.tick()
    img = sensor.snapshot()
    string = "source_freq:%dHz, ARR=%d, PSC=%d, F=%dHz, T=%ds, CNT=%d"\
    %(tim.source_freq(), tim.period(), tim.prescaler(), tim.freq(), \
    1/tim.freq(), tim.counter())
    print(string)
```
