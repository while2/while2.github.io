---
layout: post
tags: c++ 
categories: code
abstract: Programming is a process of decision making. There are many ways to get things work, but most of them are bad ways. The code of coding helps to define good from bad. This is a draft of ZJUCVG C++ style guide.

---

Programming is a process of decision making. There are many ways to get things work, but most of them are bad ways. The code of coding helps to define good from bad.

The following rules are my own understanding for ZJUCVG daily research coding, in which case Visual Studio on Windows is the main environment. Some of them may not apply to other circumstances. But remember [The Principle](#the-principle) are the same.

I borrow some of the rules from [Google C++ Style Guide](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html), but I can agree no more than 2/3 of them.
More over, rather than make forcible rules, I would like to trust people. Rules should be a reference for peer-review. Just like the Jury System in a Common Law country.

## The Principle

### Consistency comes first.
Consistency provide a stable environment where your knowledge about the code style is reliable.

There are many arbitrary rules, especially for naming, indentation, there's no reason prefer one to another. But no matter which way you choose, choose it all the time.

### Don't Repeat Yourself
Known as the **DRY** principle. Repeat is the hazard of inconsistency. What if you changed one of your copies and forget another?

Exception: When the common code need to be customized for different users, repeat is acceptable. In this case the change of one is meant not to affect (inconsistency to) the other.

> It is also possible to share common code, and abstract the differences with arguments, but this works only for sophisticated code. In research project, things are changed too quickly, copy &paste can be a better solution.

### As local as possible
One truth about scope: Keep things as local as possible. Mortals made mistakes, there's no good to release bugs to an unnecessarily larger scope.

As you can see in the following rules, more local you are, more freedom you have. On the other hand, the more global code you are dealing with, more careful you should be, in such cases a small change may cause catastrophe. Fortunately global code comes much less than local ones.

$$
\begin{align*}
stdafx &.h		& \text{global} & \\
* &.h				& \text{non-member} & \\
  &				& \text{member}  & \\
* &.cpp			& \text{static} & \\
  &				& \text{local} & \\
\end{align*}
$$

#### Global variables
Don't use external variables. You can define configurations in a singleton class for all modules to access. For global constants, you can define it in the precompiled header.

## Naming & Formatting

Quote from Google C++ Style Guide:

> The most important consistency rules are those that govern naming. The style of a name immediately informs us what sort of thing the named entity is: a type, a variable, a function, a constant, a macro, etc., without requiring us to search for the declaration of that entity. The pattern-matching engine in our brain relies a great deal on these naming rules.

Like in an essay, most letters are lower case, reserve capital letters for rare names. (MACRO/Enum >> Function >> variable). Use underscore to split words when necessary.

Avoid abbreviation unless it's necessary. When use abbreviations, comment at the right place, unless it's a commonsense that everybody knows.

### MACRO / Enum
Macros should be all capitals. Underscores can be used to split words.

Enumerators are very close to MACROs, hence follow the same rule.

### Function
Function names start with a capital letter and have a capital letter for each new word. No underscores are needed.

The name should be verb, describing an action.

### Variable
Variable names should be a none in lower case, use underscore to split words. Add a prefix `m_` for class member functions.

Exceptions:
Some words are not comfortable to be translated into lower cases. In this case capital letters are acceptable.

In research papers, there are capital variables such as $$X = RT*x$$, to keep it consistent, it is also acceptable to use variable named `RT`. In a certain context, the name `RT` is as expressive as the name `rotation_and_translate` and is much better than `camera_pose`.

### auto
Use auto only for local variables, when the type deducing is straightforward. `for (auto it = my_vector.begin(); ...)` is encouraged.

### Comments
Code is the best comment. Don't use comment unless it's necessary. Comment breaks the DRY principle. You repeat your idea twice, once in the code and once in the comment. When you change your code, can you remember to update the comment?

Put your comment at the right place, according the scope of the code you are commenting about.

1. For local expressions, comment close to the considering code.
2. To describe a function, comment before the declaration(if you are to explain the interface) or definition(to explain the implementation).
3. To describe a bunch of functions, or a class, comment at the beginning of the .h / .cpp file for interface / implementation.

Use `//` at the end of lines, `/*  */` for multiple lines.

## Header Files

Differences between headers and source files.

$$
\begin{align*}
& *.h & .cpp, *.cc \\
& \text{What can I do?} & \text{How to do it?} \\
& \text{Interface} & \text{Implementation} \\
& \text{Precompile (to text).} & \text{Compile (to Binary).} \\
& \text{Affect all files including it.} & \text{Local.} \\
& \text{Be more careful.} & \\
\end{align*}
$$

### #pragma once
To prevent multiple inclusion.

<table width = "100%">
<tbody>
<tr>
<td>
{% highlight cpp %}
#ifndef PROJ1_DIR1_DIR2_H
#define PROJ1_DIR1_DIR2_H
...
#endif // PROJ1_DIR1_DIR2_H
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
#pragma once


...
{% endhighlight %}
</td>
</tr>
<tr>
<td>
Compatible with old compilers.<br>
More headers with the same content.<br>
Name your MACRO carefully (systematically).<br>
</td>
<td>
More efficient.<br>
Can be generated by Visual Studio.
</td>
</tr>
</tbody>
</table>

### Self-contained
Make sure your header files are completed and 'compilable'. It should pass the compile when included by an empty cpp(precompiled headers are allowed).

<table width = "100%">
<tbody>
<tr><td> A.h </td><td> B.h </td> <td> C.h </td> </tr>
<tr>
<td>
{% highlight cpp %}
#pragma once

class A {

    ...


};
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
#pragma once

#include "A.h"
class B {
    ...
    A a;
    ...
};
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
#pragma once

#include "A.h"	
#include "B.h"
class C {
    A a;
    B b;
};
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

Also include A.h in C.h because it depends on A. Even if B.h has already included A.h. Otherwise B.h does not satisfy the 'compilable' rule.

One may remove B from C later, or A from B. In the latter case, the change of B causes compile error of C, that's terrible in large projects.

### Decoupling
Remove unnecessary includes from .h files. (As local as possible).

#### Forward declaration

{% highlight cpp %}
class D;
class C {
    ...
    D *d;
    ...
};
{% endhighlight %}

> A pointer is merely a number unless you need to interpret it. Until then, you don't need to know what the class looks like.

#### Hide implementations

<table width = "100%">
<tbody>
<tr><td> Solver.h </td><td> Solver.cpp </td> </tr>
<tr>
<td>
{% highlight cpp %}
#pragma once
#include "Input.h"
#include "Output.h"
class Solver {
    ...
    Output solve(Input input);
    ...
};
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
#include "Solver.h"

#include <Eigen>
#include <GCO>
Output Solver::solve(Input input)
{
    ...
}
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

### using namespace
Never import names with `using namespace` in header files. It causes name pollution. However, it's acceptable to import namespace std in precompiled files since it's a commonsense known by the entire project. 

When user defined names conflict with libs, use a namespace wrapper. If conflict happens between 3rd party libs, congratulations! As far as I know, there's no good solutions for C++, you can better encapsulate one of them with your own interfaces.

## Functions

### Keep functions short

If you are writing a long function, you may want to split it. One function means just 'one function'.

If your function takes too much arguments, maybe it's a sign. Use a struct to pack all them all, or create a class with multiple `Init()` methods and a `Run()`.

### Operator Overloading

Never use it. Use functions instead. Unless you have higher aesthetic ambitions and are not persuaded by the following:

1. +,-,*,/ makes implication that the operator is efficient and bug free, can you guarantee that?
2. Undistinguishable from build-in operators, hard to find.

> Remember you have human readers other than compilers.

### Lambda expression
Lambda expression is a new feature of C++11, supported by Visual Studio since VC11(VS 2010). As an anonymous function, lambda expression is more local than functions.

## Control flow

### Code block
There two styles to write an if (or for/while) block.
<table>
<tbody>
<tr>
<td>
{% highlight cpp %}
if (a == b) {

...
}
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
if (a == b) 
{
...
}
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

Use the second one, unless you are working on a project that following the first style (consistency comes first). Here is the reason:
1. The braces define a block of code, it can be used without condition check or function definitions. In this case, it is impossible to follow the first style. To be consistent, always use the second one.
2. The second style is symmetric and makes it easier to check the brace pairing.
3. And a bonus is that when you are debugging, you can simply comment out the if line to skip the condition check.

### goto
`goto` is acceptable but should not be overused.
Since keyword `break` can only jump out from the inner iteration, `goto` is useful in such cases. An alternative way is to set and check flags. `goto` can be much more clear than flags.

<table width = "100%">
<tbody>
<tr><td> flag </td><td> goto </td> </tr>
<tr>
<td>
{% highlight cpp %}
bool finished = false;
for (int y = 0; y < rows && !finished; ++y)
{
    for (int x = 0; x < cols && !finished; ++x)
    {
        if (...)
            finished = true;
    }
}

    ...
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
for (int y = 0; y < rows; ++y)
{
    for (int x = 0; x < cols; ++x)
    {
        if (...)
            goto finished;
    }
}

finished:
    ...
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

When you get out of a code block, the local variables will be destructed automatically, even if you jumped out with `goto`.

There's no indent before the label.

## Classes

### RAII

Resource acquisition is initialization. (Uninitialized << Initialized << Default initialization).

All for one and one for all. **Copy Constructor**, **Assignment operator** and **Destructor**, use them all or none, none is better.

A default constructor will initialize all members with their default constructors. Think a way to take the advantage.

As long as the **resource** means memory, the Three can be avoid. Think a way to use the default generated version. Use shared_ptr and vector. It's not easy to write a robust constructor for all conditions, have you thought about thread safe, no memory exceptions? Well, STL have. Do not reinvent the wheel, life is short after all.

<table width = "100%">
<tbody>
<tr><td> Bad </td><td> Good </td> </tr>
<tr>
<td>
{% highlight cpp %}
...
class A;
class B
{
public:
    B(int size);
    B(const B &b);
    ~B();
    
    B operator=(const B &b);
private:
    A *m_a;
    A *m_as;
};
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
...
class A;
class B
{
public:
    B(int size);
    
    
    
    
private:
    std::shared_ptr<A> m_a;
    std::vector<A> m_as;
};
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
B::B(int size)
    :m_a(new A)
    ,m_as(new A[size])
{
}

B::B(const B &b)
    :m_a(b.ma)
    ,m_as(new A[size])
{
    ...
}

B operator=(const B &b)
{
    if (b == *this) return *this;
    m_a = b.m_a;
    //... what a mess
}

B::~B()
{
    delete m_a;
    delete[] m_as;
}


{% endhighlight %}
</td>
<td>
{% highlight cpp %}
B::B(int size)
    :m_a(new A)
    ,m_as(size)
{
}





// The Three are unnecessary now.
// Less code, less bugs.












...
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

### Use initialization list
<table width = "100%">
<tbody>
<tr>
<td>
{% highlight cpp %}
class A
{
public:
    A(int size);
   
private:
    int m_number;
    vector<int> m_numbers;
};
{% endhighlight %}
</td>

<td>
{% highlight cpp %}
A::A(int size)
    :m_number(0)
    ,m_numbers(size)
{
    
    ...
    
    
}
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

### Avoid Complicated constructors

Put complicated initialization out of the constructor. Initialize the class to a special state, and call a explicit `Init()` method later. Keep your constructor efficient and exception free. In most cases, initialization list is sufficient. 

Do not use polymorphism in constructors before the virtual function table was initialized.

### Use inheritance carefully

Inheritance is a big thing, think before using it. 
Composition(has-a) is often more appropriate than inheritance(is-a).

Make the dependency as local as possible.

<table width = "100%">
<tbody>
<tr>
<td> Local instance </td>
<td> A member of pointer </td>
<td> A member of instance </td>
<td> Inheritance </td>
</tr>
<tr>
<td>
{% highlight cpp %}
void B::method1()
{
    A a;
    ...
}
void B::method2(A &a)
{
    ...
}
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
class A;

class B
{
    ...
    
private:
    A *m_pa;
};
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
#include "A.h"

class B
{
    ...
    
private:
    A m_a;
};
{% endhighlight %}
</td>
<td>
{% highlight cpp %}
#include "A.h"

class B : public A
{

    ...


};
{% endhighlight %}
</td>
</tr>
<tr>
<td> Include A.h in B.cpp </td>
<td> Declare A in B.h <br>
	Include A.h in B.cpp </td>
<td> Include A.h in B.h </td>
<td> Include A.h in B.h <br>
Unless <br>
B::someMethodOfA() <br>
is needed
</td>
</tr>
</tbody>
</table>

> Polymorphism is an overestimated feature in C++.

### struct or class
<table width = "100%">
<tbody>
<tr>
<td>
struct
</td>
<td>
class
</td>
</tr>

<tr>

<td>
{% highlight cpp %}
struct A
{


    int member1;
    
    
};
{% endhighlight %}
</td>

<td>
{% highlight cpp %}
class A
{
public:
    int GetMember1() const;
    
private:
    int m_member1;
};
{% endhighlight %}
</td>

</tr>

<tr>
<td>
For pure data structure. <br>
No methods, inheritance. <br>
Compatible with C style.
</td>

<td>
When in class, do as classes do. <br>
Declare members as private. <br>
Use accessor if necessary. <br>
Add a prefix to the member name.

</td>

</tr>
</tbody>
</table>

### Explicit Constructors

A constructor with a single parameter can be used as a convertor. If you don't mean it, forbid it.
Use keyword `explicit` to prevent implicit conversions. 

{% highlight cpp %}
class A
{
public:
    explicit A(int size);
};

void foo(A a);
foo(10); // Compile Error
{% endhighlight %}
