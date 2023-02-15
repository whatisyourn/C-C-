# C语言代码常见错误码修复

## 1.`error LNK2019`

`error LNK2019`问题在VC 6.0中是error LNK2001: unresolved external [symbol](https://so.csdn.net/so/search?q=symbol&spm=1001.2101.3001.7020)问题，可能错误号改了。

编译时出现类似这样的错误：Dlgcode.obj : error LNK2019: 无法解析的外部符号 _readRegmark，该符号在函数 _AboutDlgProc@16 中被引用。这种错误的<font color='red'>本质是链接器无法在已编译的obj、lib或dll文件中找到函数定义。</font>

1、这是百度找到的方法：http://jingyan.baidu.com/article/4d58d54135d7a79dd4e9c0ad.html。就是有头文件（有了函数声明）却没有lib。一般出现于你使用了第三方提供的库，下载了头文件却忘了载库文件，或库文件忘记放到相应的目录下了。

2、你自己写的函数声明的头文件也写了函数定义的cpp文件，却依然出现LNK2019错误。可能原因：<font color='red'>忘记将这两个文件加入工程了</font>。

一般出现于用Visual Studio和记事本（或UltraEdit）混合开发过程，你用记事本include了相应的头文件，却忘了在Visual Studio的工程中加入它们了。也可能出现于在解决方案的开发过程，在解决方案下的某个工程中加入了它们却忘了在其他工程中加入，经常在一个工程中写了共享的源代码，却忘了在别的工程中加入它们。这个问题类似于第1个，不同的是这个库是你自己提供的，但没有把它交给VS 2008编译出来。

3、你自己写的函数声明的头文件也写了函数定义的cpp文件也加入工程了而且你很确定函数体肯定是在这个库文件中，却依然出现LNK2019错误。可能原因：C语言和C++语言混编，因为C++支持函数重载所以C++编译器生成的库文件中的函数名会面目全非，例如C编译器会生成 _readRegmark 这个函数名，而C++编译器则生成了"void __cdecl readRegmark(char *)" (?readRegmark@@YAXPAD@Z)这么个函数名。当你的函数是用C语言写的，VS编译器会按C语言规则编译，但链接器却不知道还傻傻的用C++规则的函数名去找结果就找不到了，而你还百般肯定TM的不就在这个库中吗你个睁眼瞎。解决：在C语言的头文件中加入

```
#ifdef __cplusplus
extern "C" {
#endif

void readRegmark(char *regmark);  //这里写函数声明

#ifdef __cplusplus
}
#endif
```

给链接器提示这个函数是C语言的，别TM找错了。

4、http://blog.csdn.net/jtop0/article/details/5779782。模板声明和实现要放在同一文件夹中。

5、http://www.programlife.net/error-lnk2019.html。内联函数定义在头文件中。

6、百度的。http://jingyan.baidu.com/article/d621e8da0d7c022864913f40.html。错误的工程类型造成的。

7、<font color='red'>函数定义没有放在函数/类声明的名称空间中namespace</font>

一般类外实现要加作用域.

友元函数不存在类外实现,因为不是类的成员方法,所以不用加作用域

8、貌似还有不尽之处。http://www.douban.com/note/65638800/。