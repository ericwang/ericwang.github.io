---
layout: post
title: C++内存泄漏检测拾遗
description: ""
category: program
analytics: true
tags: [c++, memory, leak, 内存, 泄露, vs]
---

时间:2009-10-07 cnblogs 放牛娃
在MFC开发环境中，当运行退出了，Visual Studio会在输出窗口提示是否有内存泄漏。也可以借助MFC类CMemoryState动态地检测并输出内存泄漏信息。
在非MFC框架中，需要借助CRT函数实现这些功能。
1. 调用\_CrtDumpMemoryLeaks（）函数会在输出窗口中输出当前的内存泄漏。若在程序开始处加上：\_CrtSetDbgFlag（ \_CRTDBG\_ALLOC\_MEM\_DF | \_CRTDBG\_LEAK\_CHECK\_DF ）；
语句，CRT会在程序的每个出口处自动调用\_CrtDumpMemoryLeaks函数，因此程序终止时会在输出窗口显示所有的内存泄漏。
2.利用\_CrtMemState结构定点监测内存泄漏，例：

{% highlight cpp %}
//定义3个内存状态
_CrtMemState s1,s2,s3;
//记录开始的内存状态
_CrtMemCheckpoint( &s1 );
int* p = new int;
//记录结束时的内存状态
_CrtMemCheckpoint( &s2 );
//比较2个内存状态,并将差异保存到s3中
if( _CrtMemDifference( &s3, &s1, &s2 ) )
{
	//输出内存泄漏信息
	_CrtMemDumpAllObjectsSince( &s3 );
}
{% endhighlight %}

3. 重定向输出信息。内存泄漏提示默认是输出在输出窗口中，可以通过\_CrtSetReportMode重定向其输出位置。例（重定向输出到指定文件）：

{% highlight cpp %}
CAtlFile hFile;
hFile.Create( _T("D:\\report.txt"), GENERIC_WRITE, FILE_SHARE_WRITE, CREATE_ALWAYS );
_CrtSetReportMode( _CRT_WARN, _CRTDBG_MODE_FILE );
_CrtSetReportFile( _CRT_WARN, hFile );
{% endhighlight %}

此外还可以重定向为窗体提示（带有"终止"、"继续"、"忽略"按钮的窗体），断言就是输出为此窗体。还可以通过\_CrtSetReportHook函数在输出到指定目的地之前拦截消息。如：

{% highlight cpp %}
_CrtSetReportHook(MyReportingFunction);
//MyReportingFunction的定义如下：
int MyReportingFunction( int nReportType, char *szMsg, int *pRetVal )
{
	*pRetVal = 0;
	if( nReportType == _CRT_WARN )
	{
		AtlMessageBox( NULL, _U_STRINGorID( CA2T(szMsg)));
	}
	return 0;
}
{% endhighlight %}

http://www.bianceng.cn/Programming/vc/200910/11576.htm
