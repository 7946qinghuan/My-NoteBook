---
title: C语言第五次培训
date: 2026-08-01
tags: [Dev, C]
aliases: []
---

# ==C语言第五次培训==

## ------数组



[TOC]

### 一.什么是数组？

设想有这样一个题，用户让你输出1-10，10个数，你会怎么做？

在没学循环语句前，我们可能会这样做：

```c
#include <stdio.h>

int main()
{
    printf("1 2 3 4 5 6 7 8 9 10");
    return 0;
}
```



在学了循环语句后，我们可能会这样做：

```c
#include <stdio.h>

int main()
{
    int i;
    for(i = 0; i < 11; i++)
    {
        printf("%-2d", i);
    }
    return 0;
}
```



但是如果用户要求升级，要求我们保存这10个数，并且在1-10之间用户想输出哪个数就输出哪个数，我们该如何解决问题呢？

此时我们便需要使用数组：

```c
#include <stdio.h>

int main()
{
    int i, n, a[10];
    scanf("%d", &n);//输入用户需要的数
    for(i = 0; i < 10; i++)//保存1-10，10个数
    {
    	a[i] = i + 1;
    }
    printf("%d\n", a[n-1]);//输出用户需要的数
    return 0;
}
```



从运行结果上看，数组的确能够帮助我们存储我们需要的数据。



### 二.定义使用一维数组：

#### 1.定义创建一个一维数组：

```c
#include <stdio.h>

#define X 10

int main()
{
    int a[X], b[5], c[100];
    return 0;
}
```

a,b,c为数组的名字，X,5,100为数组的大小，用中括号圈起来，注意必须是常量，我们可以用define来定义，用来代表数组的大小。



#### 2.初始化一个数组：

```c
#include <stdio.h>

#define X 10

int main()
{
	int i;
    int a[X] = {1, 2};//只初始化前两个，后面为0
    int b[5] = {1, 2, 3, 4, 5};//全部初始化
    int c[6] = {0};//默认初始化
    //如果int c[6];这样，则不能正常输出，也就是说我们没有操作这个数组，就不能去输出它
    for(i = 0; i < X; i++)
    {
    	printf("%-2d", a[i]);
	}
	putchar('\n');
	for(i = 0; i < 5; i++)
    {
    	printf("%-2d", b[i]);
	}
	putchar('\n');
	for(i = 0; i < 6; i++)
    {
    	printf("%-2d", c[i]);
	}
    return 0;
}
//注意：
//1.printf函数不能一次输出整个数组，只能用访问数组的每个元素，然后逐个输出；
//2.数组的下标是从0开始到n-1结束

//代码输出：
/*
1 2 0 0 0 0 0 0 0 0
1 2 3 4 5
0 0 0 0 0 0
*/
```



#### 3.使用一维数组：

回到我们上面那个题

```c
#include <stdio.h>

int main()
{
    int i, n, a[10];
    scanf("%d", &n);//输入用户需要的数
    for(i = 0; i < 10; i++)//保存1-10，10个数
    {
    	a[i] = i + 1;//数组的下标从零开始
    }
    printf("%d\n", a[n-1]);//输出用户需要的数
    return 0;
}
```



### 三.定义使用二维数组：

#### 1.定义创建一个二维数组：

```c
#include <stdio.h>

#define X 10

int main()
{
    int a[X][X], b[5][5], c[100][100];
    return 0;
}
```





#### 2.初始化一个二维数组：

```c
#include <stdio.h>

#define X 5

int main()
{
	int i, j;
    int a[X][X] = {{1, 1, 1, 1 , 1}, {1, 1, 1, 1}, {1, 1, 1}, {1, 1}, {1}};//每行都初始化，用'{}'区分，','间隔开 
    int b[5][5] = {1, 2, 3, 4, 5};//只初始化第一行 
    int c[6][6] = {0};//默认初始化
    //如果int c[6];这样，则不能正常输出，也就是说我们没有操作这个数组，就不能去输出它
    for(i = 0; i < X; i++)
    {
    	for(j = 0; j < X; j++)
    		printf("%-3d", a[i][j]);
    	putchar('\n');
	}
	putchar('\n');
	for(i = 0; i < 5; i++)
    {
    	for(j = 0; j < X; j++)
    		printf("%-3d", b[i][j]);
    	putchar('\n');
	}
	putchar('\n');
	for(i = 0; i < 6; i++)
    {
    	for(j = 0; j < X; j++)
    		printf("%-3d", c[i][j]);
    	putchar('\n');
	}
    return 0;
}
//注意：
//1.printf函数不能一次输出整个数组，只能用访问数组的每个元素，然后逐个输出；
//2.数组的下标是从0开始到n-1结束

//代码输出：
/*
1  1  1  1  1
1  1  1  1  0
1  1  1  0  0
1  1  0  0  0
1  0  0  0  0

1  2  3  4  5
0  0  0  0  0
0  0  0  0  0
0  0  0  0  0
0  0  0  0  0

0  0  0  0  0
0  0  0  0  0
0  0  0  0  0
0  0  0  0  0
0  0  0  0  0
0  0  0  0  0

*/
```





#### 3.使用二维数组：

```c
#include <stdio.h>

int main()
{
	int a[5][5];
	int i, j, x = 1;
	for(i = 0; i < 5; i++)
	{
		for(j = 0; j < 5; j++)
		{
			a[i][j] = x;
			x++;
			printf("%-3d", a[i][j]);
		}
		putchar('\n');
	}
    return 0;
}

//代码输出：
/*
1  2  3  4  5
6  7  8  9  10
11 12 13 14 15
16 17 18 19 20
21 22 23 24 25

*/
```





### 四.定义使用字符数组：

#### 1.定义并初始化一个字符数组：

```c
#include <stdio.h>

int main()
{
	char a[5] = "Happy";
    char b[5] = {"Happy"};
    char c[5] = {'H', 'a', 'p', 'p', 'y'};
    return 0;
}
```



一般大家都会认为会同上面一样定义，但其实不然，这是错误的定义方式。正常是这样定义的：

```c
#include <stdio.h>

int main()
{
	char a[6] = "Happy";//初始化时结尾默认添加'\0'，用来代表字符串结束了，不然电脑不知道字符串什么时候结束
    char b[6] = {"Happy"};
    char c[6] = {'H', 'a', 'p', 'p', 'y'};
    return 0;
}
```



![字符数组](https://s2.loli.net/2023/08/25/LmgSfz4AKsqFyND.png)



既然电脑默认=='\0'==为结束标志，那我们在不确定字符串大小时，便通常这样做：

```c
#include <stdio.h>

int main()
{
    char a[] = {"I am very happy!"}//电脑会自动计算长度，并加一作为'\0'，所以字符数组a的长度为n+1
    return 0;
}
```



#### 2.使用字符数组：

```c
#include <stdio.h>

#define MAXSIZE 100

int main()  
{
	char user[MAXSIZE];
	int t;
	int i = 0;
	while(i < MAXSIZE - 1 && (t = getchar()) != EOF && t != '\n')
	{
		user[i] = t;
		i++;
	}
	user[i] = '\0';//添加字符串终止符

	int j = 0;
	while (user[j])
	{
		printf("%c", user[j]);
		j++;
	}
	putchar('\n');
	printf("%s", user); //输出时用数组名
	return 0;
}
```



```c
#include <stdio.h>

#define MAXSIZE 100

int main()
{
	char user[MAXSIZE];//输入时用数组名
	scanf("%s", user);
	printf("%s", user); //输出时用数组名
	return 0;
}
//注意只能输入一个连续的，不能用空格间隔开，scanf函数是以空格和'\n'为结束标志
```



还有更简单的方法，解决不连续问题我们留到下次第六次培训讲。



### 五.冒泡法排序：

冒泡法是常见的一种排序算法，我们所公认的有六大排序算法，分别是插入排序、希尔排序、选择排序、冒泡排序、堆排序、快速排序。

那什么是冒泡法排序呢？

我们从问题出发，设想有这样一个题，用户随机输入1到10，10个数，我们按从小到大的顺序排序



[冒泡法排序](https://www.bilibili.com/video/BV1qY4y1h7Lz/?spm_id_from=333.337.search-card.all.click&vd_source=5e9a84ef1facd11ea26fbfd213fb295e)



代码实现：

```c
#include <stdio.h>

int main()
{
	int i,j,t,k;
	int a[10];
	for(i=0;i<10;i++)
	{
		scanf("%d",&a[i]);
	}
	for(i=0;i<9;i++)//n个数排序，需要排n-1轮
	{
		for(j=0;j<9-i;j++)//每轮需要排n-1-i次
		{
			if(a[j]>a[j+1])//前大于后，交换
			{
				t=a[j];
				a[j] = a[j+1];
				a[j+1] = t;
			}
		}
	}
	putchar('\n');
	for(i=0;i<10;i++)
	{
		printf("%-2d",a[i]);
	}
	return 0;
}
```



具体每轮如何比较的：

```c
#include <stdio.h>

int main()
{
	int i, j, t, k;
	int a[10];
	for(i = 0; i < 10; i++)
	{
		scanf("%d", &a[i]);
	}
	for(i = 0; i < 9; i++)
	{
		for(j = 0; j < 9-i; j++)
		{
			if(a[j] > a[j+1])
			{
				t = a[j];
				a[j] = a[j+1];
				a[j+1] = t;
			}
			printf("第%d轮第%d次排序:", i+1, j+1);
			for(k = 0; k < 10; k++)
			{
				printf("%-2d", a[k]);
			}
			putchar('\n');
		}
		putchar('\n');
	}
	for(i = 0; i < 10; i++)
	{
		printf("%-2d", a[i]);
	}
	return 0;
}
```





### 六.作业：

1.输入10个数，按从小到大顺序排列并输出排列好的10个数，在此基础上，在输入一个数，按原有规律，将它插入到这个数组中并输出新的排列好的11个数；

2.输入一段英文句子，统计单词数；

3.拓展题（锻炼数学逻辑和代码逻辑）：实现输入数字，输出对应的螺旋矩阵。

例：

```
4
1   2   3   4
12  13  14  5
11  16  15  6
10  9   8   7
```



```
3
1   2   3
8   9   4
7   6   5

```



```c
//1
#include <stdio.h>

int main()
{
	int i, j, t;
	int a[11];
	for(i = 0; i < 10; i++)
	{
		scanf("%d", &a[i]);
	}
	for(i = 0;i < 9;i++)
	{
		for(j = 0;j < 9-i;j++)
		{
			if(a[j] > a[j+1])
			{
				t = a[j];
				a[j] = a[j+1];
				a[j+1] = t;
			}
		}
	}
	putchar('\n');
	for(i = 0; i < 10; i++)
	{
		printf("%-2d", a[i]);
	}
	putchar('\n');
	scanf("%d", &a[10]);
	for(i = 10; i >= 1; i--)
	{
		if(a[i] < a[i-1])
		{
			t = a[i];
			a[i] = a[i - 1];
			a[i - 1] = t;
		}
	}
	putchar('\n');
	for(i = 0; i < 11; i++)
	{
		printf("%-3d", a[i]);
	}
	return 0;
}

//代码示例输出：
/*
2 1 3 6 4 5 7 8 9 10

1 2 3 4 5 6 7 8 9 10
0

0  1  2  3  4  5  6  7  8  9  10
*/
```



```c
//2
#include <stdio.h>

#define MAXSIZE 100

int main()
{
	char c[MAXSIZE];
	int i = 0, words = 0;
	int word = getchar();
	while(word != '\n')//判断句子是否已经结束
	{
		c[i] = word;
		word = getchar();
		i++;
	}
	c[i+1] = '\0';//加上结束标志位
	i = 0;//指针归零
	while(c[i] != '\0')
	{
		if((c[i] >= 'a' && c[i] <= 'z') || (c[i] >= 'A' && c[i] <= 'Z'))//标记是否在单词内
		{
			if((c[i+1] >= 'a' && c[i+1] <= 'z') || (c[i+1] >= 'A' && c[i+1] <= 'Z'))//标记是否在单词内
			{
				words = words;
			}
			else
				words += 1;
		}
		i++;
	}
	printf("words = %d\n", words);
	return 0;
}

//代码示例输出：
/*
I am very happy.
words = 4

*/
```



```c
//2
#include <stdio.h>

#define MAXSIZE 100

int main() 
{
	char a[MAXSIZE];
    int i, in_word, word_num;
    gets(a);
    word_num = 0; // 初始化单词个数为0
    in_word = 0; // 标记位，标记是否在单词内
    for(i = 0; a[i]; i++)
	{
        if (a[i] == ' ') // 检测到空格
		{ 
            in_word = 0; // 设置标记位为不在单词内
        } 
		else if (in_word == 0) // 在单词内
		{ 
            word_num++; // 统计单词个数
            in_word = 1; // 设置标记位为在单词内
        }
    }
    printf("words = %d", word_num);
}

//代码示例输出：
/*
I am very happy.
words = 4

*/
```



```c
//拓展法一
#include <stdio.h>
#define N 1000

int arr[N][N];//存储螺旋矩阵 
int main() 
{
	int dx[] = {0, 1, 0, -1};//右走下走左走上走 行列的方向向量 
	int dy[] = {1, 0, -1, 0};
	int n;//用户输入的范围 
	int cnt = 1, low = 0;//记录数字到哪了和方向向量用到哪一个了 
	int i, j;//循环变量 
	int x = 0, y = 0, tx, ty;//现在走到的坐标和下一步计算出的坐标 
	scanf("%d", &n);
	while(cnt <= n * n)
	{
		arr[x][y] = cnt;
		cnt++;
		tx = x + dx[low];
		ty = y + dy[low];
		if(tx >= n || tx < 0 || ty >= n || ty < 0 || arr[tx][ty]) //越界或已经有数字了 
		{
			low = (low + 1) % 4;//更新方向向量 
		}
		x += dx[low], y += dy[low];//移动到下一个坐标
	}
	for(i = 0; i < n; i++)
	{
		for(j = 0; j < n; j++)
			printf("%-3d", 	arr[i][j]);
		putchar('\n');
	}
}
```



```c
//拓展法二
#include<stdio.h>

int main()
{
	int i, j, num[20][20];
	int n, m, x = 1;
	scanf("%d", &n);
	m = n-((n+1)/2);//奇数中心点的位置 
	for(i = 0; i < n/2; i++)//转的圈数 
	{
		if(n%2 != 0)//奇数中心点 
		{
			num[m][m] = n*n;
		}
		for(j = i; j <= n-i-1; j++)//右 
        {
            num[i][j]=x++;
        }
        for(j = i+1; j <= n-i-1; j++)//下 
        {
            num[j][n-i-1]=x++;
        }
       for(j = n-i-2;j >= i; j--)//左 
       {
           num[n-i-1][j]=x++;
       }
       for(j = n-i-2; j>i; j--)//上 
       {
           num[j][i]=x++;
       }
	}
	for(int i = 0; i < n; i++)
    {
        for(int j = 0; j < n; j++)
            printf("%-4d", num[i][j]);
        printf("\n");
    }
}
```



```c
//拓展法三
#include<stdio.h>

int main()
{
	int a[10][10];//最大方阵； 
	int n=0;
	scanf("%d",&n);
	for(int i=0;i<n;i++)//方阵的初始化；
	{
		for(int j=0;j<n;j++)
		{
			a[i][j]=-1;
		}
	}
	int num=1;//（填数器） 
	int flag=1;//flag=1向右;flag=2向下;flag=3向左;flag=4向上;（转向器） 
	int i=0,j=0;
	while(num<=n*n)//填进去的数的最大条件；
	{
		if(a[i][j]==-1)//a[0][0]赋值为1； 
		{
			a[i][j]=num;
			num++;
		}
		if(flag==1)//开始向右赋值； 
		{
			j++;
			if(j==n||a[i][j]!=-1)//向右赋值的条件1.未被赋值；2.没有触底；
			{
				j--;
				flag=2;
			}
		}
		else if(flag==2)//向下赋值的条件1.未被赋值；2.没有触底；
		{
			i++;
			if(i==n||a[i][j]!=-1)
			{
				i--;
				flag=3;
			}
		}
		else if(flag==3)//向左赋值的条件1.未被赋值；2.没有触底；
		{
			j--;
			if(j==-1||a[i][j]!=-1)
			{
				j++;
				flag=4;
			}
		}
		else if(flag==4)//向上赋值的条件1.未被赋值；2.没有触底；
		{
			i--;
			if(i==-1||a[i][j]!=-1)
			{
				i++;
				flag=1;
			}
		}
	}
	for(i = 0; i < n; i++)
    {
        for(j = 0; j < n; j++)
            printf("%-4d", a[i][j]);
        printf("\n");
    }
	return 0;
}
```

