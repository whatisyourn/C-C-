# C语言

## 一、C基础串讲

### C语言知识构成

- 程序逻辑
  * 数据类型、运算符、分支逻辑、循坏逻辑
  * 函数的概念
- C语言的特有内存访问方式
  * 指针、数组、（特殊的指针）
  * 结构体、共用体
  * 函数指针
  * 多维空间、多维指针(==伪命题==)
- C语言函数设计的逻辑
- C语言编译原理（多文件编译)

### C语言工具概念

- 编辑器、编译器（编译、链接）、调试器

> 人语言 编辑了一种计算机的逻辑    --》  指令 让CPU能够按照指令逻辑运算 
>
> ​                                                     （编译、链接）
>
> `int a = 10;`    <!--分配一个空间是4B 用10赋值-->



在形成可执行文件前会发生错误，包括编译错误，链接错误

编译错误（语法错误） 

​	形如xxx.c[15] error:......

链接错误 

​	形如xxx.o LINK ERROR 原材料不够/多了

执行错误

​	例如，数组越界

- IDE工具、独立编译器

##### 1.从赋值谈谈计算机如何工作

` `

![image-20230117204821101](C%E8%AF%AD%E8%A8%80.assets/image-20230117204821101.png)

> 在程序中存在映射表，将变量与内存空间联系起来
>
> 指令的三个问题：分配多大空间 空间分配在哪里 什么时候进行分配的

```c
#include<stdio.h>
//1.main的返回值不同 在系统之上一般用int 在裸机上用void
void main(int argc, char const *argv[])
{;}
int main(int argc, char const *argv[])
{;}

//2.main函数不加入参数 C90版本标准要加void C99可以不加
int main()
{;}
int main()
{;}
//3.unsigned int a = -100 有错误吗，输出结果是什么 
int main(int argc, char const *argv[]){
    unsigned int a = -100;
    printf("%d\n", a);
    return 0;
}
//unsigned 说明最高位不当成符号位 不限制赋值的行为但是限制了后面的数学运算行为
//int 说明占用4个字节（大小与编译器有关）
// -100 100000 是整形常量 占4个字节
//%d是以有符号类型去看待内存的值 %u以无符号看待。只是以不同行为看待不同，但是内存的值都是一样的
```

- C语言的上帝视角

  - CPU、内存、硬盘

  - 程序=定义内存里空间的布局+什么样的方式来操作内存

- C语言的障眼法

  printf以不同视角看待相同内存，打印出不同的值

  ```c
  int main(int argc, char const *argv[]){
      unsigned int a = -100;
      printf("%x\n", a);
      return 0;
  }
  //输出ffffff9c
  
  int main(int argc, char const *argv[]){
      unsigned char a = -100;
      printf("%x\n", a);
      return 0;
  }
  //输出9c
  
  int main(int argc, char const *argv[]){
      unsigned char a = 256; //2^8= 1 0000 0000
      printf("%x\n", a);
      return 0;
  }
  //输出0 将低8位直接赋值给a
  ```

  为什么下面代码可以通过编译?

  ```c
  int main(int argc, char const *argv[]){
      char str = "123";
      printf("%x\n", a);
      return 0;
  }
  //"123"是字符串==空间的首地址==空间 
  ```

  

![image-20230117212227727](C%E8%AF%AD%E8%A8%80.assets/image-20230117212227727.png)

- 数字的计算机表示方法

  - 1.数字的计算机表示方式 原码\反码\补码

    ​	计算机都是用补码存储,在计算的时候,如果是减法,可以把减法看成加法

  - 2.unsigned和signed的限制

    ​	内存空间的存储形式,C语言代码的查看行为

##### 2.C语言常用关键字

|  **auto**  | **break**  |   **case**   |  **char**  |  **const**   | **continue** | **default**  |   **do**   |
| :--------: | :--------: | :----------: | :--------: | :----------: | :----------: | :----------: | :--------: |
| **double** |  **else**  |   **enum**   | **extern** |  **float**   |   **for**    |   **goto**   |   **if**   |
|  **int**   |  **long**  | **register** | **return** |  **short**   |  **signed**  |  **sizeof**  | **static** |
| **struct** | **switch** | **typedef**  | **union**  | **unsigned** |   **void**   | **volatile** | **while**  |

==数据类型描述空间大小==

###### 2.1auto与register

|          |             C             |           C++            |
| :------: | :-----------------------: | :----------------------: |
|   auto   |         普通变量          | C++11中,自动推导变量类型 |
| register |   建议分配到寄存器中去    |                          |
|  static  |    变量放在静态数据区     |                          |
|  const   |       只读(建议型)        |                          |
|  extern  | 链接,有个东西在别的文件里 |                          |

```c
(auto) int a = 10;//定义了一个4B的空间,空间是自动分配,默认在变量前加
(register) int a = 10;//定义了一个4B的空间,空间建议分配到寄存器中去,建议型关键字
```

###### 2.2数据类型关键字

- 什么是数据类型
  - 定义内存空间大小的一个代名词,方便编译器能合理转换为对应的指令
- 数据类型使用注意事项
  - 越界的问题
  - 大小由sizeof来决定

- 修饰 unsigned 和signed

- 自定义数据类型 数组 结构体static 共用体union 枚举enum

###### 2.3逻辑关键字

​	分支语句: if..else switch...case break default

​	循坏语句: do..while while for continue break goto

###### 2.4 sizeof

###### 2.5 volatile

```C
int a = 10;
int fun(){
    while(a == 10);
    printf("========\n");
}
//从a中的内存地址取出值放在cache里
//从cache里取值
//看值是否为10循坏等待
int sig(){
    a=100;
}


volatile int a = 10;//加入volatile后编译器说明这个变量是易变的所以不会对值进行缓存
int fun(){
    while(a == 10);
    printf("========\n");
}
//从a中的内存地址取出值
//看值是否为10循坏等待
int sig(){
    a=100;
}
```

##### 3.常用标点符号

- +、-、*、/、%

- &、|、^、~

- &&、||、|

- <、>、>=、<=、==

  ```C
  //^
  //n个数找出唯一的那个数
  //交换两个数
  ```

  

## 二、C语言预处理技术

### 1.C语言预处理机制

- 预处理的时机
  - ==在编译前，只做替换，不做语法检测==
  - C语言到相关体系结构下可执行文件的过程
- 预处理的语法形式
  - ==#开头==
  - 接入特殊字符串：include define ifdef ifndef...
- 预处理的意义

```C
#define ABC 10//形成一个宏表和映射表
/* ABC */  //这里可能会变
int main()
{
    int a = ABC;
    
    int b = a+10;
    printf("%d\n",b);
    return b;
}
```



#### 1.1条件编译

- 调试期、发布期
  - #ifdef...#else...#endif
  - 内置变量：`__FILE__`、`__FILE_NAME__`、`__FUNCTION__`、`__LINE__`

```C
//想调试时执行，发布时不执行 
#include<stdio.h>

int main(){
    int a = 10;
    int b = a+10;
#ifdef DEBUG
    printf("%d\n",b);
#endif
    printf("=============\n");
    return b;
}
```

编译器提供了一个特殊开关，让程序员通过命令的的方式，在预处理前人为定义一个宏

在gcc中

```linux
gcc -D DEBUG -o build hello.c
```

在VS中

![image-20230119200655292](C%E8%AF%AD%E8%A8%80.assets/image-20230119200655292.png)

![image-20230119200758599](C%E8%AF%AD%E8%A8%80.assets/image-20230119200758599.png)

![image-20230119200848393](C%E8%AF%AD%E8%A8%80.assets/image-20230119200848393.png)

在最下面加入，实现开关的定义。

- 针对不同平台

#### 1.2宏替换以及#、##的应用

- 替换
- 宏函数 
  - 普通替换
  - #作用
  - ##作用

```C
//宏函数替换要加括号，是直接替换不会在替换前进行运算
#define ABC(x,y) x+y
#define ABC(x,y) ((x)+(y))
```

```C
//#和##的区别
//前缀+变量
#include<stdio.h>
#define ABC 10
#define min(x,y) ((x)<(y) ? (x):(y))
#define S_LOG(LEVEL) printf("MY_PRJ"#LEVEL)//#进行字符串替换拼接，理解为字符串
#define POS(x) point##x//##是进行变量拼接理解为单词

int main(){
    int pointA = ABC;
    int pointB = 20;
    printf("the a = %d\n", pointA);
    S_LOG(DEBUG);//S_LOG(DEBUG)=printf("MY_PRJDEBUG")
    printf("\nthe v = %d\n", POS(B));
    printf("%d\n",min(POS(A),POS(B)));//POS(A)=pointA,POS(B)=pointB
    return 0;
}
```



#### 1.3头文件引入

```C
//头文件的一开始要加警卫,防止重定义
#ifndef A_H
#define A_H
....
#endif
```

## 三、C语言内存管理

### 1.计算机程序运行机制简介

- CPU主要对指令进行解析、执行，受限于芯片体积、功耗、成本因素。CPU内部很难做到大存储，这样CPU要执行的指令和数据必须外置
- 内存芯片，往往提供地址总线和数据总线，方便CPU直连

#### 1.1CPU和内存关系

![image-20230120210921976](C%E8%AF%AD%E8%A8%80.assets/image-20230120210921976.png)

#### 1.2两大体系结构

- 冯诺依曼结构

![image-20230120211002888](C%E8%AF%AD%E8%A8%80.assets/image-20230120211002888.png)

- 哈佛结构

![image-20230120211038090](C%E8%AF%AD%E8%A8%80.assets/image-20230120211038090.png)

### 2.C语言内存访问空间方式

#### 2.1变量名本质

- 对一段内存空间，方便程序员描述，让编译器可以正确==映射关系==的**名称**
- 编译器帮助程序员，将这段空间访问的代码，翻译成正确的CPU指令
- 变量名的注意事项：
  - 编译器能正确识别 =》 有效字符
  - 不能有重复的变量名 =》 一个变量名代表一个专属空间
  - 变量名必须要有类型 =》这样空间才有约束
  - 变量名有作用域 =》 申请空间和释放空间

> 

#### 2.2指针的本质

- 保存地址的容器，C语言的地址和计算机的地址，在定义时发生了改变
- C语言地址如何映射到计算机底层地址：
  - C语言地址定义
  - C语言之定义了地址形态，具体怎么使用，全部交给程序员来处理
    - 不管越界访问
    - 不管有没有访问权限
  - C语言定义地址的访问：
    - *、[]
  - 多维指针的逻辑模型

> 指针==内存地址 ：值 （数字）+ 操作方式（一次操作多少单位）

### 3.C语言内存空间结构化分配

#### 3.1一维数组空间及访问

- 一维数组的编译器行为（语法）
  - C语言只管分配，不管越界，因为在C语言看来，内存就是连续的，访问的权限应该交给程序员，而不是编译器。
    - 分配关心的是个数和每个元素的大小
  - 数组名不是变量名，只是为了标识这个空间，引入的常量标签（C语言压根没有数组变量这个概念）

> int a1[10];    a1 ---------- 数组名（连续空间的首地址）
>
> ​					 10 ----------- 个数
>
> ​                      int ----------- 按一个整形操作
>
> sizeof(p)=8B;
>
> sizeof(a1)=10*sizeof int;
>
> void fun(int a2[100]){
>
> sizeof(a2)=8B;
>
> }

- 特殊的一维数组
  - 字符串和字符数组
  - 重要的字符串空间处理函数
    - strlen/sizeof
    - strcpy/strncpy
    - memcpy/memset单位B
    - sprintf/snpritf
    - printf的%s的含义

#### 3.2二维以及多维数组空间以及访问

- 二维数组的内存模型
  - 计算机内存是线性编址，一维空间如何描述人类社会的二维空间
  - 二维空间的地址本质 
  - > int a[3] [4]
    >
    > int (*key) [4] = a;
- 三维或者多维数组的内存模型
- ==main函数参数空间运行分析==

  - char**args和char buf[4] [5]的区别  

- 怎么看变量名：编译器完成映射表的过程 名字   访问方式 先向右再向左

```C
int *p1; char *p2; double *p3;
char *a1[4];
char (*a2)[4];

char *b1(int, int);
char (*b2)(int,int);

char *c1[3][2];
char (*c2)[3][2];

char (*d1[5])(int, int);

char **key = a1;
```

| 名字 | 访问方式                                                     |
| ---- | ------------------------------------------------------------ |
| p1   | *  (4B/8B)  -----按照int方式一个对象一个对象访问             |
| a1   | 数组  （4个）（*） ------ 按照char的方式一个对象一个对象访问 |
| a2   | * --------[]数组（4个） （char)                              |
| b1   | （）函数 输入参数 返回值                                     |
| b2   | * 指针 ------函数（） 传入参数 返回值                        |
| c1   | [] []数组 放钥匙 * -------按照char方式一个一个对象访问       |
| c2   | * 指针 -----[] []数组 char                                   |
| d1   | * 指针 ------ [5] 数组 -------（）函数 传入参数 返回值       |



#### 3.3结构体空间和数组空间

- 区别和应用场景
- 语法说明

### 4.C语言内存分段管理以及权限管理

#### 4.1一个经典错误

```C 
int main(){
    char *str = "2021";
    str[3] = '2';
    printf("str=%s\n",str);
    return 0;
}
```

#### 4.2内存分段管理模型和权限

![image-20230120212721435](C%E8%AF%AD%E8%A8%80.assets/image-20230120212721435.png)

|          | 段             | 属性                                                         |
| :------: | -------------- | ------------------------------------------------------------ |
| 4G-3G OS | kernel         | 不可读，不可写，被操作系统中指                               |
|          | stack段        | 所有函数局部变量都在这里  RW。每个子函数都拥有一个独立的栈空间。每个子函数的栈空间都有大小限制，一段操作这个值，栈会溢出 |
|          | 堆段           | C语言编译器不维护，由程序员自己维护。空间没有限制 RW  通过malloc申请空间，通过free释放空间，但该空间依然可以访问。如果不释放，会导致内存泄漏变慢 |
|          | 数据段         | 不依赖于函数调用而定诞生，生命周期从程序运行开始，道程序运行结束全局变量、静态变量 |
|          | 只读数据段     | 双引号、常量数字                                             |
|          | 代码段         | R                                                            |
|          | 操作系统保护区 | No RW                                                        |



- 各段使用注意事项
  - 低地址段和3G以上的段，不可读，不可写，被操作系统保护，写和读都会发生段错误
  - 栈空间由编译器编译时，保存内存 资源的释放和申请
  - 堆空间必须由程序员来维护（内存泄漏的重灾区）

#### 4.3内存各段空间分配和使用

- 代码段.text
- 只读数据段.rodata
- 数据段.data
  - static区
  - 全局区
- 堆段
- 栈段

## 四.C语言内存面试常见题（变量类型、地址访问）

常见题

### 1.已知代码如下，填空

```C
unsigned char *p1;
unsigned long *p2;
p1 = (unsigned char *)0x801000;
p2 = (unsigned long *)0x810000;
```

p1 + 5 = _________ ; p2 + 5 = __________ ;  

> 0x801000+5*sizeof(char) =0x801005
>
> 0x810000+5sizeof(long)=0x8010014

```C
#include "stdio.h"
void main() {
    int a[5] = {1, 2, 3, 4, 5};
    int b[5] = {0, 2, 1, 3, 0};
    int i, s = 0;
    for(i = 0; i < 5; i++)
    	s = s + a[b[i]];
    printf("%d\n", s);
}
```

打印结果是 _________;  

> s = 0 + 1 + 3 + 2 + 4 + 1=11

在64位系统下，分别定义如下两个变量：  

```C
char *p[10]; char(*p1)[10];
```

请问， sizeof(p)和sizeof (p1)分别值为:____.  

> 第一个p为10*8B  第二个为8B

### 2.请说明下⾯变量表示的是什么  

```C
int (*s[10])(int);
void (*signal(int,void(*)(int)))(int);
```

> s是数组，每个元素都是是指针，每个指针指向函数
>
> signal----（） 函数名
>
> ​			输入参数(int,void(*)(int)))  一个int 一个函数地址
>
> ​			返回值  *地址-------返回一个函数地址，这个函数返回值void   传入参			数int
>
> typedef void (*handerl) (int)
>
> handerl signal (int, handerl); 

### 3.在64bit系统下，  下列代码输出的是什么  

```C
#include <stdio.h>
char buffer[6] = {0};
char *mystring(){
    char *s = "Hello world";
    for(int i = 0;i < (sizeof(buffer) - 1); i++){
    	buffer[i] = *(s+i);
    }
    return buffer;
}
int main(int args,char**argv){
    printf("%s\n",mystring());
    return 0;
}
```

> 由于数组结尾的/0存在故结果为Hello

4. ### 以下代码⽚段，有问题没有，如果有，请指出，没有的话，给出答案。  

```C
void UpperCase(char str[]) {
for( size_t i=0; i<sizeof(str)/sizeof(str[0]); ++i) {
    if('a'<=str[i] && str[i]<='z')
    	str[i] -= ('a'-'A' );
	}
}
char str[] = "aBcDe";
cout << "str字符⻓度为: " << sizeof(str)/sizeof(str[0]) << endl;
UpperCase(str);
cout << str << endl;
```

5. ### 下⾯程序的运3⾏结果：  

```C
int main() {
	int a[5] = {1, 2, 3, 4, 5};
	int *ptr = (int *)(&a+1);
	printf("%d %d\n", *(a+1), *(ptr-1));
}
```

### 6.下⾯程序的运⾏结果是：  

```C
void test2() {
    int i;
    char **p;
    int msg[16] = {0x40, 0x41, -1, 0x00, 0x01, -1, 0x12, 	-1, 0x20, 0x27, 0x41, 0x35, -1, 0x51, 0x12, 0x04};
    char *strArr[] = {"beijing", "shanghai", "guangzhou", "guangdong","Tokyo",
    "American"};
    char *(*pStr)[6];
    pStr = &strArr;
    p = strArr;
    for (i = 0; i < 16; i++) {
        if (msg[i] == -1) {
            putchar(' ');
            continue;
        } else if ((msg[i] & 0xF0) == 0x40) {//取出高四位为4
            putchar(p[msg[i] >> 4][msg[i] & 0x0F]);
            //等价于putchar(p[4][msg[i] & 0x0F]);
            continue;
        } else if ((msg[i] & 0xF0) == 0x30) {//取出高四位位3
            putchar(*(strArr[msg[i] >> 4] + (msg[i] & 0x0F)));
            //等价于putchar(*(strArr[3] + (msg[i] & 0x0F)));
            continue;
        } else {
            putchar(*((*pStr)[msg[i] >> 4] + (msg[i] & 0x0F)));
        }
    }
    printf("\n");
}
```

> To be a good man

7. ### 若有以下输⼊（代表回⻋换⾏符），则下⾯程序的运⾏结果为:  

```C
1, 2<CR>
int main(void) {
    int a[3][4] = {1,2,3,4,
                   5,6,7,8,
                   9,10,11,12};
    int (*p)[4], i, j;
    p = a;
    scanf(“%d,%d”,&i,&j);
    printf(“%d\n”, *(*(p+i)+j));
    return 0;
}
```

> 第二行第3个元素为7即a[i] [j]

## 五、函数设计面试题

> 函数语法:
>
> ​	特殊的代码空间
>
> ​	函数名进行标记 === 地址常量(不能放在等号左边)
>
> ​	输入参数 返回值

> main函数执行流程
>
> ​	程序 OS进行初始化 内存 资源
>
> ​             调用main函数（只是中间的过程）
>
> ​             返回OS的资源释放区
>
> ​             OS回收 内存 资源

```C
#include<stdio.h>

__attribute__ ((constructor)) int my_start(){
    printf("我是入口，我比main先执行");
    return 0;
}

__attribute__ ((destructor)) void my_exit(){
    printf("我是出口，我在main后执行");
}

__attribute__ ((.text)) int main(){
    printf("在main这里");
    return 0;
}
```

### 函数的承上启下的作用

- 承上
  - 接受调用者传递信息 (调用者将参数内容拷贝给函数形参)
  - 如果调用者想把自身空间的值，让子函数去改变，输入参数反向更新上层空间
    - 调用这个子函数时，必须传递这个空间的地址
    - 子函数中，必须使用*或者[]间接访问调用者空间，放在等号左边
- 启下
  - 通过返回值进行更新

```C
void fun1(int *addr){
    addr = 200;
}
//不能改变a和data
void fun1(int *addr){
    *addr = 200;
}
//改变了data
void fun2(int **addr){
    *addr = 200;
}
//改变了a
int main(){
    int data = 100;
    int *a = &data;
    fun(a);
    fun2(&a);
    printf("%d\n", data);
    printf("%d\n", *a);
    return 0;
}
```



1. 以下代码有什么错误，运⾏结果是什么？

  ```C
  char *s = "AAA";
  printf("%s", s);
  s[0] = 'B';
  printf("%s", s);
  ```

2. 请分析下⾯的代码，分析有什么问题（共4题）  

```C
/* 第⼀题 */
void getMemory(char *p) {
	p = (char *)malloc(100);
}
void test1(void) {
    char *str = NULL;
    getMemory(str);
    strcpy(str, "hello world");
    printf("%s", str);
}

//str不会被改变因为没有传地址进去所以str依旧是空
//正确版本
void getMemory(char **p) {
	*p = (char *)malloc(100);
}
void test1(void) {
    char *str = NULL;
    getMemory(&str);
    strcpy(str, "hello world");
    printf("%s", str);
}
```

```C
/* 第⼆题 */
char *getMemory(void) {
	char p[] = "hello world";
	return p;
}
void test2(void) {
    char *str = NULL;
    str = getMemory();
    printf("%s", str);
}
//p在出函数会被释放可以改成statiC放在静态区中
```

```C
/* 第三题 */
void getMemory(char **p, int num) {
	*p = (char *)malloc(num);
}
void test3(void) {
    char *str = NULL;
    getMemory(&str, 100);
    strcpy(str, "hello");
    printf("%s", str);
}
//版本非常正确，但是没有释放 在最后free str
```

```C
/* 第四题 */
void swap(int *p1, int *p2) {
    int *p;
    *p = *p1;
    *p1 = *p2;
    *p2 = *p;
}
//应该使用int p而不是指针，因为不初始化会进行默认初始化，非法访问
```

3. 请问下列代码有什么问题  

```C
int main() {
    char a;
    char *str = &a;
    strcpy(str, "hello");
    printf(str);
    return 0;
}
//str指向空间不够大，会产生越界访问
```

4. 以下代码，会产⽣什么问题？  

```C
char szstr[10];
strcpy(szstr,"0123456789");
//会产生越界访问，因为将\0放在了第
```

## 六、多文件编译原理

### 6.1编译需要的元素

- 自定义类型的内存模型
- 函数操作模型
- 预处理的符号

> 首先独立编译 将各个.c文件编译成.o文件
>
> 然后进行多目标集合 a.o + b.o +c.o = build

### 6.2头文件的作用

- 自定义数据类型，函数声明的统一化载体
- 由于有统一的定义规范，编译器编译后的链接确保不会出错

### 6.3链接问题

- 没有定义符号
- 重复定义符号
- 
