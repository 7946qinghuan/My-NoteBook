---
title: C语言第八次培训
date: 2026-08-01
tags: [Dev, C]
aliases: []
---

# ==C语言第八次培训==

## ------不同的结构类型



[TOC]



### 一.什么是结构体？

在学习结构体之前我们需要了解什么是结构体？我们为什么要引用结构体这个概念呢？

拿一个学生来说，一个学生的身上有很多的属性例如：年龄，姓名，身高，性别，分数等等，而学生又不止一个，有很多的学生，这时候我们想要记录每个学生的数据，首先我们想到的是我们可以定义多个变量来表示不同的学生，但是这样的话我们不仅要定义学生变量还要定义多个学生的属性，这样的话如果有1000个学生的话光是定义变量就会消耗不少时间。

这时我们想到我们可以定义一个学生的模板，然后把学生的各个属性全部放在这个模板中这样的话每定义一个学生只需要引用一下这个模板就行了，大大的减少了我们的工作量。



简单来说：结构体是一个模板，是一些值的集合，这些值称为成员变量。而结构体的每个成员又可以是不同类型的变量。



#### 1.结构体的声明：

格式：

```c
struct 结构体名
{
    成员表列;
}变量名表列;
```



结构体名也就是我们结构体的名字，可以随便命名；成员表列也就是我们不同类型的变量；变量名表列也就是我们定义的变量类型的变量名。当然，变量名表列可以没有，可以后面再定义。

定义如果没有懂，我们通过一个例子来加强理解。

例：

```c
struct Student
{
    int age;
    char name[30];
    char sex;
    int class;
}student_1,student_2;
```

我们定义了一个Student类型的结构体，它的成员有int类型的变量age，char类型的数组name，char类型的变量sex，int类型的变量class，然后我们定义了两个Student类型的变量student_1和student_2。

我们再看另外一种情况：

```c
struct Student
{
    int age;
    char name[30];
    char sex;
    int class;
};

struct Student student_1,student_2;
```



#### 2.typedef结构体：

typedef的作用是起别名：

例如：

```c
typedef  int   NEW_INT;
```

我们重新给int类型起了一个名字叫NEW_INT；

所以定义整形变量时，我们便可以：

```c
int a;

NEW_INT b;
```

好了，我们知道typedef的作用后我们就可以简化结构体的定义：

```c
typedef struct NUM
{
     int a;
     int b;
}DATA,*PTRDATA;
```

此时DATA等同于struct NUM，*PTRDATA等同于struct NUM *。

```c
DATA data;       //定义结构体变量
 
PTRDATA pdata;   //定义结构体指针
```



#### 3.结构体的定义和使用：

在使用我们定义的结构体后，访问它的成员要用成员访问符号**"==.=="**

```c
结构体变量名.成员名
```

我们直接上代码：

```c
#include <stdio.h>

struct Student//声明一个Student类型的结构体
{
    int age;
    char name[30];
    char sex;
    int class_;
}student_1 = {19, "张三", 'm', 1};//定义并初始化一个Student类型的结构体变量student_1

int main()
{
    struct Student student_2 = {19, "李四", 'm', 2};//定义并初始化一个Student类型的结构体变量student_2
	printf("年龄：%d\n姓名：%s\n性别：%c\n班级：%d\n", student_1.age, student_1.name, student_1.sex, student_1.class_);
    putchar('\n');
	printf("年龄：%d\n姓名：%s\n性别：%c\n班级：%d\n", student_2.age, student_2.name, student_2.sex, student_2.class_);
	return 0;
}
/*代码输出：
年龄：19
姓名：张三
性别：m
班级：1

年龄：19
姓名：李四
性别：m
班级：2

*/
```



```c
#include <stdio.h>

typedef struct Student//声明一个Student类型的结构体
{
    int age;
    char name[30];
    char sex;
    int class_;
}Stu;//取别名Stu 

int main()
{
    Stu student_1 = {19, "张三", 'm', 1};//定义并初始化一个Student类型的结构体变量student_1
    Stu student_2 = {19, "李四", 'm', 2};//定义并初始化一个Student类型的结构体变量student_2
	printf("年龄：%d\n姓名：%s\n性别：%c\n班级：%d\n", student_1.age, student_1.name, student_1.sex, student_1.class_);
    putchar('\n');
	printf("年龄：%d\n姓名：%s\n性别：%c\n班级：%d\n", student_2.age, student_2.name, student_2.sex, student_2.class_);
	return 0;
}
```



实例练习：

输入两个同学的信息，比较成绩，输出成绩高者，否则两个都输出。

```c
#include<stdio.h>

int main() 
{
	struct Student 
	{
		int num;
		char name[20];
		float score;
	} student1, student2;
	scanf("%d%s%f", &student1.num, &student1.name, &student1.score);
	scanf("%d%s%f", &student2.num, &student2.name, &student2.score);
	printf("The higer score is:\n");
	if(student1.score > student2.score)
		printf("%d  %s  %6.2f\n",student1.num,student1.name,student1.score);
	else if(student1.score<student2.score)
		printf("%d  %s  %6.2f\n",student2.num,student2.name,student2.score);
	else 
	{
		printf("%d  %s  %6.2f\n",student1.num,student1.name,student1.score);
		printf("%d  %s  %6.2f\n",student2.num,student2.name,student2.score);
	}
	return 0;
}
```





#### 4.结构体的嵌套：

如同我们的循环和函数一样，结构体也可以嵌套，那么如何嵌套呢？

我们直接上代码：

```c
#include <stdio.h>

struct Data//声明一个Data类型的结构体 
{
	int year;
	int month;
	int data;
};

struct Student//声明一个Student类型的结构体 
{
    char name[20];
    Data birthday;
    int age;
}student_1 = {"张三", {2000, 5, 1}, 23};//定义并初始化一个Student类型的变量student_1 

int main()
{
	printf("姓名：%s\n出生日期：%d.%d.%d\n年龄：%d\n", student_1.name, student_1.birthday.year, student_1.birthday.month, student_1.birthday.data, student_1.age);
	return 0;
}
```



应该很好理解，结构体的嵌套简单来说就是先访问其父亲在访问其子女，父亲就是名字，子女就是成员。



#### 5.结构体变量和指针

结构体指针的使用就如同普通变量和指针一样，就是需要注意一些点：

两种访问方式：

**（1）（\*p）. 成员名（.的优先级高于\*，（\*p）两边括号不能少）**

**（2）  p->成员名（->指向符）**



```c
#include<stdio.h>

typedef struct Inventory//商品 
{
	char description[20];//货物名 
	int quantity_1;//库存数据 
	int quantity_2;//库存数据 
}inv, *inv_p;

int main()
{
	inv sta = { "iphone",6, 10};//定义并初始化一个结构体变量sta 
	inv_p stp = &sta;//定义一个结构体指针stp，并引用指针 
	(*stp).quantity_1 = 13;
	stp->quantity_2 = 14;
	printf("%s %d\n", stp->description, stp->quantity_1);
	printf("%s %d\n", (*stp).description, (*stp).quantity_2);
	return 0;
}
```



#### 6.结构体和函数：

把结构体看作是函数的参数就如同普通变量一样，不过我们需要注意的是，我们==可以把结构体看作是指针==，具体为什么，需要自己去查找资料，这里不具体说明。所以就有两种方式把结构体作为参数传给函数

例：

```c
#include<stdio.h>
#include<string.h>

typedef struct School 
{
	char s_name[100];//学校 
	int s_age;
}sch;

void Print_a(struct School sx) 
{
	printf("%s %d\n", sx.s_name, sx.s_age);
}

void Print_c(struct School* sp) 
{
	printf("%s %d\n", sp->s_name, sp->s_age);
}

int main() 
{
	sch sx = { "电子科技大学成都学院",21};
	Print_a(sx);
	Print_c(&sx);
	return 0;

}
```



#### 7.结构体与数组：

**结构体数组，是指数组中的每一个元素都是一个结构体类型。**在实际应用中，C语言结构体数组常被用来表示有相同的数据结构的群体，比如一个班的学生，一个公司的员工等；

例：

```c
#include<stdio.h>

typedef struct Student
{
	char s_name[20];//姓名 
	int age;//年龄 
	float score;//成绩 
}Stu;
int main()
{
	Stu cla[] =
	{
		{"liuwen",18,149.9},
		{"qnge",18,145},
		{"anan",19,188},
	};
	return 0;
}
```



### 二.结构体的深入：

#### 1.结构体的大小：

首先，我们需要知道计算结构体大小必须要掌握**==结构体的对齐规则==**：

```c
1. 第一个成员在与结构体变量偏移量为0的地址处。

2. 其他成员变量要对齐到某个数字（对齐数）的整数倍的地址处。 对齐数 = （编译器默认的一个对齐数 与 该成员大小）的较小值。

	VS和Dev-C++默认的值为8

3. 结构体总大小为最大对齐数（每个成员变量都有一个对齐数）的整数倍。

4. 如果嵌套了结构体的情况，嵌套的结构体对齐到自己的最大对齐数的整数倍处，结构体的整 体大小就是所有最大对齐数（含嵌套结构体的对齐数）的整数倍。
```



我们来看代码理解:

```c
#include <stdio.h> 

struct S1 
{
	char c1; //1个字节 
	int i;   //4个字节 
	char c2; // 1个字节 
};

struct S2 
{
	char c1; //1个字节 
	char c2; //1个字节 
	int  i;	 //4个字节 
};

int main() 
{
	struct S1 s1;
	struct S2 s2;
	printf("%d\n", sizeof(s1));
	printf("%d\n", sizeof(s2));
	return 0;
}
```



正常情况下，你会认为两个结构体s1和s2的大小为6，但是真的如此吗？

```c
/*代码输出：
12
8

*/
```

当你观看到答案时，你会非常惊讶！为什么差距如此之大？这时候我们就需要回到我们的结构体的对齐规则；



先理解第一条：

```c
#include <stdio.h> 

struct S1 
{
	char c1; //1个字节 
	int i;   //4个字节 
	char c2; // 1个字节 
};

struct S2 
{
	char c1; //1个字节 
	char c2; //1个字节 
	int  i;	 //4个字节 
};

int main() 
{
	struct S1 s1;
	struct S2 s2;
	printf("%p\n", &s1);	//struct S1*
	printf("%p\n", &s1.c1); //char*
	putchar('\n');
    printf("%p\n", &s2);	//struct S2*
	printf("%p\n", &s2.c1); //char*
	return 0;
}

/*代码输出：
000000000062FE10
000000000062FE10

000000000062FE00
000000000062FE00

*/
```



不难理解，第一条的意思就是说结构体的地址和我们结构体首成员的地址相同。



后面几条我们一起看：



![结构体大小](https://s2.loli.net/2023/10/04/WpIQrdv98KflDVb.png)



我们先看s1，因为最大对其数为int类型，对齐数为4，占四个字节，所以每次都是分配四个空间；

看结构体s1的内存分配，先分配四个空间，第一个是char类型的c1（对齐数为1，每个地址都可以用），占一个字节，浪费三个空间，再分配四个空间，为什么呢？

因为int类型i的对齐数为4，对齐的地址要是4的整数倍；所以第二个int类型的i需要重新分配四个空间来占用；

同理第三个是char类型的c2（对齐数为1，每个地址都可以用），占一个字节，浪费三个空间；

所以最后结构体s1的大小为12；



我们再看s2，因为最大对其数为int类型，对齐数为4，占四个字节，所以每次都是分配四个空间；

看结构体s2的内存分配，先分配四个空间，第一个是char类型的c1（对齐数为1，每个地址都可以用），占一个字节，第二个是char类型的c2（对齐数为1，每个地址都可以用），占一个字节，所以浪费两个空间，再分配四个空间，第三个为int类型i（对齐数为4，对齐的地址要是4的整数倍），占四个字节；

所以最后结构体s2的大小为8；



我们用**offsetof**函数来验证一下：

**offsetof**函数就是计算结构体成员相对于起始位置的偏移量。

```c
#include <stdio.h> 
#include<stddef.h>

typedef struct S1 
{
	char c1; //1个字节 
	int i;   //4个字节 
	char c2; // 1个字节 
}S1;

typedef struct S2 
{
	char c1; //1个字节 
	char c2; //1个字节 
	int  i;	 //4个字节 
}S2;

int main() 
{
	S1 s1;
	S2 s2;
	printf("%d\n", offsetof(S1,c1));
	printf("%d\n", offsetof(S1,i));
	printf("%d\n", offsetof(S1,c2));
	putchar('\n');
    printf("%d\n", offsetof(S2,c1));
	printf("%d\n", offsetof(S2,i));
	printf("%d\n", offsetof(S2,c2));
	return 0;
}

/*代码输出：
0
4
8

0
1
4

*/
```



需要特别注意的是：

1.int和float都占四个字节，double占八个字节；

2.没有更大的类型下，指针在Dev占八个字节；

3.数组可以看作是多个变量；



实例练习：

```c
#include <stdio.h> 

typedef struct S1 
{
	char c1;
	int i1;
	char c2;
	float f1;
}S1;

typedef struct S2 
{
	char c1;
	char c2;
	int i1;
	double d1;
}S2;

typedef struct S3 
{
	float f1;
	int i1;
	double d1;
}S3;

typedef struct S4
{
	char c1;
	char c2;
	float f1;
	int i1;
	int *p;
	
}S4;

typedef struct S5
{
	char c1[10];
	float f1;
	int i1;
	
}S5;

typedef struct S6
{
	int i1[10];
	float f1;
	int i2;
	int *p;
}S6;

int main() 
{
	S1 s1;
	S2 s2;
	S3 s3;
	S4 s4;
	S5 s5;
	S6 s6;
	printf("%d\n", sizeof(s1));
	printf("%d\n", sizeof(s2));
	printf("%d\n", sizeof(s3));
	printf("%d\n", sizeof(s4));
	printf("%d\n", sizeof(s5));
	printf("%d\n", sizeof(s6));
	return 0;
}
```



答案：

```c
/*代码输出：
16
16
16
24
20
56

*/
```



#### 2.修改默认对齐数：

```c
#pragma 这个预处理指令，可以改变我们的默认对齐数。
```



```c
#include <stdio.h> 
#include<stddef.h>

#pragma pack(1)

typedef struct S1 
{
	char c1; //1个字节 
	int i;   //4个字节 
	char c2; // 1个字节 
}S1;

typedef struct S2 
{
	char c1; //1个字节 
	char c2; //1个字节 
	int  i;	 //4个字节 
}S2;

int main() 
{
	S1 s1;
	S2 s2;
	printf("sizeof(s1) = %d\n", sizeof(s1));
	printf("c1的偏移量 = %d\n", offsetof(S1,c1));
	printf("i的偏移量 = %d\n", offsetof(S1,i));
	printf("c2的偏移量 = %d\n", offsetof(S1,c2));
	putchar('\n');
	printf("sizeof(s2) = %d\n", sizeof(s2));
    printf("c1的偏移量 = %d\n", offsetof(S2,c1));
	printf("c2的偏移量 = %d\n", offsetof(S2,c2));
	printf("i的偏移量 = %d\n", offsetof(S2,i));
	return 0;
}
```



大家可以猜一猜现在结构体s1和s2的大小为多少？

答案都是6！



### 三.枚举类型：

`enum` 是一种[数据类型](https://so.csdn.net/so/search?q=数据类型&spm=1001.2101.3001.7020)，可以在程序中创建新的类型，并为其赋予一组明确的值。



#### 1.定义enum：

`enum` 类型的定义使用[关键字](https://so.csdn.net/so/search?q=关键字&spm=1001.2101.3001.7020) `enum`，然后是一个可选的[类型名称](https://so.csdn.net/so/search?q=类型名称&spm=1001.2101.3001.7020)，以及一对花括号内的标识符列表。例如：

```c
enum day//枚举类型的声明,enum是枚举的关键字 
{
	MONDAY, 
	TUESDAY, 
	WEDNESDAY, 
	THURSDAY,
	FRIDAY,
	SATURDAY,
	SUNDAY
};
```

此代码定义了一个新的 `enum` 类型 `day`，以及一组可能的值：`MONDAY`、`TUESDAY`、`WEDNESDAY`等。



#### 2. enum的默认值

每个 `enum` 类型的成员都有一个整数值，如果不指定，则从 0 开始递增。例如，在上述 `day` 类型的例子中，`MONDAY` 的值为 0，`TUESDAY` 的值为 1，以此类推。



#### 3.使用enum：

```c
#include <stdio.h>

enum day
{
	MONDAY, 
	TUESDAY, 
	WEDNESDAY, 
	THURSDAY,
	FRIDAY,
	SATURDAY,
	SUNDAY
};

int main() 
{
    enum day today;
    today = TUESDAY;
    printf("%d\n", today);//1
    return 0;
}
```



当然，我们也可以自己指定枚举类型里的变量的值



```c
#include <stdio.h>

enum day
{
	MONDAY = 1, 
	TUESDAY, 
	WEDNESDAY, 
	THURSDAY,
	FRIDAY,
	SATURDAY,
	SUNDAY
};

int main() 
{
    enum day today;
    today = TUESDAY;
    printf("%d\n", today);//2
    return 0;
}
```



此时第一个的值为1，后面的所有默认在上一个的基础上加一

所以当我们给第n个赋值后，后面的所有默认在上一个的基础上加一



实例练习：



```c
#include <stdio.h>

enum day
{
	MONDAY = 11, 
	TUESDAY = 12, 
	WEDNESDAY, 
	THURSDAY,
	FRIDAY,
	SATURDAY,
	SUNDAY
};

int main() 
{
    enum day today;
    today = WEDNESDAY;
    printf("%d\n", today);//2
    return 0;
}
```



#### 4.typedef枚举类型：

和结构体的使用一样：

```c
#include <stdio.h>

typedef enum day
{
	MONDAY = 1, 
	TUESDAY = 2, 
	WEDNESDAY, 
	THURSDAY,
	FRIDAY,
	SATURDAY,
	SUNDAY
}week;

int main() 
{
    week today;
    today = WEDNESDAY;
    printf("%d\n", today);//3
    return 0;
}
```



#### 5.enum作为函数参数:

```c
#include <stdio.h>

typedef enum people
{
	man,
	woman
}peo;

void display(peo x)
{
	switch(x)
	{
		case man : printf("boy\n");break;
		case woman : printf("girl\n");break;
        default : printf("error\n");break;
	}
}

int main() 
{
    peo student_1,student_2;
	student_1 = man;
	student_2 = woman;
	printf("%d %d\n", student_1, student_2);
	display(student_1);
	display(student_2);
    return 0;
}
```



**enum的优点：**

1. **增加代码的可读性和可维护性**

2. **和#define定义的标识符比较枚举有类型检查，更加严谨。**

3. **防止了命名污染（封装）**

4. **便于调试**

5. **使用方便，一次可以定义多个常量**



### 四.联合类型：

**联合是一种特殊的自定义类型**，该种类型定义的变量也包含一系列的成员，特征是这些成员共用同一块空间，所以联合体也被称为共用体。



#### 1.定义union:

```c
#include<stdio.h>

union Un//联合类型的声明,union是联合体关键字 
{
	char c;//1字节 
	int i;//4字节 
};

int main()
{

	union Un u = {0};
	printf("%d\n", sizeof(u));
	printf("%p\n", &u);
	printf("%p\n", &(u.c));//u.c表示联合体的成员c，该引用方法类似结构体 
	printf("%p\n", &(u.i));
}

/*代码输出：
4
000000000062FE10
000000000062FE10
000000000062FE10

*/
```



由sizeof（u）我们知道这个联合体总计占4个字节，而联合体成员i是int类型的，它占了4个字节，另外一个c是char类型占了1个字节，两个一起占了4个字节。说明c和i必然有一处是共用一块空间的，再者有u本身和它的两个成员是一个地址000000000062FE10，说明首地址是重合的，简易示图如下：



![共用体](https://s2.loli.net/2023/10/04/n84NGjpsoB6idDE.png)



由于共用空间这种特点就导致了，你改变c，i也会随之改变。这里和结构体是完全不一样的，结构体成员相互独立，但联合体不一样，改一个，其他的也会改变。**所以这里，在同一时间，你只能使用一个联合体成员**，你使用c就不要用i，因为你c改变的时候，一定会影响到你i的使用，程序非常容易出问题。



#### 2.typedef联合体：

和结构体的使用一样：

```c
typedef union Un1
{
	char c[5];//1个char类型占1字节，5个占5字节 
	int i;//4字节 
}Un1;

typedef union Un2
{
	short c[7];//1个short类型占2字节，7个占14字节 
	int i;//4字节 
}Un2;
```



#### 3.联合体的大小:

在计算联合体大小之前我们必须知道两个知识点：
**1.联合的大小至少是最大成员的大小**
**2.当最大成员大小不是最大对齐数的整数倍的时候，就要对齐到最大对齐数的整数倍。**



```c
#include<stdio.h>

typedef union Un1
{
	char c[5];//1个char类型占1字节，5个占5字节 
	int i;//4字节 
}Un1;

typedef union Un2
{
	short c[7];//1个short类型占2字节，7个占14字节 
	int i;//4字节 
}Un2;

int main()
{
	printf("%d\n", sizeof(Un1));//打印8 
	printf("%d\n", sizeof(Un2));//打印16 
}
```



Un1解释:
char创建一个大小为5的数组和放5个char类型的是一样道理，对齐数仍然是1
int类型的i自身大小4字节，默认对齐数8，对齐数是4。i和c两个最大的对齐数是4，而最大成员大小是数组c（5个字节），5不是4的倍数，我们需要对齐到最大对齐数的整数倍，也就是8（从5到8会浪费3个字节空间）
Un2解释：
short创建的c数组，我们同上可知其c对齐数是2，i对齐数是4，最大对齐数为4。最大成员大小也就是c数组大小为14,14并不是最大对齐数4的整数倍，14往上对齐到16,16是4的整数倍。



实例练习：

```c
#include<stdio.h>

typedef union Un1
{
	float f;//4字节 
	int i;//4字节 
}Un1;

typedef union Un2
{
	short s[2];//1个char类型占1字节，2个占2字节 
	int i;//4字节 
}Un2;

typedef union Un3
{
	char c;
	short s;
}Un3;

typedef union Un4
{
	char c[7];
	short s;
}Un4;

int main()
{
	printf("%d\n", sizeof(Un1));
	printf("%d\n", sizeof(Un2));
	printf("%d\n", sizeof(Un3));
	printf("%d\n", sizeof(Un4));
	return 0;
}
```



```c
/*代码输出：
4
4
2
8

*/
```



#### 4.联合体的使用：

共用体在一般的编程中应用较少，在单片机中应用较多。对于 PC 机，经常使用到的一个实例是： 现有一张关于学生信息和教师信息的表格。学生信息包括姓名、编号、性别、职业、分数，教师的信息包括姓名、编号、性别、职业、教学科目。请看下面的表格：



| Name        | Num  | Sex  | Profession | Score / Course |
| ----------- | ---- | ---- | ---------- | -------------- |
| HanXiaoXiao | 501  | f    | s          | 89.5           |
| YanWeiMin   | 1011 | m    | t          | math           |
| LiuZhenTao  | 109  | f    | t          | English        |
| ZhaoFeiYan  | 982  | m    | s          | 95.0           |

f 和 m 分别表示女性和男性，s 表示学生，t 表示教师。可以看出，学生和教师所包含的数据是不同的。现在要求把这些信息放在同一个表格中，并设计程序输入人员信息然后输出。

如果把每个人的信息都看作一个结构体变量的话，那么教师和学生的前 4 个成员变量是一样的，第 5 个成员变量可能是 score 或者 course。当第 4 个成员变量的值是 s 的时候，第 5 个成员变量就是 score；当第 4 个成员变量的值是 t 的时候，第 5 个成员变量就是 course。

经过上面的分析，我们可以设计一个包含共用体的结构体，请看下面的代码：



```c
#include <stdio.h>
#include <stdlib.h>
 
#define TOTAL 4 //人员总数
 
struct
{
    char name[20];
    int num;
    char sex;
    char profession;
    union
    {
        float score;
        char course[20];
    } sc;
} bodys[TOTAL];
 
int main()
{
    int i;
    //输入人员信息
    for(i=0; i<TOTAL; i++)
    {
        printf("Input info: ");
        scanf("%s %d %c %c", bodys[i].name, &(bodys[i].num), &(bodys[i].sex)   &(bodys[i].profession));            
 
    if(bodys[i].profession == 's')
    { //如果是学生
        scanf("%f", &bodys[i].sc.score);
    }
    else
    { //如果是老师
        scanf("%s", bodys[i].sc.course);
    }
    fflush(stdin);
}
 
//输出人员信息
printf("\nName\t\tNum\tSex\tProfession\tScore / Course\n");
for(i=0; i<TOTAL; i++)
{
    if(bodys[i].profession == 's')
    { 
        //如果是学生
        printf("%s\t%d\t%c\t%c\t\t%f\n", bodys[i].name, bodys[i].num, bodys[i].sex,                 bodys[i].profession, bodys[i].sc.score);
 
    }
    else
    { 
        //如果是老师
    printf("%s\t%d\t%c\t%c\t\t%s\n", bodys[i].name, bodys[i].num, bodys[i].sex,     bodys[i].profession, bodys[i].sc.course);
 
    }
}
 
return 0;
 
}
```



```c
Input info: HanXiaoXiao 501 f s 89.5
Input info: YanWeiMin 1011 m t math
Input info: LiuZhenTao 109 f t English
Input info: ZhaoFeiYan 982 m s 95.0
 
Name            Num     Sex     Profession      Score / Course
HanXiaoXiao     501     f       s               89.500000
YanWeiMin       1011    m       t               math
LiuZhenTao      109     f       t               English
ZhaoFeiYan      982     m       s               95.000000
 
```



[关于fflush](https://blog.csdn.net/qq_34793133/article/details/85713413?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169640504616800225539289%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=169640504616800225539289&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-85713413-null-null.142^v94^insert_down1&utm_term=fflush%28stdin%29%3B&spm=1018.2226.3001.4187)



### 五.作业：

1.输入3个学生信息，按成绩排序输出；

提示：

```c
typedef struct student
{
    char name[30];
    int age;
    int class;
    float souce;
}stu;
```



2.使用枚举类型，输出数字1-10的平方；

提示：

```c
enum num
{
	one = 1;
    two,
    three,
    four,
    five,
    six,
    seven,
    eight,
    nine,
    ten
};
```











第一题：

```c
#include <stdio.h>

typedef struct student
{
    char name[30];
    int age;
    int class_;
    float souce;
} stu;

void SortAndPrint(stu *x)
{
    int i, j;
    stu temp;
    for (i = 0; i < 2; i++)
    {
        for (j = 0; j < 2 - i; j++)
        {
            if (x[j].souce > x[j + 1].souce)
            {
                temp = x[j];
                x[j] = x[j + 1];
                x[j + 1] = temp;
            }
        }
    }
    for (i = 0; i < 3; i++)
    {
        printf("姓名：%s\t年龄：%d\t班级：%d\t成绩：%f\n", x[i].name, x[i].age, x[i].class_, x[i].souce);
    }
}

int main()
{
    stu s[3];
    int i;
    for (i = 0; i < 3; i++)
    {
        printf("第%d位同学的姓名：", i + 1);
        scanf("%s", s[i].name);
        printf("第%d位同学的年龄：", i + 1);
        scanf("%d", &s[i].age);
        printf("第%d位同学的班级：", i + 1);
        scanf("%d", &s[i].class_);
        printf("第%d位同学的成绩：", i + 1);
        scanf("%f", &s[i].souce);
        putchar('\n');
    }

    SortAndPrint(s);

    return 0;
}
```



第二题：

```c
#include <stdio.h>

typedef enum num 
{
    one = 1,
    two,
    three,
    four,
    five,
    six,
    seven,
    eight,
    nine,
    ten
} num;

int square(int x) 
{
	return x*x;
}

int main() 
{
	int i;
	int a[10] = {one, two, three, four, five, six, seven, eight, nine, ten};
	for(i=0; i<10; i++) 
	{
		a[i] = square(a[i]);
		printf("%-5d", a[i]);
	}
	return 0;
}
```

