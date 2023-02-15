# 一:C++语言和C语言区别

## 1.1 C语言是面向过程的语言,C++是面向对象的范编程

```C++
#include<iostream>

using namespace std;

int main(){
    cout << "hello world" << endl;//endl:换行 cout:打印 <<:符号重载
    system("pause");//进行阻塞
    return EXIT_SUCCESS;
}
```

面向对象的三大特性:封装、继承、多态

## 1.2 双冒号作用域运算符（：）

未使用优先使用就近的声明

双冒号表示全局作用域

```C++
#include<iostream>

using namespace std;

int a = 20;

int main(){
    int a = 10;
    cout << a << endl;//10
    cout << ::a << endl;//20
    
    system("pause");
    return EXIT_SUCCESS;
}
```

## 1.3 namespace的使用

```C++
namespace AAA
{
    //code here(可以是函数/变量/结构体/类)
}
```

用途:解决命名冲突的问题

注意:

- 命名空间必须定义在全局作用域下
- 命名空间可以嵌套命名使用`A::B::C::AAA`进行声明使用
- 命名空间是开放的,可以随时增加内容.相同命名空间内容会进行合并
- 无名、匿名命名空间等无名命名空间时相当与于加入了static静态变量
- 命名空间可以起别名 `namespace B = A`; 

## 1.4 using声明和using编译指令

方式一:`using B::m_c`

using声明要注意避免二义性问题,

<font color='red'>用了using声明代表以后的m_c都代表B命名空间下的</font>

方式二:`using namespace B`

using的编译指令相当于打开了房间,也要注意避免二义性.

<font color='red'>如果局部范围有相同命名的变量,优先使用局部的</font>

## 1.5 C++对C语言的增强

### 1.5.1全局变量

```C++
int a;
int a = 10;
```

在C中不会报错,但若是局部变量也存在重定义.

C++会报错,不允许重定义行为

### 1.5.2函数检测增强

```C++
int get(w, h)
{
    ;
}
```

<font color='red'>C中会有警告 C++会报错,函数参数类型没有</font>

```C++
get(a,b,c);
```

<font color='red'>C中无警告 C++会报错,不接受三个参数,没有与匹配的重载函数实例</font>

### 1.5.3类型转换增强

```C++
char *p = malloc(4) //返回的时void *类型
```

C中警告 C++会报错

### 1.5.4 struct增强

```C++
struct person{
    int age;
    void add();
}
```

C中会报错,struct不能加函数,且使用必须加struct关键字

<font color='red'>C++可以加函数,使用不必加struct关键字</font>

### 1.5.5布尔类型增强

```C++
bool flag;
flag = 100;
```

C中不允许布尔类型关键字

C++中允许 bool类型关键字大小为1字节只有真假

### 1.5.6 三目运算符

```C++
int a = 10;
int b = 20;
(a > b ? a : b )= 100;
```

C语言不允许, 20=100是赋值, 返回的是值 *(a > b ? &a : &b )= 100;

<font color='red'>C++允许,返回的是变量 相当于 b = 100是赋值</font>

### 1.5.7 const增强(尽量使用const 代替#define)

```C++
int main()
{
    const int a = 10;
    int *p = (int *)&a;
    *p = 20;
    int arr[a];
}

```

C中可以通过指针绕过const修改局部变量,但全局变量受到保护不可以被改,且此时a为常变量

<font color='red'>C++中a是真常量,可以用来初始化数组 此时 *p=20,a=10;</font>

原因:C语言中,const修饰的变量是伪常量,编译器会为其分配内存

​		 C++中,const不会分配内存,而是放在符号表中

上面代码相当于下面代码

```C++
int main()
{
    const int a = 10;
    int tmp = a;//生成了一块临时空间
    int *p = (int *)&tmp;
    *p = 20;
    int arr[a];
}
```

## 1.6 Const的链接属性

```C++
//test.c中
const int a = 10;

//test1.c中
extern const int a;
```

C语言可以成功,因为C语言中Const默认是外部链接

<font color='red'>C++失败,因为const默认为内部链接,要用extern提高作用域</font>

```C++
//test.c中
extern const int a = 10;

//test1.c中
extern const int a;
```

## 1.7 const分配内存的情况

- const分配内存 取地址会分配一块临时内存
- extern编译器也会分配变量
- 用普通变量初始化const变量
- 自定义数据类型

```C++
class person{
  	int a = 10;  
};

extern const int b = 20;//分配空间
int main(){
    const int a = 10;//不分配内存
    int *p = (int *)&a;//分配一块临时空间
    
    int c = 20;
    const int d = c;//分配空间
    
    const person p;//分配空间
}
```

## 1.8 &引用的基本语法和注意

<font color='red'>&左边引用,右边取地址</font>

语法: `Type & 别名 = 原名`

<font color='red'>引用必须初始化,且初始化之后不能进行修改</font>

### 1.8.1对数组的引用

```C++
//方式一
int arr[10] = {0};
int &parr[10] = arr;
//方式二
typedef int(ARRAY)[10];
ARRAY &parr = arr;
```

## 1.9引用注意事项

- 引用必须引一块合法的内存空间 `int &a = 10 //错误`

- 不要返回局部的引用 因为局部变量被销毁了,如果要返回要加static关键字保存
- 若函数返回值为引用那么这个函数可以作为左值被修改

```C++
int &test(){
    static int a = 10;//添加static
    rturn a;
}

int main()
{
    int &b = test();
    test() = 10;//作为左值
}
```

## 1.10引用的本质

```C++
int & b = a;   //等价于 int * const b = &a;
b = 20; // 等价于 *b = 20;
```

C++中编译器会自动进行转换,本质为指针常量

## 1.11指针引用

```C++
int *& pa = p;
```

## 1.12常量引用

```C++
int &ref = 10;//错误没有分配内存
const int & ref = 10;//等价于  int tmp=10; const &ref = tmp;
```

用来修饰形参`test(const int &a)`