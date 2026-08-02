---
title: C语言第六次培训
date: 2026-08-01
tags: [Dev, C]
aliases: []
---

# ==C语言第六次培训==

## ------函数



[TOC]



### 一.八大字符串处理函数:

#### 1.puts函数：

```C
#include <stdio.h>

int main()
{
    char c[20] = {"I'm waiting!"};
    char word[10] = {"Sunset"}; 
    printf("%s", c);
    puts(c);
    puts(word);
    return 0;
}
/*代码输出：
I'm waiting!
Sunset

*/
```

puts函数可以直接输出字符串。

注意：puts函数将末尾空字符"\0"自动转换为换行符"\n"



#### 2.gets函数：

```c
#include <stdio.h>

int main()
{
    char c[20];
    gets(c);
    puts(c);
    return 0;
}
```

gets函数可以直接输入字符串，并自动在末尾添加空字符"\0"作为结束标志。

当然，gets函数并不完美，虽然用 gets() 时有空格也可以直接输入，但是 gets() 有一个非常大的缺陷，即它==不检查预留存储区是否能够容纳实际输入的数据==，换句话说，**如果输入的字符数目大于数组的长度，gets 无法检测到这个问题，就会发生内存越界**，所以编程时建议使用 fgets。

**fgets() 的原型为：**

\# include <stdio.h>
char *fgets(char *s, int size, FILE *stream);

fgets() 虽然比 gets() 安全，但安全是要付出代价的，代价就是它的使用比 gets() 要麻烦一点，有三个参数。它的功能是从 stream 流中读取 size 个字符存储到字符指针变量 s 所指向的内存空间。它的返回值是一个指针，指向字符串中第一个字符的地址。  

其中：s 代表要保存到的内存空间的首地址，可以是字符数组名，也可以是指向字符数组的字符指针变量名。size 代表的是读取字符串的长度。stream 表示从何种流中读取，可以是标准输入流 stdin，也可以是文件流，即从某个文件中读取，这个在后面讲文件的时候再详细介绍。标准输入流就是前面讲的输入缓冲区。所以如果是从键盘读取数据的话就是从输入缓冲区中读取数据，即从标准输入流 stdin 中读取数据，所以第三个参数为 stdin。

下面写一个程序：

```c
# include <stdio.h>

int main(void)
{
	char str[20]; /*定义一个最大长度为19, 末尾是'\0'的字符数组来存储字符串*/
	printf("请输入一个字符串:");
	fgets(str, 7, stdin); /*从输入流stdin即输入缓冲区中读取7个字符到字符数组str中,注意能够储存6个，第7个为'\0'*/
	printf("%s\n", str);
	return 0;
}
```

输出结果是：
请输入一个字符串:i love you
i love

我们发现输入的是“i love you”，而输出只有“i love”。原因是 fgets() 只指定了读取 7 个字符放到字符数组 str 中。“i love”加上中间的空格和最后的 '\0' 正好是 7 个字符。

那有人会问：“用 fgets() 是不是每次都要去数有多少个字符呢？这样不是很麻烦吗？”不用数！fget() 函数中的 size 如果小于字符串的长度，那么字符串将会被截取；如果 size 大于字符串的长度则多余的部分系统会自动用 '\0' 填充。所以假如你定义的字符数组长度为 n，那么 fgets() 中的 size 就指定为 n–1，留一个给 '\0' 就行了。

例：

```c
#include <stdio.h>

int main()
{
	char arr[5];
	char *p = arr;
	//按照上面的叙述，这里4赋值给fget中间的参数size，3个数据位，一个'\0'，数组长度为5剩一个补位'\0'
	fgets(p,4,stdin);
    //fgets(arr,4,stdin);//数组的首地址就是指针
	for(int i = 0;i != 5;i++)
	{
		if(arr[i] == '\0')
			printf("\\0");
		else if(arr[i] == '\n')
			printf("\\n");
		else
			printf("%c",arr[i]);
	}
		return 0;
}
```

后面的函数都需要导入库<string.h>

#### 3.strcat函数：

格式：strcat(字符数组1, 字符数组2)

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str1[20] = {"exagge"};
    char str2[20] = {"rate"};
    printf("%s", strcat(str1, str2));
    return 0;
}

/*代码输出：
exaggerate
*/
```

注意strcat函数在string.h库中

strcat函数全名"string catenate",作用是字符串连接,将str2连接到str1后面，所以这就要求str1足够大，要能够容纳两个字符串。

所以我们规范应该这样写:

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str1[30] = {"exagge"};
    char str2[5] = {"rate"};
	strcat(str1, str2);
    printf("%s\n", str1);
    return 0;
}

/*代码输出：
exaggerate

*/
```

**==注意：strcat函数不会检查str1数组的长度；==**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str1[2] = {"a"};
    char str2[10] = {"bcdef"};
	strcat(str1, str2);
    printf("%s\n", str1);
    return 0;
}

/*代码输出：
abcdef

*/
```

对于str1数组的字符串，str1数组能够容纳它本身，由于在使用strcat函数时不会检查str1数组的长度，所以代码不会报错。那这是正确的还是错误的呢？

这是错误的，为什么呢？ 

```c
#include <stdio.h>
#include <string.h>

int main()
{
	int x, y;
    char str1[2] = {"a"};
    char str2[10] = {"bcdef"};
	strcat(str1, str2);
    printf("%s\n", str1);
    x = strlen(str1);//计算长度
    y = sizeof(str1);//计算占用内存大小
    printf("%d\n",x);
    printf("%d\n",y);
    printf("%c\n", str1[5]);
    return 0;
}

/*代码输出：
abcdef
6
2
f

*/
```

我们发现，计算出来的字符串长度为6，但是数组占用的内存是2个字节，代表系统分配给你的合法空间就是2个字节，但是你擅自更改了后面的空间，属于数组越界，是非法操作，编译器也许会报错，也许不会报错，但最终都可能导致意想不到的后果！

也就是说在一定范围内越界，还能正常输出，但是超过这个范围则不能输出。而越界则是占用了其他内存的空间。

```C
#include <stdio.h>
int main()
{	
    int a[10], i;
	for(i = 1; i <= 11 ;i++)
    {
		a[i] = 0;
		printf("%d %d\n",i,a[i]);
	}
    return 0;
}
```

这里只给了a数组a[0]~a[9]这10个空间，但是循环会非法访问到a[10]和a[11]，而a[11]这个空间恰好是i的空间，在修改a[11] = 0时，就相当于把i = 0了，所以会造成死循环。



#### 4.strcpy函数：

格式：strcpy(字符数组, 字符数组/字符串)

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str1[20] = {"Monday"};
    char str2[20] = {"Tuesday"};
    char str3[20] = {"Wednesday"};
    printf("%s\n", strcpy(str1, str2));
    printf("%s\n", strcpy(str3, "Thursday"));
    //等同于
    /*
    strcpy(str1, str2);
    strcpy(str3, "Thursday");
    printf("%s\n", str1);
    printf("%s\n", str3);
    */
    return 0;
}

/*代码输出：
Tuesday
Thursday

*/
```

strcpy函数全名"string copy",作用是将str2的字符串复制到str1里面，所以这就要求str1不低于str2的长度。



#### 5.strcmp函数：

 格式：strcpy(字符数组/字符串, 字符数组/字符串)

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str1[20] = {"Monday"};
    char str2[20] = {"Tuesday"};
    char str3[20] = {"Monday"};
    int x = strcmp(str1, str2);
    if(x == 0)
    	printf("字符串1与字符串2相同\n");
    else if(x > 0)
    	printf("字符串1比字符串2大\n");
    else if(x < 0)
    	printf("字符串1比字符串2小\n");
    return 0;
}

/*代码输出：
字符串1比字符串2小

*/
```

strcpy函数全名"string compare",作用是==比较==字符串str1与字符串str2==第一个不同字符的大小==，大于返回正整数，小于返回负整数，若全部相同则返回0。



#### 6.strlen函数：

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str1[20] = {"Monday"};
    char str2[20] = {"Tuesday"};
    char str3[20] = {"Wednesday"};
    printf("%d\n", strlen(str1)); 
    printf("%d\n", strlen(str2)); 
    printf("%d\n", strlen(str3)); 
    return 0;
}

/*代码输出：
6
7
9

*/
```

strlen函数全名"string length",作用是计算字符串长度。



#### 7.strlwr函数：

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str1[20] = {"Monday"};
    char str2[20] = {"Tuesday"};
    char str3[20] = {"Wednesday"};
    printf("%s\n", strlwr(str1)); 
    printf("%s\n", strlwr(str2)); 
    printf("%s\n", strlwr(str3)); 
    return 0;
}

/*代码输出：
monday
tuesday
wednesday

*/
```

strlwr函数全名"string lowercase",作用是将字符串中全部大写字母转换为小写。



#### 8.strupr函数：

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str1[20] = {"Monday"};
    char str2[20] = {"Tuesday"};
    char str3[20] = {"Wednesday"};
    printf("%s\n", strupr(str1)); 
    printf("%s\n", strupr(str2)); 
    printf("%s\n", strupr(str3)); 
    return 0;
}

/*代码输出：
MONDAY
TUESDAY
WEDNESDAY

*/
```

strlwr函数全名"string uppercase",作用是将字符串中全部小写字母转换为大写。



### 二.什么是函数？为什么要用函数？怎么用函数？

函数是经过我们封装好的一个功能函数，不同的函数实现不同的功能。随着程序功能变得越来越复杂，我们的代码会变得越来越多，为了减轻代码的冗杂性，我们便封装不同的函数从而实现不同的功能。

概念：函数是一个可以反复执行的程序段。在一个程序中，如果需要多次执行某项功能或操作，则可 以把完成该功能或操作的程序段从程序中独立出来定义为函数，而原来程序中需要执行该功能或操作时可以通过函数调用来代替，以达到简化程序的目的。

格式： 函数类型+函数名(形参1，形参2，......)也可以没有形参或函数类型为空

```
数据类型符 函数名(形式参数表) 
{ 
	数据定义语句部分; 
	执行语句部分; 
}
```



```c
int return_sum()
{
	int x, y, z, sum;
	scanf("%d %d %d", x, y, z);
	sum = x+y+z;
	return sum;
}
```



```c
#include <stdio.h>

int return_sum()//无参函数
{
	int x, y, z, sum;
	scanf("%d %d %d", &x, &y, &z);
	sum = x+y+z;
	return sum;
}

int main()
{
    int x = return_sum();
	printf("%d", x);  
    return 0;
}

/*代码输出：
1 2 3
6
*/
```



```c
int return_sum(int x, int y, int z)
{
	int sum;
	sum = x+y+z;
	return sum;
}
```



```c
#include <stdio.h>

int return_sum(int x, int y, int z)//形参与实参一一对应
{
	int sum;
	sum = x+y+z;
	return sum;
}

int main()
{
    int a, b, c, sum;
    scanf("%d %d %d", &a, &b, &c);
    sum = return_sum(a, b, c);
	printf("%d",sum);  
    return 0;
}

/*代码输出：
1 2 3
6
*/
```



```c
void return_sum(int x, int y, int z)
{
	int sum;
	sum = x+y+z;
	printf("%d", sum);
}
```



```c
#include <stdio.h>

void return_sum(int x, int y, int z)//无返回值函数，也就是空函数
{
	int sum;
	sum = x+y+z;
	printf("%d", sum);
}

int main()
{
    int a, b, c, sum;
    scanf("%d %d %d", &a, &b, &c);
    return_sum(a, b, c);
    return 0;
}

/*代码输出：
1 2 3
6
*/
```

**注意：若函数定义在主函数后面，则需要在主函数里声明。**

```c
#include <stdio.h>

int main()
{
	void return_sum(int x, int y, int z);
    int a, b, c, sum;
    scanf("%d %d %d", &a, &b, &c);
    return_sum(a, b, c);
    return 0;
}

void return_sum(int x, int y, int z)//无返回值函数，也就是空函数
{
	int sum;
	sum = x+y+z;
	printf("%d", sum);
}
```

我们来做题练练手：

1.设计一个程序实现输入整数a,b的值，输出更大的那个数。

```c
#include <stdio.h>

int find_max(int x, int y)
{
	int max;
	if(x > y)
		max = x;
	else
		max = y;
    //max = (x>y)?x:y;
	return max;
}

int main()
{
    int a, b, max;
    scanf("%d %d", &a, &b);
    max = find_max(a, b);
    printf("%d", max);
    return 0;
}
```



**注意：函数的返回值只能有一个（常量或表达式），但可以有多个 return 语句，一旦执行到其中一个 return 语句，则立即返回主调函数，被调函数中的其他语句不再执行。**

例如：

```c
#include <stdio.h>

int find_max(int x, int y)
{
	
	if(x > y)
		return x;
	else
		return y;
}

int main()
{
    int a, b, max;
    scanf("%d %d", &a, &b);
    max = find_max(a, b);
    printf("%d", max);
    return 0;
}
```



```c
#include <stdio.h>

int f(int x, int y)
{ 
	return(x + y);
	return(x * y);
	 
}
int main(void)
{
	int a;
	a = f(1, 2);
	printf("%d\n", a);
}
```



如同条件判断和循环，函数也能实现嵌套，我们需要注意的是嵌套的逻辑，不要嵌套混乱了。

2.设计程序实现输入4个整数，输出最大者(使用函数的嵌套)

```c
#include <stdio.h>

int find_max_2(int x, int y)
{
	
	if(x > y)
		return x;
	else
		return y;
}

int find_max_4(int a, int b, int c, int d)
{
//	if(find_max_2(a,b)>find_max_2(c,d))
//		return find_max_2(a,b);
//	else
//		return find_max_2(c,d);
	int max = find_max_2(a,b) > find_max_2(c,d) ? find_max_2(a,b) : find_max_2(c,d);
	return max;
}

int main()
{
    int a, b, c, d, max;
    scanf("%d %d %d %d", &a, &b, &c, &d);
    max = find_max_4(a, b, c, d);
    printf("max = %d", max);
    return 0;
}
```



### 三.函数的递归：

#### 1.递归：

递归就是一个函数在它的函数体内**调用它自身**。执行递归函数将反复调用其自身，每调用一次就进入新的一层。递归函数必须有**结束条件**。当函数在一直递推，直到遇到墙后返回，这个墙就是**结束条件**。所以递归要有两个要素，==结束条件与递推关系==



**注:**递归的时候，每次调用一个函数，计算机都会为这个函数分配新的空间，这就是说，当被调函数返回的时候，调用函数中的变量依然会保持原先的值，否则也不可能实现反向输出。



#### 2.理解递归：  

例题1：设计程序实现输入n计算n的阶乘并输出。

```c
#include <stdio.h> 

int factorial(int n)
{
    int result;
    if (n < 0)                                          //判断例外
    {
        printf("输入错误!\n");
        return 0;
    } 
    else if (n == 0 || n == 1)
    {
        result = 1;  //回推墙
    }
    else
    {
        result = factorial(n-1) * n;  //递推关系，这个数与上一个数之间的关系。
    }
    return result;
}

int main()
{
    int n = 5;                                              //输入数字5，计算5的阶乘
    printf("%d的阶乘=%d", n, factorial(n));
    return 0;
}

//代码输出：
/*
5的阶乘=120
*/
```



程序在计算5的阶乘的时候，先执行递推，当n=1或者n=0的时候返回1，再回推将计算并返回。由此可以看出递归函数必须有**结束条件**。



例题2.设计程序实现输入n输出斐波那契的第n项。

```c
#include <stdio.h>

long fibonacci( long num )
{
    if ( num == 0 || num == 1 )
    {
        return num;
    }
    else
    {
        return fibonacci( num -1 ) + fibonacci( num -2 );
    }
}
    
main()
{
    long number;
    puts("请输入一个正整数: ");
    scanf("%ld", &number);
    printf("斐波那契数列第%ld项为: %ld\n", number, fibonacci( number ) );

}

//代码输出：
/*
请输入一个正整数:
20
斐波那契数列第20项为: 6765
*/
```



![斐波那契](https://img-blog.csdnimg.cn/img_convert/65f2ab6278d63d6da80a27eb4a033c59.png)

怎么理解呢？递就是我们要求斐波那契的第5项，就要求第4项和3项，要求第4项就要求第3项和第2项，要求第3项就要求第2项和第1项，以此类推，最后我们的回推墙（结束条件）就是第1项和第0项，这两个知道了，后面就都知道了，这就是归。归就是知道了回推墙依次往回算我们要求的结果。





### 四.数组在函数中的应用：

#### 1.数组作为函数参数：

数组可以作为函数的参数使用，进行数据传送。数组用作函数参数有两种形式，一种是把数组元素(下标变量)作为实参使用；另一种是把数组名作为函数的形参和实参使用。

（1）数组元素作函数实参：

数组元素就是下标变量，它与普通变量并无区别。 因此它作为函数实参使用与普通变量是完全相同的，在发生函数调用时，把作为实参的数组元素的值传送给形参，实现单向的值传送。

例：

设计程序实现判别一个整数数组中各元素的值，若大于 0 则输出该值，若小于等于 0 则输出 0值。

```c
#include <stdio.h>

void nzp(int v)
{
	if(v > 0)
		printf("%d ", v);
	else
		printf("%d ", 0);
}
int main()
{
	int a[5], i;
	printf("input 5 numbers\n");
	for(i = 0; i < 5; i++)
	{
        scanf("%d", &a[i]);//数组元素单个做实参
		nzp(a[i]);
	}
	return 0;
}

//代码输出：
/*
input 5 numbers
2 -1 4 -5 6
2 0 4 0 6
*/
```

本程序中首先定义一个无返回值函数 nzp，并说明其形参 v 为整型变量。在函数体中根据 v 值输出相应的结果。在 main 函数中用一个 for 语句输入数组各元素，每输入一个就以该元素作实参调用一次 nzp 函数，即把 a[i]的值传送给形参 v，供 nzp 函数使用。



（2）数组名作为函数参数：

用数组名作函数参数与用数组元素作实参有几点不同：

1.对数组元素的处理是按普通变量对待的。用数组名作函数参数时，则要求形参和相对应的实参都必须是**类型相同**的数组，都必须有明确的数组说明。
2.在普通变量或下标变量作函数参数时，形参变量和实参变量是由编译系统分配的两个不同的内存单元。在函数调用时发生的值传送是把实参变量的值赋予形参变量。数组名作函数参数时所进行的传送只是**地址的传送**，也就是说把实参数组的**首地址赋予形参数组名**。形参数组名取得该首地址之后，也就等于有了实在的数组。实际上是**形参数组和实参数组为同一数组，共同拥有一段内存空间**。



![在这里插入图片描述](https://img-blog.csdnimg.cn/f042b7c90f764e4c9d228d92ac8cea39.png)

**==实质是地址的传递，将数组的首地址传给形参，形参和实参共用同一存储空间，形参的变化就是实参的变化。(不分配内存，直接使用实参的内存地址)==**

```c
#include <stdio.h> 

float average(float array[5])
{
	 int i;
	 float aver, sum = array[0];
	 for(i = 1; i < 10; i++)
		sum = sum + array[i];
	 aver = sum / 5;
	 return aver;
}
main()
{
	 float score[5], aver;
	 int i;
	 printf("input 5 score:\n");
	 for(i = 0; i < 5; i++)
		scanf("%f", &score[i]);
	 printf("\n");
	 aver = average(score);//使用数组名作为实参
	 printf("average score is %5.2f\n", aver);
}


```



当然，也可以不用写数组得长度，也是同理得

```c
#include <stdio.h>

//如果是b[15] 这里系统是没有分配内存的,b[10]不属于a[10],非要往这里写入内容,很可能会导致崩溃;因为a[10]传过来只有a[0]-a[9] 这十个内存地址,系统并没有分配15个内存地址
//这里可以是b[10],也可以是b[],形参数组的大小可以不指定,即使指定了也可以与实参数组大小不一致,因为C编译器对形参数组的大小不做检查,只是将实参数组的首地址传递给形参数组

void sort(int b[],int n)
{
	for(int i = 0; i < n - 1; i ++)
		for(int j = 0; j < n-i-1; j ++)
		if(b[j] > b[j+1])
		{
            int temp;
            temp = b[j];
            b[j] = b[j+1];
            b[j+1] = temp;
        }
}
int main()
{
	int a[10];//能引用的是a[0]--a[9]
	printf("请输入数组的元素:");
	for(int i = 0;i < 10; i ++)
	{
		scanf("%d",&a[i]);
	}
	printf("排序后的数组顺序是:");
	sort(a,10);//使用数组名作为实参
	for(int i = 0; i < 10;  i++)
	{
		printf("%d ",a[i]);
	}
    return 0;
} 

```



#### 2.用多维数组作为函数实参

可以用多维数组名作为形参和实参，形参数组在定义时，可以指定每一维的大小，也可以忽略第一维大小，但不能省略第二维大小。

实参是这些行这些列，形参就尽量跟实参一样(也是这些行，这些列)，这样实参能引用的下标和形参一样可以引用，就会保证你不出错。

```c
#include<stdio.h>

void changevalue(int b[5][8])//这里5可以省略，但是 8不可以省略 
{
    b[0][2] = 15;//实参里面能引用的下标，形参里面也可以引用 
    
    return;
}
int main()
{
    int a[5][8];//能引用的是a[0]--a[4]
    a[0][2]=3;
    changevalue(a);//使用数组名作为实参
    printf("a[0][2]=%d\n",a[0][2]);
 
 
    return 0;
}
```



### 五.全局变量与局部变量：

**局部变量（Local Variable）**：定义在函数体内部的变量，作用域仅限于函数体内部。离开函数体就会无效。再调用就是出错。

**全局变量（Global Variable）:**定义：所有的函数外部定义的变量，它的作用域是整个程序，也就是所有的源文件，包括.c和.h文件。



#### 1.局部变量：

```c
#include <stdio.h>

int main()
{
	int iTemp = 10;// 这是一个局部变量
	printf("iTemp = %d\n", iTemp);
    return 0;
}
```



#### 2.静态局部变量：

顾名思义，还是一个局部变量，同样是在函数内部定义，只不过是 **静态** 的，也就是存储方式和生存周期不一样。

```c
#include <stdio.h>

int TempAdd()
{
	static int iTemp = 0;// 这是一个静态局部变量
	iTemp++;
	return iTemp;
}

int main()
{
	for(int i = 0; i < 5; i++)
	{
		printf("Temp = %d\n", TempAdd());
	}
    return 0;
}
```



#### 3.全局变量：

全局变量，定义在所有函数之外的变量，对整个工程文件有效。当前文件可直接使用，如果不是当前文件，则需在调用文件开头加上关键字 **extern** 。

```c
#include <stdio.h>

int iTemp = 0;// 这是一个全局变量

int main()
{
	for(int i = 0; i < 5; i++)
	{
		printf("iTemp = %d\n", iTemp++);
	}
    return 0;
}
```

为了更明显的体现出来是全局变量，我再写一个子函数调用，代码如下：

```c
#include <stdio.h>

int iTemp = 0;// 这是一个全局变量

void TempAdd()
{
	iTemp++;
}

int main()
{
	for(int i = 0; i < 5; i++)
	{
		printf("iTemp = %d\n", iTemp++);
	}

	printf("----------This is a Dividing line!---------\n");

	TempAdd();

	printf("iTemp = %d\n", iTemp);
}
```



#### 4.静态全局变量：

静态全局变量，也是一个全局变量，不过加上 **静态** 之后，就限定了此变量的作用范围。

```c
// main.c
#include <stdio.h>
#include "static.h"

int main()
{
	iTemp = 10;
	printf("iTemp = %d\n", iTemp);
    return 0;
}
```



```C
// static.c
#include <stdio.h>
#include "static.h"

void TempTest()
{
	for(int i = 0; i < 5; i++)
	{
		iTemp++;
		printf("iTemp = %d\n", iTemp);
	}
}
```



```c
// static.h
#ifndef _STATIC_H_
#define _STATIC_H_

static int iTemp = 0;		// 这是一个静态全局变量

#endif
```



### 六.变量的储存方式于变量期：

单片机时在做了解



### 七.内部函数与外部函数：



单片机时在做了解



### 八.作业：

1.理解汉诺塔，仿造写一遍，并理解思路；

2.设计程序实现写一个函数，功能为输入3X3的矩阵，并将3X3的矩阵行列转换；

3.写两个函数，分别求最大两个整数的最大公约数和最小公倍数。

4.设计一个程序实现输入字符串，并排序输出。（练习fgets函数与自定义函数）提示：冒泡排序

 一.什么是[汉诺塔](https://so.csdn.net/so/search?q=汉诺塔&spm=1001.2101.3001.7020)？

**汉诺塔起源于印度的一个古老传说，传说是什么不重要。重要的是它是怎么实现的以及代码是怎么写的。**
是这样的，总共有 3 个柱子，第一个柱子上有 n 个圆盘，圆盘的大小由下到上依此递减。
**以 n = 3为例**  

![在这里插入图片描述](https://img-blog.csdnimg.cn/d2c6aee2cd5c4609ae98ab544d2e2e98.png)

**大概就是这么个东西。**
我们要做的就是利用第二个柱子把第一个柱子上的所有圆盘放到第三个柱子上，在移的过程中大盘子不能放在小盘子上。

如何实现：

假如有 n 个盘子

我们要将这 n 个盘子移到第三个柱子上，先需将前 n-1 个盘子移到第二个柱子上，再将第 n 个盘子移到第三个柱子上，最后将这 n 个盘子移到第三个柱子上。

要将这 n-1 个盘子移到第三个柱子上，先需将前 n-2 个盘子移到第二个柱子上，再将第 n-1 个盘子移到第三个柱子上，最后将这 n-1 个盘子移到第三个柱子上。

以此类推。。。

所以，不难看出，汉诺塔的问题本身是一个大问题，我们可以将这个大问题转化为许多个相似的小问题来进行解决。若要将一个大问题转化为许多个相似的小问题来解决，我们就需要用到递归了。



第一题：

```c
#include <stdio.h>

void hanoi(int n, char pos1, char pos2, char pos3)
{
	if (n == 1)
	{
		printf("%c->%c ", pos1, pos3);
	}
	else
	{
		hanoi(n - 1, pos1, pos3, pos2);
		printf("%c->%c ", pos1, pos3);
		hanoi(n - 1, pos2, pos1, pos3);
	}
}

int main()
{
	int n = 0;
	scanf("%d", &n);
	hanoi(n,'A','B','C');
	return 0;
}
```



第二题：

```c
#include<stdio.h>

void sort(int x[][100])
{
	for(int i = 0; i < 3; i++)
	{
        for(int j = 0; j < 3; j++)
		{
            scanf("%d", &x[i][j]);
        }
    }
    putchar('\n');
    for(int i = 0; i < 3; i++)
	{
        for(int j = 0; j <3 ; j++)
		{
            printf("%d ", x[j][i]);
        }
        putchar('\n');
    }
}
int main()
{
    int num[100][100];
    sort(num);
    return 0;
}
```



第三题：

```c
#include<stdio.h>

int main()
{
	int gys(int x, int y);
	int gbs(int x, int y);
	int m, n, max, min, t;
	printf("请输入两个正整数m和n:\n");
	scanf("%d %d", &m, &n);
	if(m < n)
	{
		t = m;
        m = n;
        n = t;
	}
	max = gys(m, n);
	min = gbs(m, n);
	printf("gys=%d\ngbs=%d\n",max, min);
	return 0;
}
int gys(int x, int y)
{
	int z;
	z = x % y;
	while(z != 0)
	{
		x = y;
		y = z;
		z=x % y;
	}
	return (y);
}
int gbs(int x, int y)
{
	return ((x*y)/gys(x,y));
}
```



第四题：

```c
#include <stdio.h>
#include <string.h>

#define MAXSIZE 100

void sort(char a[], int x)
{
	int temp;
	for(int i = 0; i < x - 1; i++)
	{
		for(int j = 0; j < x - 1 - i; j++)
		{
			if(a[j] > a[j+1])
			{
				temp = a[j];
				a[j] = a[j+1];
				a[j+1] = temp;
			}
		}
	}	
} 

int main()
{
	int x;
	char words[MAXSIZE], temp;
	fgets(words, 100, stdin);
	x = strlen(words);
	sort(words, x);
	for(int i = 1; i < x; i++)//首位为'\n'，所以下标从1开始 
	{
		printf("%c", words[i]);
	}
	//printf("\n%d", x);
}

/*
int main()
{
	int x;
	char words[MAXSIZE], temp;
	scanf("%[^\n]", words); //scanf的正则表达式,作用相当于gets,%[^\n]的作用是过滤字符串'\n'
	x = strlen(words);
	sort(words, x);
	for(int i = 0; i < x; i++)
	{
		printf("%c", words[i]);
	}
	//printf("\n%d", x);
}
*/
/*
int main()
{
	int x;
	char words[MAXSIZE], temp;
	fgets(words, 100, stdin);
	x = strlen(words) - 1;
	sort(words, x);
	for(int i = 0; i < x; i++)
	{
		printf("%c", words[i]);
	}
	//printf("\n%d", x);
}
*/
```

