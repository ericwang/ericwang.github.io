---
layout: post
title: ostringstream函数的使用
description: "当存在未知数据大小的时候,可以使用 ostringstream 来代替 sprintf , 避免总是申请大量的缓冲区.用法可以参照下面转载的文章."
category: program
analytics: true
tags: [c++, sstream]
---

当存在未知数据大小的时候,可以使用 ostringstream 来代替 sprintf , 避免总是申请大量的缓冲区.用法可以参照下面转载的文章.

另外解决今天遇到的一个问题,如果要重复使用一个ostringstream对象,并且需要在下次使用前清空缓冲区,则可以使用str()函数重设置缓冲区. 如:

{% highlight cpp %}
ostringstream  osSql;
//first time
osSql<<"SELECT  COUNT(*) FROM t_XXXX";
...
clsConnection.Query( osSql );
....
//second time
osSql.str("");//重新使用一个空的缓冲区
osSql<<"INSERT INTO **********"<<  strBigText  ;
.......
{% endhighlight %}

以下转载 一篇关于 ostringstream 的用法 的文章

在写程序的时候,我们往往需要对字符串进行格式化, 比如写SQL语句的时候. 在ANSI C 中可以sprintf(), 在MFC中可以用CString::Format()对字符串进行格式化. 但前者无法实现字符串的动态增加,比如你定义的字符缓存为100个字节,如果你格式化以后的内容超出了100个字节,那边后面的内容就无法看见. 所以一般来讲都为定义一个足够的字符缓冲,但这样的效率是很差的. 后者虽然可以解决这个问题,但有一点, 他和前者一样,存在着安全隐患. 比如下面的代码

{% highlight cpp %}
char buf[100]={0};
const char* str = "string";
sprintf(buf,"this is a string : %s , %s", str);
{% endhighlight %}

或者

{% highlight cpp %}
CString s;
s.Format("this is a string: %s, %s",str);
{% endhighlight %}

前面那个%s对应第一个参数str,那么第二个%s呢, 指针指向何方? 如果你幸运的话,什么事也不会发生或者你仅仅获得一个你觉得莫名奇妙的字符串值. 但更严重的情况呢, 不用说----访问越界![](你的程序就等着迎接臭名远扬的Windows红框吧)

如果你使用STL的sstream,那么一切将归于寂然:

{% highlight cpp %}
#include<sstream>
...
ostringstream str;
str<<"this is a str" << "string"<<"and this is a interger<<3<<endl;
{% endhighlight %}

你不必考虑字符串的增长问题, 也不用在写程序时去一一匹配你的format函数的format参数是否一一匹配(如果你的参数很多,那么这项检查工作将是一个让你头痛万分的工作).

一切都是如此的简单.

如果你想获取格式化好的字符串, 通过ostringstream::str()函数就可以返回一个string对象, 调用string::c\_str() 或string::data()函数就可以获得一个指向字符缓冲的char\*变量.

另外, 我坚决建议用string替代CString . 因为CString在设计时考虑到效率问题, 内存是重复利用的, 即:一个进程中所有的CString对象都使用同一个内存缓冲区,且这个内存区运行时不会释放,直到进程结束为止. 如果缓冲不够,他会无限制往上增长. 如果你设计的系统是一个7\*24运行的系统且字符串分析工作量很大,程序中产生了大量的CString对象的话, 你可能会发现你的内存会不断的上涨. string同样有一个很优秀且高效率的内存管理机制,但他不会死守着你的内存不放,这一点你看看STL的源码就知道了.

如果你觉定采用STL,那么建议你使用VC7.0作为开发环境或者GCC , 因为这两款开发工具对STL支持是还算不错.尤其是GCC. 如果你坚持用VC6的话,那么你最好打上SP5补丁包, VC6对STL的支持不够理想.
