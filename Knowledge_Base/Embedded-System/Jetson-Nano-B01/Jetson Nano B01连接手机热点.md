# Jetson Nano B01连接网络--通过手机热点

博主是一名大学生，买来这块板子很久了，一直没有用上，为什么呢？因为博主太菜了，根本连不上网络，所以一直无法继续下一步学习，又不想花钱买网卡，然后呢，网络上关于使用网线来连接网络的教程又讲的不是很清楚，所以一直没有弄出来。（因为是小白所以不清楚很多知识）当然，已经不重要了，经过我不懈努力，终于弄出来了，相信你看完这篇教程你会恍然大悟，原来也不是那么难。





## Step1

首先第一步，你需要有一根网线（两端都是RJ45接口，这是买板子时送的）

然后你需要将电脑和板子通过这跟网线连接在一起。



## Step2

打开手机热点，让电脑连接手机热点，此时，下面的步骤都很重要，不能错过一步。

> 1.打开电脑控制面板  $-->$ 网络和Internet $-->$ 网络和共享中心

![image-20241116235014597](https://s2.loli.net/2024/11/16/orhLB6EcnZJ5Ipq.png)

此时，有两个网络，上方为你自己的手机热点，下方那个未识别的网络是我们通过RJ45接口连接的板子那端。

此时，你有多种方式打开我们需要的那个界面，第一个方法

> 2.点击我们的WLAN（就是我们的热点） ，









```python
# 设置超级用户密码
sudo passwd root

# 重启网络服务
sudo /etc/init.d/networking restart 或 sudo systemctl restart networking


# 使用此条命令查看全局代理的地址和端口
env | grep -i proxy

jetson@yahboom:/$ env lgrep -i proxy
HTTP PRoXYhttp://192.168.137.1:7890
FTP PRoXY-http://192.168.137.1:7890/
https proxyhttp://192.168.137.1:7890
http proxy http://192.168.137.1:7890
ALL PRoXY=socks://192.168.137.1:7890
no proxy=localhost,127.0.0.0/8,::1
NO PRoXY=localhost,127.0.0.0/8,::1
HTTPS PRoXY-http://192.168.137.1:7890
all proxy=socks://192.168.137.1:7890/
ftp proxy=http://192,168.137.1:7890/
jetson@yahboom:/$
```




deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
