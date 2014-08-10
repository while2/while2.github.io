---
layout: post
tags: code
title: Semantics "The" by C++ template
---
在程序中经常使用singleton模式，倒不是为了限制说只能有一个实例，而是为了通过类名直接访问一个实例，不必关心这个实例放在哪里。在面向对象的世界里，我们关心的是一些对象的状态而非过程，有很多对象始终存在，但是我们又不得不在某个地方将其定义。这一点在GUI程序中显得尤其明显。一般界面元素会在程序执行的整个生命周期存在于内存中，我并不关心他们之间的什么层次关系。有时候不管我的程序运行在哪，我就是想要更新界面上的一个Menu。

于是用C++写了这么一个伪singleton模板，实现“The语义”。假如定义了类UIMenu1，The<UIMenu1>()可以自动转化为一个UIMenu1的指针，进而做任何想做的事情。使用的时候我不用关心这个UIMenu1的实例放在哪里，实际上它是全局的，但是我知道每次使用The得到的是同一个实例。

~~~ cpp
template<class Cls, int = 0>
class The
{
public:
	Cls* operator->()	{ return get_instance(); }
 
	operator Cls*&()	{ return get_instance(); }
	
private:
	Cls *&get_instance()
	{
		static Cls *s_instance = new Cls;
		return s_instance;
	}
};
~~~

这几乎完全就是一个singleton的实现，只不过没有把构造函数私有化。据我自己的经验，私有化构造函数从来没派上过用场。当我只需要全局唯一的一个实例的时候，我自然会用The来访问。如果自己另外实例化一个对象，很显然和The访问到的不是同一个，这在语义上也说得过去。况且，偶尔可以存在类似The<MyClass>()->swap(MyClass())之类的用法。因此不限制实例化，程序就更灵活一些。

另外，也许有时候可能我们需要访问到的实例不是唯一的，而是唯二，唯三的呢？一种办法是可以写一个Wrapper类把几个对象放进去，再用The<Wrapper>()->GetFirstOne()之类的办法访问。但是这样太麻烦，所以我给The模板加了一个int型参数，默认参数0，之前的用法不受影响。现在The<MyClass, 1>提供了“The MyClass with index 1”的语义，访问到的和The<MyClass>（等同于The<MyClass, 0>）是不同的实例。

有关The和更多his模板库里的内容在：[https://github.com/while2/his](https://github.com/while2/his)