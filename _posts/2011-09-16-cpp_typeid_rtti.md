---
layout: post
title: 使用typeid和RTTI C++获取对象运行时类名称【转】
description: ""
category: program
analytics: true
tags: [c++, typeid, RTTI, 运行时, 对象, 名称]
---

观点有一些值得商榷的地方

关于typeid和RTTI的问答
问：在c++里怎么能知道一个变量的具体类型，如：c\#里的typeof.还有我怎么知道一个变量的类型是某个类型的子类，也就是实现关键字IS

答：
1。运行时获知变量类型名称，可以使用 typeid(变量).name，需要注意不是所有编译器都输出"int"、"float"等之类的名称，对于这类的编译器可以这样使用：float f = 1.1f; if( typeid(f)  typeid(0.0f) ) ……
2。对于多态类实例，想得到实际的类名称，需要使用到RTTI，这需要在编译的时候加上参数"/GR"。
3。对于普通变量，既然是自己写的，那当然也就应该知道它的类型，其实用不着运行时获知；对于多态类实例，既然需要运行时获知实际类型，那么就说明这里不具有多态性，既然没有多态性就不应该抽象它，这属于设计错误，总之，我认为RTTI是多余的。
4。对于多态类实例，使用 typeid(value)  typeid(value)来判断，不如使用 dynamic\_cast 来判断，它们的原理是一样的。

示例代码：

{% highlight cpp %}
#include 
using namespace std;
int main( void )
{
// sample 1
    cout << typeid(1.1f).name() << endl;
// sample 2
    class Base1
    {
    };
    class Derive1 : public Base1
    {
    };
    Derive1 d1;
    Base1& b1 = d1;
    cout << typeid(b1).name() << endl; // 输出"class Base1",因为Derive1和Base1之间没有多态性
// sample 3, 编译时需要加参数 /GR
    class Base2
    {
        virtual void fun( void ) {}
    };
    class Derive2 : public Base2
    {
    };
    Derive2 d2;
    Base2& b2 = d2;
    cout << typeid(b2).name() << endl; // 输出"class Derive2",因为Derive1和Base1之间有了多态性
// sample 4
    class Derive22 : public Base2
    {
    };
    // 指针强制转化失败后可以比较指针是否为零，而引用却没办法，所以引用制转化失败后抛出异常
    Derive2* pb1 = dynamic_cast(&b2);
    cout << boolalpha << (0!=pb1) << endl; // 输出"true",因为b2本身就确实是Derive2
    Derive22* pb2 = dynamic_cast(&b2);
    cout << boolalpha << (0!=pb2) << endl; // 输出"true",因为b2本身不是Derive2
    try {
        Derive2& rb1 = dynamic_cast(b2);
        cout << "true" << endl;
    } catch( bad_cast )
    {
        cout << "false" << endl;
    }
    try {
        Derive22& rb2 = dynamic_cast(b2);
        cout << "true" << endl;
    } catch( ... ) // 应该是 bad_cast,但不知道为什么在VC++6.0中却不行
    {
        cout << "false" << endl;
    }
    return 0;
}
{% endhighlight %}

/////////////////////////////////////////////

MFC六大关键技术之运行时类型识别
运行时类型识别（RTTI）即是程序执行过程中知道某个对象属于某个类，我们平时用C++编程接触的RTTI一般是编译器的RTTI

运行时类型识别（RTTI）即是程序执行过程中知道某个对象属于某个类，我们平时用C+*编程接触的RTTI一般是编译器的RTTI，即是在新版本的VC编译器里面选用“使能RTTI”，然后载入typeinfo.h文件，就可以使用一个叫typeid（）的运算子，它的地位与在C*+编程中的sizeof()运算子类似的地方（包含一个头文件，然后就有一个熟悉好用的函数）。typdid()关键的地方是可以接受两个类型的参数：一个是类名称，一个是对象指针。所以我们判别一个对象是否属于某个类就可以象下面那样：

if （typeid (ClassName)== typeid（\*ObjectName））{
((ClassName\*)ObjectName)-&gt;Fun();
}

　　象上面所说的那样，一个typeid（）运算子就可以轻松地识别一个对象是否属于某一个类，但MFC并不是用typeid（）的运算子来进行动态类型识别，而是用一大堆令人费解的宏。很多学员在这里很疑惑，好象MFC在大部分地方都是故作神秘。使们大家编程时很迷惘，只知道在这里加入一组宏，又在那儿加入一个映射，而不知道我们为什么要加入这些东东。

　　其实，早期的MFC并没有typeid（）运算子，所以只能沿用一个老办法。我们甚至可以想象一下，如果MFC早期就有template（模板）的概念，可能更容易解决RTTI问题。

　　所以，我们要回到“古老”的年代，想象一下，要完成RTTI要做些什么事情。就好像我们在一个新型（新型到我们还不认识）电器公司里面，我们要识别哪个是电饭锅，哪个是电磁炉等等，我们要查看登记的各电器一系列的信息，我们才可以比较、鉴别，那个东西是什么！
要登记一系列的消息并不是一件简单的事情，大家可能首先想到用数组登记对象。但如果用数组，我们要定义多大的数组才好呢，大了浪费空间，小了更加不行。所以我们要用另一种数据结构——链表。因为链表理论上可大可小，可以无限扩展。

　　链表是一种常用的数据结构，简单地说，它是在一个对象里面保存了指向下一个同类型对象的指针。我们大体可以这样设计我们的类：

struct CRuntimeClass
{
……类的名称等一切信息……
CRuntimeClass \* m\_pNextClass;//指向链表中下一CRuntimeClass对象的指针
};

　　链表还应该有一个表头和一个表尾，这样我们在查链表中各对象元素的信息的时候才知道从哪里查起，到哪儿结束。我们还要注明本身是由哪能个类派生。所以我们的链表类要这样设计：

struct CRuntimeClass
{
……类的名称等一切信息……
CRuntimeClass \* m\_pBaseClass;//指向所属的基类。
CRuntimeClass \* m\_pNextClass;//定义表尾的时候只要定义此指针为空就可以 了。
static CRuntimeClass\* pFirstClass;//这里表头指针属于静态变量，因为我们知道static变量在内存中只初始化一次，就可以为各对象所用！保证了各对象只有一个表头。
};

　　有了CRuntimeClass结构后，我们就可以定义链表了：

static CRuntimeClass classCObject={NULL,NULL};//这里定义了一个CRuntimeClass对象，

　　因为classCObject无基类，所以m\_pBaseClass为NULL。因为目前只有一个元素（即目前没有下一元素），所以m\_pNextClass为NULL（表尾）。

　　至于pFirstClass（表头），大家可能有点想不通，它到什么地方去了。因为我们这里并不想把classCObject作为链表表头，我们还要在前面插入很多的CRuntimeClass对象，并且因为pFirstClass为static指针，即是说它不是属于某个对象，所以我们在用它之前要先初始化：

CRuntimeClass\* CRuntimeClass：：pFirstClass=NULL;

　　现在我们可以在前面插入一个CRuntimeClass对象，插入之前我得重要申明一下：如果单纯为了运行时类型识别，我们未必用到m\_pNextClass指针（更多是在运行时创建时用），我们关心的是类本身和它的基类。这样，查找一个对象是否属于一个类时，主要关心的是类本身及它的基类：

CRuntimeClass classCCmdTarget={ &classCObject, NULL};
CRuntimeClass classCWnd={ &classCCmdTarget ,NULL };
CRuntimeClass classCView={ &classCWnd , NULL };

　　好了，上面只是仅仅为一个指针m\_pBaseClass赋值（MFC中真正CRuntimeClass有多个成员变量和方法），就连接成了链表。假设我们现在已全部构造完成自己需要的CRuntimeClass对象，那么，这时候应该定义表头。即要用pFirstClass指针指向我们最后构造的CRuntimeClass对象——classCView。

CRuntimeClass：：pFirstClass=&classCView;

　　现在链表有了，表头表尾都完善了，问题又出现了，我们应该怎样访问每一个CRuntimeClass对象？要判断一个对象属于某类，我们要从表头开始，一直向表尾查找到表尾，然后才能比较得出结果吗。肯定不是这样！

　　大家可以这样想一下，我们构造这个链表的目的，就是构造完之后，能够按主观地拿一个CRuntimeClass对象和链表中的元素作比较，看看其中一个对象中否属于你指定的类。这样，我们需要有一个函数，一个能返回自身类型名的函数GetRuntimeClass()。

　　上面简单地说一下链表的过程，但单纯有这个链表是没有任何意义。回到MFC中来，我们要实现的是在每个需要有RTTI能力的类中构造一个CRuntimeClass对象，比较一个类是否属于某个对象的时候，实际上只是比较CRuntimeClass对象。

　　如何在各个类之中插入CRuntimeClass对象，并且指定CRuntimeClass对象的内容及CRuntimeClass对象的链接，这里起码有十行的代码才能完成。在每个需要有RTTI能力的类设计中都要重复那十多行代码是一件乏味的事情，也容易出错，所以MFC用了两个宏代替这些工作，即DECLARE\_DYNAMIC(类名)和IMPLEMENT\_DYNAMIC(类名，基类名)。从这两个宏我们可以看出在MFC名类中的CRuntimeClass对象构造连接只有类名及基类名的不同！

　　到此，可能会有朋友问：为什么要用两个宏，用一个宏不可以代换CRuntimeClass对象构造连接吗？个人认为肯定可以，因为宏只是文字代换的游戏而已。但我们在编程之中，头文件与源文件是分开的，我们要在头文件头声明变量及方法，在源文件里实具体实现。即是说我们要在头文件中声明：

public:
static CRuntimeClass classXXX //XXX为类名
virtual CRuntime\* GetRuntimeClass() const;

　　然后在源文件里实现：

CRuntimeClass\* XXX：：classXXX={……}；
CRuntime\* GetRuntimeClass() const;
{ return &XXX:: classXXX;}//这里不能直接返回&classXXX，因为static变量是类拥有而不是对象拥有。

　　我们一眼可以看出MFC中的DECLARE\_DYNAMIC(类名)宏应该这样定义：

\#define DECLARE\_DYNAMIC(class\_name) public: static CRuntimeClass class\#\#class\_name;
virtual CRuntimeClass\* GetRuntimeClass() const;

　　其中\#\#为连接符，可以让我们传入的类名前面加上class，否则跟原类同名，大家会知道产生什么后果。

　　有了上面的DECLARE\_DYNAMIC(类名)宏之后，我们在头文件里写上一句：

DECLARE\_DYNAMIC(XXX)

　　宏展开后就有了我们想要的：

public:
static CRuntimeClass classXXX //XXX为类名
virtual CRuntime\* GetRuntimeClass() const;

　　对于IMPLEMENT\_DYNAMIC(类名，基类名)，看来也不值得在这里代换文字了，大家知道它是知道回事，宏展开后为我们做了什么，再深究真是一点意义都没有！

　　有了此链表之后，就像有了一张存放各类型的网，我们可以轻而易举地RTTI。CObject有一个函数BOOL IsKindOf(const CRuntimeClass\* pClass) const;，被它以下所有派生员继承。

　　此函数实现如下：

BOOL CObject::IsKindOf(const CRuntimeClass\* pClass) const
{
CRuntimeClass\* pClassThis=GetRuntimeClass();//获得自己的CRuntimeClass对象指针。
while(pClassThis!=NULL)
{
if(pClassThis==pClass) return TRUE;
pClassThis=pClassThis-&gt;m\_pBaseClass;//这句最关键，指向自己基类，再回头比较，一直到尽头m\_pBaseClass为NULL结束。
}
return FALSE;
}

　　说到这里，运行时类型识别（RTTI）算是完成了。写这篇文章的时候，我一直重感冒。我曾一度在想，究竟写这东西是为了什么。因为如果我把这些时间用在别的地方（甚至帮别人打打字），应该有数百元的报酬。

　　是什么让“嗜财如命”的我继续写下去？我想，无非是想交几个计算机的朋友而已。计算机是大家公认高科技的东西，但学习它的朋友大多只能默默无闻，外界的朋友也不知道怎么去认识你。程序员更不是“潮流”的东西，更加得不到别人的认可。

　　有一件个人认为很典型的事情，有一天，我跟一个朋友到一个单位里面。里面有一个女打字员。朋友看着她熟练的指法，心悦诚服地说：“她的电脑水平比你的又高了一个很高的层次！”，那个女的打字高手亦自豪地说：“我靠电脑为生，电脑水平肯定比你（指笔者）的好一点！换着是你，如果以电脑为生，我也不敢说好过你！”。虽然我想声明我是计算机专业的，但我知道没有理解，所以我只得客气地点头。

　　选择电脑“潮流”的东西实际是选择了平凡，而选择做程序员就是选择了孤独！幸好我不是一个专门的程序员，但即使如此，我愿意做你们的朋友，因为我爱你们！

http://www.yesky.com/SoftChannel/72342371928702976/20050228/1915827.shtml

///////////////////////////////
首先，很不好意思的说明，我还正在看C** language programming，但还没有看到关于RTTI的章节。另外，我也很少使用C** RTTI的特性。所以对RTTI的理解仅限于自己的摸索和思考。如果不正确，请大家指正。

RTTI特性是C+*语言加入较晚的特性之一。和其他语言（比如JAVA）相比，C的RTTI能力算是非常差的。这与C的设计要求应该有重要的关系：性能。没错，性能的因素使得C的很多地方不能称的上完美，但是也正因为如此，在高级通用语言里面，只有C能和C*+的性能可以相提并论。

1：typeid的研究

在C++中，似乎与RTTI相关的只有一个东西，就是dynamic\_cast，本来我认为typeid是RTTI的一部分，但是我的实验表明，并非如此。typeid的操作是在编译时期就已经决定的了。下面的代码可以证明：

{% highlight cpp %}
#include 
#include
class A
{
};
class B:public A
{
};
int main()
{
   A *pa;
   B b,*pb;
   pb = &b;
   pa = pb;
   std::cout<<"Name1:"
        << (typeid(pa).name())
        <<"\tName2:"
        <<(typeid(pb).name())
        <<:endl;< p="">
   std::cout<<"pa == pb:"<< (typeid(pa) == typeid(pb))<<:endl;
   return 0;
}
{% endhighlight %}

typeid根本不能判别pa实际上是一个B\*。换句话说，typeid是以字面意思去解释类型，不要指望它能认出一个void\*实际上是int\*(这个连人也做不到:P）。实际上实用价值不大。

当然，在某些特殊地方，也是能够有些效用的，比如模板。

{% highlight cpp %}
template 
void test(T t)
{
 if(typeid(t) == typeid(char *))
 {
   // 对char *特殊处理
 }
 //...
}
{% endhighlight %}

如果编译器优化的好的话，并不会产生废代码，因为typeid编译时期就可以决定了。

2：dynamic\_cast

抱歉现在才讲到正题，我对dynamic\_cast第一印象就是，它究竟是怎么实现的呢？经过一些思考，我认为最简单的方案就是将信息保存在vtable里，它会占用一个vtalbe表的项目。实验和书籍也证明了这一点。但是就会有一个问题，没有vtable的类怎么办？内建类型怎么办？其实，没有vtable的类，它不需要多态，它根本就不需要RTTI，内建类型也一样。这就是说，dynamic\_cast只支持有虚函数的类。而且， dynamic\_cast不能进行non\_base\_class **到 class T\*的转换，比如void** --&gt; class T \*，因为它无法去正确获得vtable。

这样，dynamic\_cast的意义和使用方法就很清楚了，它是为了支持多态而存在的。它用于实现从基类到派生类的安全转换。同时它也在绝大多数情况下避免了使用static\_cast－－不安全的类型转换。

3:结论

C** 的RTTI机制虽然简单，或者说简陋，但是它使得静态类型转换变得无用了。这也是C+*的一个不可缺少的机制。在未来，如果C*+能够提供可选的更强的RTTI机制，就像JAVA里的那样，这种语言可以变得更加强大。当然，到时如何提供不损失性能的 RTTI机制，更是一个值得深入研究的话题了。
