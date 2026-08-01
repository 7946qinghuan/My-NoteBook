

# ==C语言第四次培训==

## ------选择语句与循环语句

[TOC]



### 一.选择语句：

选择语句分if语句和switch语句：

#### 1.if语句：

上次培训我们已经讲过if语句和他的衍生结构，这次培训我们讲他的升级用法***==“if语句的嵌套”==***。

依然是这个问题，设计程序实现输入整数a,b,c的值，按从小到大的顺序输出，我们用if语句的嵌套该如何解决呢？

我们直接上代码：

```c
#include <stdio.h>

int main()
{
	int a, b, c, t;
	printf("please input three int a,b,c:\n");
	scanf("%d %d %d", &a, &b, &c);
	if(a > b)
	{
		if(b > c)
            printf("three sored int :\n%d %d %d\n", c, b, a);
		else if(a > c)//不满足b>c
            printf("three sored int :\n%d %d %d\n", b, c, a);
    	else//不满足b>c,a>c
			printf("three sored int :\n%d %d %d\n", b, a, c);
	}
	else
	{
		if(a > c)
         	printf("three sored int :\n%d %d %d\n", c, a, b);
		else if(b > c)//不满足a>c
		 	printf("three sored int :\n%d %d %d\n", a, c, b);
		else//不满足a>c,b>c
		 	printf("three sored int :\n%d %d %d\n", a, b, c);
	}
    return 0;
}
```



#### 2.switch语句：

设想有这样一个问题，随机输入一个数字1-7，输入对应的英文星期几。我们当然可以用if语句解决这个问题，但是代码会很长，此时我们便可以采用switch语句。

##### 格式：

```c
switch(表达式)
{
	case 常量1:语句1 break;
	case 常量2:语句2 break;
	default:语句n break;
   
}
//break的作用是跳出整个循环（break语句：当switch语句运行时遇到break关键字时会跳出，意思就是当语句运行到break时就不再运行了，接下来剩下的case语句也不会再执行，switch语句结束。）
```

```c
#include <stdio.h>

int main()
{
	int n;
	scanf("%d", &n);
	switch(n)
	{
		case 1: printf("Monday\n");break;
		case 2: printf("Tuesday\n");break;
		case 3: printf("Wednesday\n");break;
		case 4: printf("Thursday\n");break;
		case 5: printf("Friday\n");break;
		case 6: printf("Saturday\n");break;
		case 7: printf("Sunday\n");break;
		default : printf("输入错误\n");break;
	}
    return 0;
}
```



break可以怎么使用呢？我们通过一个程序来加深理解。

设计一个程序实现输入一个1-7的数字，输出它是工作日还是休息日。

```c
#include <stdio.h>

int main()
{
	int n;
	scanf("%d", &n);
	switch(n)
	{
		case 1: 
		case 2: 
		case 3: 
		case 4: 
		case 5: printf("工作日\n");break;//如果元素之间有相同性质，输出同样的结果
		case 6:
		case 7: printf("休息日\n");break;
		default : printf("输入错误\n");break;
	}
    return 0;
}
```

当然，如同if语句一样，stitch语句同样可以嵌套。我们通过一个程序来加深理解。

设计一个程序实现输入两个数字m，n，m=1为左转；m=2为右转；m=3为十字路口，若此时n=1左直转，n=2右直转，n=其他值加速；m=其他值直行。



```c
#include <stdio.h>

int main()
{
	int m, n;
	scanf("%d%d", &m, &n);
	switch(m)
	{
		case 1 : printf("左转\n");break;
		case 2 : printf("右转\n");break;
		case 3 : 
        {
            switch(n)
            {
                case 1 : printf("左直转\n");break;
                case 2 : printf("右直转\n");break;
                default : printf("加速\n");break;
            }
            break;
        }
		default : printf("直行\n");break;
	}
    return 0;
}
```





### 二.循环语句：

#### 1.while语句：

设想如果设计程序让你计算1到100的所有整数的和你该怎么解决？

如果我们这样解决：

```c
int a = 1, b = 2, ......;
int sum = a + b + ......;
printf("%d", sum);
```

代码太过冗长，所以我们使用while语句来使这种单一的操作简单化；

##### 格式：

```c
while(条件)
{
	执行语句;
    执行语句;
}
```

```c
#include <stdio.h>

int main()
{
    int x = 1, sum = 0;
    while(x != 101)//等价于while(x<101),while(x<=100)
    {
        sum = sum + x;
        x++;
    }
    printf("sum = %d\n", sum);
    return 0;
}

/*代码输出：
sum = 5050

*/
```



#### 2.do while语句：

do while 语句与whle语句的区别是：

while：先判断再执行

do while：先执行再判断

##### 格式：

```c
do
{
	执行语句;
    执行语句;
}while(条件);
```

```c
#include <stdio.h>

int main()
{
    int x = 1, sum = 0;
    do
    {
        sum = sum + x;
        x++;
    }while(x < 101);
    printf("sum = %d\n", sum);
    return 0;
}

/*代码输出：
sum = 5050

*/
```



虽然结果都一样，但我们怎么理解它们的区别呢？

```c
#include <stdio.h>

int main()
{
    int m = 1, n = 1, sum_m = 0, sum_n = 0;
    do
    {
        sum_m = sum_m + m;
        m++;
    }while(m < 1);
    printf("sum_m = %d\n", sum_m);
    printf("m = %d", m);
    putchar('\n');
    while(n < 1)
    {
        sum_n = sum_n + n;
        n++;
    }
    printf("sum_n = %d\n", sum_n);
    printf("n = %d", n);
    return 0;
}

/*代码输出：
sum_m = 1
m = 2
sum_n = 0
n = 1
*/
```

如果一次代码都不执行，我们便会发现端倪。



#### 3.for语句：

##### 格式：

```c
for(表达式1;表达式2;表达式3)
{
    执行语句;
    执行语句;
}
//表达式1：设置初始条件，只会执行一次
//表达式2：设置循环条件，用来判断是否继续循环
//表达式3：设置循环调整
```

同样是计算1-100之间所有整数的和，我们还可以采用for语句：

```c
#include <stdio.h>

int main()
{
    int i, sum = 0;
    for(i = 1; i < 101; i++)
    {
        sum = sum + i;
    }
    printf("sum = %d\n", sum);
	return 0;
}
```



#### 4.循环语句的嵌套：

循环语句之间是可以相互嵌套的。

我们讲最常用的**for语句的嵌套**：

设想有这样一个题第一行输出1到5，用水平制表符隔开，第二行输出第一行的两倍，第三行输出第一行的三倍，以此类推输出五行。

```c
/*代码讲解*/
#include<stdio.h>

int main()
{
 	int i, j, n = 0;
 	for(i = 1; i <= 5; i++)//限定行数5
	 	for(j = 1; j <= 5; j++, n++)//限定每行5个数据 
	 	{
	 		if(n % 5 == 0 && n != 0)
				printf("\n");//控制每行5个数据;换行 
	 		printf("%d\t",i*j);//输出每行的数据;
		}
	printf("\n");
	return 0;
}

/*代码输出：
1       2       3       4       5
2       4       6       8       10
3       6       9       12      15
4       8       12      16      20
5       10      15      20      25

*/
```

如果很难理解，我们可以换一个例子：

```c
#include <stdio.h>

int main()
{
    int i;
    for(i = 1; i <= 5; i++)
        printf("*");
	return 0;
}

/*代码输出：
*****
*/
```



```c
#include <stdio.h>

int main()
{
    int i, j;
    for(i = 1; i <= 5; i++)
        for(j = 1; j <= 5; j++)
        {
        	printf("*");
			if(j == 5 && i != 5)
        		putchar('\n');	
		}
	return 0;
}
/*代码输出：
*****
*****
*****
*****
*****
*/

```



#### 5.改变循环的状态，break与continue语句：

在讲switch语句时，我们已经讲过break：

回顾：break的作用是跳出整个循环（break语句：当switch语句运行时遇到break关键字时会跳出，意思就是当语句运行到break时就不再运行了，接下来剩下的case语句也不会再执行，switch语句结束。）

break：跳出整个循环；

continue：跳出本次循环。

```c
/*代码理解*/
#include <stdio.h>

int main()
{
    int i;
    for(i = 1; i < 11; i++)
    {
        if(i % 2 == 0)
            continue;
        printf("%-3d", i);
    }
    return 0; 
}

/*代码输出：
1  3  5  7  9
*/
```



```c
/*代码理解*/
#include <stdio.h>

int main()
{
    int i;
    for(i = 1; i < 11; i++)
    {
        if(i % 2 == 0)
           break;
        printf("%-3d", i);
    }
    return 0; 
}

/*代码输出：
1  
*/
```



### 三.任务：

1.设计程序实现输入年份，输出是闰年还是平年；

2.设计程序实现输入整数a，b和一个算数运算符，输出它们的计算结果（不管怎么输入，在计算过程中，a必须大于b）；

3.判断100到200之间所有的素数（质数），并输出所有素数；

4.用for循环语句设计程序实现输出一个三角形。每行(2n - 1)个*，共5行。

```
    *
   ***
  *****
 *******
*********
```

（拓展1：菱形）

```
   *
  ***
 *****
*******
 *****
  ***
   *
```

（拓展2：自定义可输入菱形）比如边长为3，符号为&

```
  &
 &&&
&&&&&
 &&&
  &
```



#### 1.设计程序实现输入年份，输出是闰年还是平年；

```c
/*法一*/
#include <stdio.h>

int main()
 {
 	int year, leap;
	printf("请输入年份：");
	scanf("%d", &year);
	if(year % 4 == 0)
	{
		if(year % 100 == 0)
		{
			if(year % 400 == 0)
                leap = 1;
			else
                leap = 0;
		}
		else
            leap = 1;
	}
	 else
         leap = 0;
	 if(leap != 0)//if(leap!=0) = if(leap) 
	 	printf("%d该年份是闰年。\n", year);
	 else
	 	printf("%d该年份是平年。\n", year);
	 return 0; 
}
```



```c
/*法二*/
#include <stdio.h>

int main()
{   int year, a;
 	printf("请输入年份：");
 	scanf("%d", &year);
 	a=((year % 4 == 0 && year % 100 != 0) || year % 400 == 0);
 	//printf("%d", a);
 	if(a)
 		printf("%d该年份是闰年。\n", year);
 	else
 		printf("%d该年份是平年。\n", year);
 	return 0;
}
```



```c
/*法三*/
#include <stdio.h>

int main()
{
	int year, leap;
	printf("请输入年份：");
	scanf("%d", &year);
	if(year % 4 == 0 && year % 100 != 0)
		leap = 1;
	else
		leap = 0;
	if(year % 400 == 0)
		leap = 1;
	else
		leap = 0;
	if(leap)
		printf("%d该年份是闰年。", year);
	else
		printf("%d该年份是平年。", year);
	return 0;
}
```



#### 2.设计程序实现输入整数a，b和一个算数运算符，输出它们的计算结果（不管怎么输入，在计算过程中，a必须大于b）；

```c
/*法一*/
#include <stdio.h>

int main()
{
	int a, b, t, n;
	char c;
	scanf("%d %d %c", &a, &b, &c);
	if(a < b)
	{
		t = a;
		a = b;
		b = t;
	}
	if(c == '+')
		n = 1;
	else if(c == '-')
		n = 2;
	else if(c == '*')
		n = 3;
	else if(c == '/')
		n = 4;
	else
		n = 0;
	switch(n)
	{
		case 1 : printf("%d %c %d = %d\n", a, c, b, a + b); break;
		case 2 : printf("%d %c %d = %d\n", a, c, b, a - b); break;
		case 3 : printf("%d %c %d = %d\n", a, c, b, a * b); break;
		case 4 : printf("%d %c %d = %d\n", a, c, b, a / b); break;
		default : printf("输入有误\n"); break;
	}
	return 0;
}
```



```c
/*法二*/
#include <stdio.h>

int main()
{
	int a, b, t;
	char c;
	scanf("%d %d %c", &a, &b, &c);
	if(a < b)
	{
		t = a;
		a = b;
		b = t;
	}
	if(c == '+')
		printf("%d %c %d = %d\n", a, c, b, a + b);
	else if(c == '-')
		printf("%d %c %d = %d\n", a, c, b, a - b);
	else if(c == '*')
		printf("%d %c %d = %d\n", a, c, b, a * b);
	else if(c == '/')
		printf("%d %c %d = %d\n", a, c, b, a / b);
	else
		printf("输入有误\n");
	return 0;
}
```



```c
/*法三*/
#include <stdio.h>

int main()
{
	int a, b, t;
	char c;
	scanf("%d %d %c", &a, &b, &c);
	if(a < b)
	{
		t = a;
		a = b;
		b = t;
	}
	switch(c)
	{
		case '+' : printf("%d %c %d = %d\n", a, c, b, a + b); break;
		case '-' : printf("%d %c %d = %d\n", a, c, b, a - b); break;
		case '*' : printf("%d %c %d = %d\n", a, c, b, a * b); break;
		case '/' : printf("%d %c %d = %d\n", a, c, b, a / b); break;
		default : printf("输入有误\n"); break;
	}
	return 0;
}
```



#### 3.判断100到200之间所有的素数（质数），并输出所有素数；

```c
#include <stdio.h>

int main()
{
	int i, j, p;
	for(i = 100; i <= 200; i++)
	{
		for(j = 2; j < i; j++)
		{
			if(i % j == 0)
				break;
		}
		if(j == i)
		printf("%-5d", i);
	}
	putchar('\n');
	return 0;
}
```



#### 4.用for循环语句设计程序实现输出一个三角形。每行(2n - 1)个*，共5行；



```
    *
   ***
  *****
 *******
*********
```

（拓展1：菱形）

```
   *
  ***
 *****
*******
 *****
  ***
   *
```



（拓展2：自定义可输入菱形）比如边长为3，符号为&

```
  &
 &&&
&&&&&
 &&&
  &
```



```c
#include <stdio.h>

int main()
{
	int i, m, n;
	for(i = 1; i <= 5; i++)
	{
		for(m = 1; m <= 5-i; m++)
		{
			printf(" ");
		}
		for(n = 1; n <= 2*i-1; n++)
		{
			printf("*");
		}
		if(i != 5)
			printf("\n");
    }
	return 0;
}
```



```c
//拓展1
#include <stdio.h>

int main()
{
	int i, j, k;
	for(i = 1; i <= 4; i++)
	{
		for(j = 1; j <= 4-i; j++)
		{
			printf(" ");
		}
		for(k = 1; k <= 2*i-1; k++)
		{
			printf("*");
		}
		printf("\n");
	}
	for(i = 1; i < 4; i++)
	{
		for(j = 1; j <= i; j++)
		{
			printf(" ");
		}
		for(k = 1; k <= 2*(4-i)-1; k++)
		{
			printf("*");
		}
		if(i != 3)
			printf("\n");
	}
}
```



```c
//拓展2
#include <stdio.h>

int main()
{
	char x;
	int y, i, j, k;
	scanf("%c %d", &x, &y);
	for(i = 1; i <= y; i++)
	{
		for(k = 1; k <= y-i; k++)
			printf(" ");
		for(j = 1; j <= 2*i-1; j++)
		{
			printf("%c", x);
		}
		printf("\n");
	}
	for(i = 2; i <= y; i++)
	{
		for(k = 1; k <= i-1; k++)
			printf(" ");
		for(j = 1; j < 2*(y+1)-2*i; j++)
			printf("%c", x);
		if(i != y)
			printf("\n");
	}
	return 0;
}
```



```c
//拓展2
#include <stdio.h>
#include <math.h>

int main()
{
	int l;//菱形的行数(奇数) 
	int cx, cy;
	int i, j;
    char c;
    scanf("%d %c", &l, &c);
    cx = cy = l / 2 + 1;
	for(i = 1; i <= l; i++)
	{
		for(j = 1; j <= l; j++)
		{
			if((abs(i - cx) + abs(j - cy)) <= l / 2)//计算出曼哈顿距离的绝对值
				putchar(c);
			else putchar(' ');
		}
		if(i != l)
			putchar('\n');
	}
}
```

