---
title: C语言第二次培训
date: 2026-08-01
tags: [Dev, C]
aliases: []
---

# ==C语言第二次培训==

## **------输入与输出函数**

[TOC]



### 一.scanf输入函数

上节课我们学习了printf输出函数，同理也有scanf输入函数

**==scanf函数称为格式输入函数，即按用户指定的格式从键盘上把数据输入到指定的变量之中。==**

**==注意：从键盘的缓冲区拿数据，键盘输入的字符会放入缓冲区，若用户不按回车键，所有放入缓冲区的字符都不会被读。==**

scanf函数是一个标准库函数，它的函数原型在头文件“stdio.h”中。（如同printf函数）

scanf函数的一般形式为：
    **==scanf(“格式控制字符串”, 地址表列);==**

注：地址列表项有些书说的是输入参数列表，说输入参数列表其实不够精确。

实现的功能即：将键盘输入的数据，按照规定的格式，存储到变量所在内存地址内。

其中，格式控制字符串的作用与printf函数相同，但不能显示非格式字符串，也就是不能显示提示字符串。地址表列中给出各变量的地址。地址是由地址运算符“&”后跟变量名组成的。

例如：&a、&b分别表示变量a和变量b的地址。

这个地址就是编译系统在内存中给a、b变量分配的地址。在C语言中，使用了地址这个概念，这是与其它语言不同的。 应该把变量的值和变量的地址这两个不同的概念区别开来。变量的地址是C编译系统分配的，用户不必关心具体的地址是多少。

变量的地址和变量值的关系
在赋值表达式中给变量赋值，如：
    ==**a=3;**==

![int类型](https://img-blog.csdnimg.cn/ac5808b33aac4672941c35fa6f285af6.png)

则，a为变量名，3是变量的值，&a是变量a的地址。

但在赋值号左边是变量名，不能写地址，而scanf函数在本质上也是给变量赋值，但要求写变量的地址，如&a。这两者在形式上是不同的。&是一个取地址运算符，&a是一个表达式，其功能是求变量的地址。



```c
/*代码讲解*/
#include <stdio.h>

int main()
{
    int a,b,c;
    printf("please input a,b,c:\n");
    scanf("%d%d%d",&a,&b,&c);//默认用空格间隔开
    //1 2 3
    /*注意scanf函数里的所有东西都需要用户原封不动的输入*/
    //例：
    //scanf("%d %d %d",&a,&b,&c);
    //1 2 3
    //scanf("%d,%d,%d",&a,&b,&c);
    //1,2,3
    printf("a=%d,b=%d,c=%d",a,b,c);
    //a=1,b=2,c=3
    return 0;
}
```

### 二.什么是*ASCII码值*

| 1.定义：ASCII 码使用指定的7 位或8 位[二进制数](https://baike.baidu.com/item/二进制数)组合来表示128 或256 种可能的[字符](https://baike.baidu.com/item/字符)。标准ASCII 码也叫基础ASCII码，使用7 位[二进制数](https://baike.baidu.com/item/二进制数)（剩下的1位二进制为0）来表示所有的大写和小写字母，数字0 到9、标点符号，以及在美式英语中使用的特殊[控制字符](https://baike.baidu.com/item/控制字符) [1] 。 |
| ------------------------------------------------------------ |

![ASCII码](https://s2.loli.net/2023/08/17/GDLTRAHQ4l3VtbE.png)

**2.常用ASCII码：**

（1）ASCII值为8、9、10 和13 分别转换为**退格**、**制表**、**换行**和**回车**字符。它们并没有特定的图形显示，但会依不同的应用程序，而对[文		  本](https://baike.baidu.com/item/文本)显示有不同的影响；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210530115601805.png)

（2）48～57为0到9十个阿拉伯数字；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210530115629240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81MTI3NjA1Ng==,size_16,color_FFFFFF,t_70)

（3）65～90为26个大写英文字母；

（4）97～122号为26个小写英文字母。



### 三.getchar()和putchar()函数

**1**.定义：getchar()和putchar()是一对字符输入/输出函数。getchar()不带任何参数，getchar()用于读取用户从键盘输入的单个字符。putchar()向终端输出一个字符，其格式为putchar()。getchar()和putchar()函数包含在头文件stdio.h中，使用时需包含此头文件。

```c
/*代码讲解*/
#include<stdio.h>
 
int main()
{
	int ch = getchar();//实际变量ch中放的是读到的那个字符的ASCII码值
	putchar(ch);//putchar接收到一个参数（ASCII码值），输出相对应的字符
	return 0;
}
```

**==2.getchar（）函数为什么用int定义返回值类型。==**

int getchar ( void );

  getchar()函数的返回值类型时整形，当发生读取错误时，返回整型值是-1，把一个负值赋给一个char型的变量是不正确的。当读取正确时，他会返回用户从键盘输的第一个字符的ASCII码值，ASCII码值是数字符号，通过这里也可以看出来getchar()返回值类型应用int定义。

功能：在标准输入缓冲区读取一个字符，返回读取到字符的ASCII码值，如果读取失败返回EOF(-1)。

而EOF就是

```c
#define EOF (-1)
```

那么什么时候会读取失败呢？

比如：ctrl+z

```c
/*代码讲解*/
#include<stdio.h>
 
int main()
{
	char ch = getchar();//getchar()获得一个字符，ch为char类型，则返回字符本身
    printf("%c\n", ch);
    putchar(ch);//putchar接收到一个参数（ASCII码值），输出相对应的字符
    /*
    代码输出：
   ^Z
	
	
	
    */
    int ch = getchar();//getchar()获得一个字符，ch为int类型，则返回字符的ASCII码
    printf("%d\n",ch);
    putchar(ch);//putchar接收到一个参数（ASCII码值），输出相对应的字符
     /*
    代码输出：
    ^Z
	-1
	
    
    */
	return 0;
}
```

**3**.函数getchar（）和函数scanf（）的工作原理：

**工作原理：**

相同 ：

getchar()和scanf()不是直接从键盘上拿数据，他们是从键盘的**==缓冲区==**拿数据，键盘输入的字符会放入缓冲区，***==若用户不按回车键，所有放入缓冲区的字符都不会被读==***。

不同 ：

在用户按下回车键后，缓冲区内会存在’\n’，scanf只会读'\n'之前的字符，不读' \n'和空格.   getchar会将缓冲区的所有字符全部读走，其中包括空格和'\n'。在windows下如果想结束，就输入Ctrl+z,表示EOF。

```c
/*代码实例，输入密码和确认流程*/
//现在暂时不做了解，等到大家熟悉相应知识后再回头来看
#include <stdio.h>

int main()
{
	char password[20] = { 0 };
	printf("请输入密码：>");
	scanf("%s", password);//scanf不是直接从键盘拿数据，scanf的工作原理是：在scanf和键盘之间 
                          //  的输入缓冲区中拿数据，
	                      //输入缓冲区有数据他就拿，没有他就等，当从键盘上输入字符abcdef为了 
                          // 让字符abcdef来到缓冲区
	                      //在键盘上输入\n（回车）字符连同\n一起来到缓冲区,scanf会拿走\n之前 
                          // 的字符abcdef缓冲区剩下\n
	                      //scanf不读空格和回车
	int tmp = 0;
	while ((tmp = getchar()) != '\n')//用来清理缓冲区
	{
		;
	}
	printf("请确认密码(y/n):"); 
	int ch = getchar();  //getchar和scanf的工作原理一样，他会读走缓冲区里剩余的\n,ch里边是\n
	if (ch == 'y')       //getchar和putchar每次只会输入和输出一个字符
	{
		printf("确认成功\n");
	}
	else
	{
		printf("确认失败\n");
	}
	return 0;

}
```



***==注意：putchar()输出指定的字符，不会在输出后自动换行，所以putchar(a)和putchar(b)之间要加putchar('\n')，用作换行。==***

```c
/*代码实列*/
#include <stdio.h>

int main()
{
    int a,b,c;
    printf("请输入字符：\n");
    a = getchar();
    b = getchar();
    c = getchar();
    putchar(a);
    putchar('\n');//注意只能用单引号不能用双引号
    putchar(b);
    putchar('\n');
    putchar(c);
    putchar('\n');
    return 0;
}

/*
代码输出：
请输入字符：
123
1
2
3

*/
```

```c
int __cdecl putchar(int _Ch);
```

在Dev-C++中如果用双引号则发生报错**==[Error] invalid conversion from 'const char*' to 'int' [-fpermissive]==**

putchar()函数的形参为int类型。

针对老版本c/c++编译器而言，调用putchar("\n")会报错，因为"\n"是char * 类型，不可以强制类型转换为int类型（补充：如果没有报错，会报warning!）；

针对现代c/c++编译器而言，调用putchar("\n")，将自动转化为putchar('\n')，输出换行。



### 四.math库常用函数讲解：

#### 1.计算整数x的绝对值用abs()

函数原型：

```c
int abs(int x)
```

使用：

适用整形变量

```c
#include <stdio.h>
#include <math.h>//导入库函数math.h

int main()
{
	int a = -5;
	int b = -2023;
	a = abs(a);//将整形变量a的值变为绝对值再重新赋值给a
	b = abs(b);//将整形变量b的值变为绝对值再重新赋值给b
	printf("a = %d\nb = %d\n", a, b);//输出打印a和b的值
	return 0;
} 
```

拓展：

```c
#include <stdio.h>
#include <math.h>//导入库函数math.h

int main()
{
	float a = -2023.599;
	double b = -5.1321312312;
	//注意点1：单，双精度变量使用整形绝对值函数abs时，会把小数点后所有数去掉 
	a = abs(a);
	b = abs(b);
	printf("a = %f\nb = %lf\n", a, b);//注意点2：输出打印单，双精度变量a和b的值时，只用用对应的格式字符串 
	printf("a = %d\nb = %d\n", a, b);//否则为0;  
	return 0;
} 
```



#### 2.计算双精度[浮点数](https://so.csdn.net/so/search?q=浮点数&spm=1001.2101.3001.7020)x的绝对值用fabs()

函数原型：

```c
double fabs(double x)
```

使用：

适用单精度和双精度变量

```c
#include <stdio.h>
#include <math.h>

int main()
{
	double a = -5.2;//定义双精度浮点数a并赋值 
	double b = -3.1415926;//定义单双精度浮点数b并赋值 
	float c = -1.141199;//定义单精度浮点数c并赋值 
	a = fabs(a);//将双精度浮点数a的值变为绝对值再重新赋值给a  
	b = fabs(b);//将双精度浮点数b的值变为绝对值再重新赋值给b
	//注意：单精度浮点数使用fabs函数时，编译器会自动强转类型;  
	c = fabs(c);//将单精度浮点数c的值变为绝对值再重新赋值给c
	printf("a = %lf\n", a);
	printf("b = %lf\n", b);
	printf("c = %f\n", c);//格式字符串用%f否则为0
	return 0;
} 
```



#### 3.计算x^y^(x的y次方)用pow()

函数原型：

```c
double pow(double x,double y)
```

使用：

适用所有数值型变量（整形，单，双精度）

```c
#include <stdio.h>
#include <math.h>

int main()
{
	int a = 2, b = 3, x;
	float c = 0.5, d = 2, y; 
	x = pow(a, b);//使用pow函数，前者为底后者为幂  
	y = pow(c, d);//使用pow函数，前者为底后者为幂  
	printf("x = %d\n", x);//打印输出x的值;  
	printf("y = %f\n", y);//打印输出y的值; 
	return 0;
} 
```



#### 4.计算e^x^用exp()

函数原型：

```c
double exp(double x)
```

使用：

```c
#include <stdio.h>
#include <math.h>

int main()
{
	int a = 2;
	float b = 0.5; 
	double x, y;
	x = exp(a);//使用exp函数 
	y = exp(b);//使用exp函数 
	printf("x = %lf\n", x);//打印输出x的值;  
	printf("y = %lf\n", y);//打印输出y的值; 
	return 0;
} 
```



#### 5.计算x的平方根用sqrt()

函数原型：

```c
double sqrt(double x)
```

使用：

适用所有数值型变量（整形，单，双精度）

```c
#include <stdio.h>
#include <math.h>

int main()
{
	int a = 16, x;
	float b = 1.21, y;
	x = sqrt(a);//使用exp函数 
	y = sqrt(b);//使用exp函数 
	printf("x = %d\n", x);//打印输出x的值;  
	printf("y = %f\n", y);//打印输出y的值; 
	return 0;
} 
```



#### 6.计算弧度角x的正弦值用sin()

函数原型：

```c
double sin(double x)
```

使用：

```
#include <stdio.h>
#include <math.h>

#define PI 3.1415926//首先需要定义我们的Π 

int main()
{
	int a = 1;
	float b = 0.5;
	double c = sqrt(2)/2;
	printf("%.1f度\n",asin(double(a))*180/PI);//x=1,输出90.0度 
	printf("%.1f度\n",asin(double(b))*180/PI);//x=0.5,输出30.0度 
	printf("%.1f度\n",asin(c)*180/PI);//x=根号2/2,输出45.0度 
	return 0;
} 
```



**课堂回顾：**

1弧度 = 180/pai 度

1度 = pai/180 弧度

1弧度等于57.3度，1弧度等于60弧分，1弧分等于60弧秒，所以1弧秒就是3600分之一弧度，就是0.01592度。

因为：角度180°=π弧度

所以：

1弧度=(180/π)°角度

1角度=π/180弧度

[![img](https://iknow-pic.cdn.bcebos.com/a9d3fd1f4134970a79c77a9c98cad1c8a6865dc8?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_600%2Ch_800%2Climit_1%2Fquality%2Cq_85%2Fformat%2Cf_auto)](https://iknow-pic.cdn.bcebos.com/a9d3fd1f4134970a79c77a9c98cad1c8a6865dc8)

## 扩展资料

根据定义，一周的弧度数为2πr/r=2π，360°角=2π弧度，因此，1弧度约为57.3°，即57°17'44.806''，1°为π/180弧度，近似值为0.01745弧度，周角为2π弧度，平角（即180°角）为π弧度，直角为π/2弧度。

在具体计算中，角度以弧度给出时，通常不写弧度单位，直接写值。最典型的例子是三角函数，如sin 8π、tan (3π/2)。

在初中数学中，我们学过圆弧长公式：

弧长=nπr/180，在这里n就是角度数，即圆心角n所对应的弧长。

但如果我们利用弧度的话，以上的式子将会变得更简单：（注意，弧度有正负之分）

l=|α| r，即α的大小与半径之积。



使用：

```c
#include <stdio.h>
#include <math.h>

#define PI 3.1415926//首先需要定义我们的Π 

int main()
{
	int a = 90;//90°角
	int b = 180;//180°角
	int c = 270;//270°角
	double x, y, z;
	double x_1, y_1, z_1; 
	x = sin(a*PI/180);//使用sin函数，单位为角度所以*PI/180
	y = sin(b*PI/180);//使用sin函数 
	z = sin(c*PI/180);//使用sin函数 
	x_1 = sin(PI/2);//sin(90°)
	y_1 = sin(PI);//sin(180°)
	z_1 = sin((3/2.0)*PI);//sin(270°)
	printf("x = %.2lf\n", x);//打印输出x的值;  
	printf("y = %.2lf\n", y);//打印输出y的值; 
	printf("z = %.2lf\n", z);//打印输出z的值; 
	printf("x_1 = %.2lf\n", x_1);//打印输出x_1的值;  
	printf("y_1 = %.2lf\n", y_1);//打印输出y_1的值; 
	printf("z_1 = %.2lf\n", z_1);//打印输出z_1的值; 
	return 0;
} 
```



#### 7.计算弧度角x的余弦值用cos()

函数原型：

```c
double cos(double x)
```

使用：

```c
#include <stdio.h>
#include <math.h>

#define PI 3.1415926//首先需要定义我们的Π 

int main()
{
	int a = 90;//90°角
	int b = 180;//180°角
	int c = 270;//270°角
	double x, y, z;
	double x_1, y_1, z_1; 
	x = cos(a*PI/180);//使用cos函数，单位为角度所以*PI/180
	y = cos(b*PI/180);//使用cos函数 
	z = cos(c*PI/180);//使用cos函数 
	x_1 = cos(PI/2);//cos(90°)
	y_1 = cos(PI);//cos(180°)
	z_1 = cos((3/2.0)*PI);//cos(270°)
	printf("x = %.2lf\n", x);//打印输出x的值;  
	printf("y = %.2lf\n", y);//打印输出y的值; 
	printf("z = %.2lf\n", z);//打印输出z的值; 
	printf("x_1 = %.2lf\n", x_1);//打印输出x_1的值;  
	printf("y_1 = %.2lf\n", y_1);//打印输出y_1的值; 
	printf("z_1 = %.2lf\n", z_1);//打印输出z_1的值; 
	return 0;
} 
```



#### 8.计算弧度角x的正切值用tan()

函数原型：

```c
double tan(double x)
```

使用：

```c
#include <stdio.h>
#include <math.h>

#define PI 3.1415926//首先需要定义我们的Π 

int main()
{
	int a = 30;//30°角
	int b = 45;//45°角
	int c = 135;//135°角
	double x, y, z;
	double x_1, y_1, z_1; 
	x = tan(a*PI/180);//使用tan函数，单位为角度所以*PI/180
	y = tan(b*PI/180);//使用tan函数 
	z = tan(c*PI/180);//使用tan函数 
	x_1 = tan(PI/6);//tan(30°)
	y_1 = tan(PI/4);//tan(45°)
	z_1 = tan((3/4.0)*PI);//tan(135°)
	printf("x = %.2lf\n", x);//打印输出x的值;  
	printf("y = %.2lf\n", y);//打印输出y的值; 
	printf("z = %.2lf\n", z);//打印输出z的值; 
	printf("x_1 = %.2lf\n", x_1);//打印输出x_1的值;  
	printf("y_1 = %.2lf\n", y_1);//打印输出y_1的值; 
	printf("z_1 = %.2lf\n", z_1);//打印输出z_1的值; 
	return 0;
} 
```



#### 9.计算x的反正弦值用asin()—反正弦函数,如数学中的arsin(x),x的范围为[-1,1]

函数原型:

```c
double asin(double x)  //返回值:弧度值
```

使用：

```c
#include <stdio.h>
#include <math.h>

#define PI 3.1415926//首先需要定义我们的Π 

int main()
{
	int a = 1;
	float b = 0.5;
	double c = sqrt(2)/2;
	printf("%.1f度\n",asin(double(a))*180/PI);//x=1,输出90.0度 
	printf("%.1f度\n",asin(double(b))*180/PI);//x=0.5,输出30.0度 
	printf("%.1f度\n",asin(c)*180/PI);//x=根号2/2,输出45.0度 
	return 0;
} 
```



#### 10.计算x的反余弦值用acos()—反余弦函数,如数学中的arcos(x),x的范围为[-1,1]

函数原型：

```c
double acos(double x)//返回值:弧度值
```



```c
#define PI 3.1415926
printf("%.1f度\n",acos(double(x))*180/PI);//x=-1,输出180.0度
```



#### 11.计算x的反正切值用atan()—反正切函数,如数学中的artan(x)

函数原型:

```c
double atan(double x)//返回值:弧度值
```



```c
#define PI 3.1415926
printf("%.1f度\n",atan(double(x))*180/PI);//x=-1,输出-45.0度
```



#### 12.对x进行四舍五入用round()

函数原型:

```c
double round(double x)
```

使用：

```c
#include <stdio.h>
#include <math.h>

int main()
{
	float a = 1.1;
	float b = 0.524234;
	double c = sqrt(2)/2;
	printf("%f\n", round(a));
	printf("%f\n", round(b));
	printf("%1f\n", round(c));
	putchar('\n');
	printf("%.2f\n", a);
	printf("%.2f\n", b);
	printf("%1f\n", c);
	return 0;
} 
```



#### 13.计算ln(x)用log()

函数原型:

```c
double log(double x)
```



#### 14.计算log10(x)用log10() ----以10为底的对数

函数原型:

```c
double log10(double x)
```



#### 15.计算x/y的余数用fmod()

```c
double fmod(double x,double y)
```



#### 16.返回不大于x的最大整数值用floor()

函数原型:

```c
double floor(double x)
```



#### 17.返回不小于x的最小整数值用ceil()

函数原型:

```c
double ceil(double x)
```



#### 18.返回一个数的小数部分用modf()

函数原型:

```c
double modf(double x,double* integer)//设置integer为整数部分
```

使用：

```c
#include <stdio.h>
#include <math.h>

int main()
{
	double x = 12.345;
	double integer;
	double small = modf(x, &integer);
	printf("%f\n%.0f\n", small, integer);
	return 0;
}
/*
输出:
0.345000
12
*/
```





### 五.任务：

1.设计程序实现输入大写A输出a；

2.设计程序实现China变Glmre并输出；

3.设计程序实现分三行输入 3 个浮点数，表示三角形的三个边长a、b、c 的长度，计算并依次输出三角形的周长和面积，结果严格保留2位小数。输入的数据要保证三角形三边可以构成三角形。

三角形面积计算公式： 

![img](https://img-blog.csdnimg.cn/img_convert/dffd0b1e60065d01d256ad172db8b096.png)

其中

```c
s = (a+b+c)/2
```



4.设计程序实现：

输入r，h的值：

输出圆的周长，面积；球的表面积，体积；圆柱的表面积，体积。



```c
#include<stdio.h>

int main ()
{
	char a;
	scanf("%c",&a);
	//a=(a>='A' && a<='Z')?(a+32):a;
    if(a>='A' && a<='Z')
    {
        a = a+32;
        printf("%c",a);
    }
	else
        printf("输入错误\n");
	return 0;
} 
```



```c
#include<stdio.h>

int main()
{
 	char a1='C',a2='h',a3='i',a4='n',a5='a';
 	a1=a1+4;
 	a2=a2+4;
 	a3=a3+4;
 	a4=a4+4;
 	a5=a5+4;
 	printf("password is %c%c%c%c%c\n",a1,a2,a3,a4,a5);
 	return 0;
}
```



```c
#include <stdio.h>
#include <math.h>

int main() 
{
	float a, b, c, l, area, s;
	scanf("%f\n%f\n%f", &a, &b, &c);
	l = a + b + c;
	s = (a + b + c) / 2;
	area = sqrt(s*(s-a)*(s-b)*(s-c));
	printf("三角形的周长为：%.2f\n", l);
	printf("三角形的面积为：%.2f", area);
}
```



```c
#include<stdio.h>

#define PI 3.1415926
int main()
{
	float r,h,c,s,v,V,v1,S;
	scanf("%f %f",&r,&h);
	c = 2*r*PI;
	s = PI*r*r;
	v = 4*PI*r*r;
	V = 4/3*PI*r*r*r;
	v1 = s*h;
    S = c*h+2*s;
	printf("圆的周长c=%0.2f\n圆的面积s=%0.2f\n圆球的表面积v=%0.2f\n圆球的体积V=%0.2f\n圆柱的表面积S=%.2f\n圆柱的体积v1=%.2f\n", c, s, v, V, S, v1);
//	printf("圆的周长c=%.2f\n", c);
//	printf("圆的面积s=%0.2f\n", s);
//	printf("圆的表面积v=%0.2f\n", v);
//	printf("圆球的体积V=%0.2f\n", V);
//	printf("圆柱的表面积S=%.2f\n", S);
//	printf("圆柱的体积v1=%.2f\n", v1);
	return 0;
}
```

