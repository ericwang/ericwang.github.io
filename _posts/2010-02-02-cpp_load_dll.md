---
layout: post
title: 动态加载DLL的方法与注意的问题
description: ""
category: program
analytics: true
tags: [c++, dylib, 动态, 加载, __declspec, dllexport]
---

加载DLL的方法主要有两种：一种是隐式链接，另外一种是动态加载。
隐式链接会把DLL中所有标志为\_\_declspec(dllexport)的函数都加载，如果有多个DLL加载时，可能会影响到程序执行的效率。而用动态加载DLL的方式则可以根据需要去加载用到的函数。
动态加载DLL的方法：
1.把生成的.DLL文件复制到测试工程DLLTest目录下。这里假设该.DLL文件为add.dll，主要代码是：

{% highlight cpp %}
__declspec(dllexport) int add(int x, int y)
{
    return x + y;
}
{% endhighlight %}

2.在DLLTest工程中添加DllTest.cpp文件.
首先使用LoadLibrary("add.dll")加载add.dll文件:

{% highlight cpp %}
HMODULE hmod = LoadLibrary("add.dll");
{% endhighlight %}

然后定义一个函数指针的类型：

{% highlight cpp %}
typedef int (*AddAddr)(int x, int y);
{% endhighlight %}

注意，这里的参数与返回类型务必与add.dll文件中函数add的声明一样。

接着：

{% highlight cpp %}
AddAddr Add = (AddAddr)GetProcAddress(hmod, "add");
//如果Add值为空，则获取函数的地址失败！
if(!Add)
{
    printf("获取函数地址失败！");
    return;
}
{% endhighlight %}

最后，可以测试一下：

{% highlight cpp %}
printf("test add(): 1+2=%d", add(1,2));
{% endhighlight %}

运行结果一看，会出现“获取函数地址失败！”。为什么会这样？

打开命令行，用cd命令到add.dll工程目录的debug目录下，然后使用命令：
dumpbin -exports add.dll

则会看到add.dll文件中的add函数的名称为“?add`@YAHHH`Z”，而不是函数名add，这是C++编译器的命名改编机制。 修改原来的代码：

{% highlight cpp %}
AddAddr Add = (AddAddr)GetProcAddress(hmod, "?add@@YAHHH@Z");
{% endhighlight %}

这时运行就成功了。但如果按这样去动态加载DLL，那每次获取函数地址都要使用dumpbin命令去获取，则会很麻烦。

那怎样可以直接使用add而不是 ?add`@YAHHH`Z这个长长的字符串呢，修改add.dll的add函数，在函数前加上extern "C"，再编译add.dll文件所在的工程，复制新生成的add.dll覆盖DLLTest工程目录下的add.dll，原来的代码获取函数地址时使用add，结果运行就成功了。

而再使用dumpbin -exports add.dll命令，显示add.dll的中的add函数的名称变成了add.
