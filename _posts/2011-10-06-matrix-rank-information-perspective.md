---
layout: post
tags: linear-algebra
categories: math
title: 从信息的角度看矩阵的秩
---
以前曾经看过MIT的线性代数公开课，其中有一节是用线性代数的方法解图的问题，印象比较深。随便一个图，然后每个节点对应矩阵一行，每条边对应矩阵一列。一条边从一点出就在对应位置写上-1，入的话写+1，其他地方全0。再加上一个诡异的电学里的定律，大概是说每个点流入流出的电流相等。这样竟然证明了欧拉的一个关于图的边和点和环路公式。点+环=边+1。

当时第一感觉就是好奢侈，把图用这种方式表达为一个矩阵，稀稀拉拉到处都是0（每行就两个元素，一个1，一个-1）。从那时起就隐约觉得矩阵的秩和他包含的信息量有很大关系。

信息量有一种衡量方法，就是看获得这个信息之后能够排除多少可能性。假如把一个矩阵看做一个线性方程组，方程组的解越多，说明不确定性越大，矩阵的信息量包含的越少。因此满秩矩阵包含的信息量最大。

几个矩阵秩的不等式更加体现了这一点。

最简单的一个如r(AB)<=r(A)，直观上看A乘B就是把A的列拿来组合，如何组合视B中的列而定，如果B不给力，就可能漏掉A中的列，秩就减少了。或者说信息就丢失了，漏掉的那个列怎么也找不回来了，因为原来那个列是什么对乘积都没有影响了。这个很容易理解。

另一个不等式是r(AB)>=r(A)+r(B)-n，n是A的列数，等于B的行数。已知AB相乘会损失信息了，那损失多少呢？看极端的情况，如果A满秩，r(A)=n，r(AB)=r(B)，损失的秩等于n-r(B)。前面说了满秩的矩阵包含了最大的信息量，那也就最容易损失信息。美好的东西总是比较脆弱的。所以对一般的A，损失的秩小于等于n-r(B)，这样就有了r(A)-r(AB)<=n-r(B)。当然这只是看待问题的一个角度，并不是严格的证明。

一个函数的傅里叶展开，在不连续点可能并不等于原函数。傅里叶系数只和积分有关，而原函数在某几个点改变一下值根本影响不到积分结果。换句话说傅里叶展开后可能无法还原原函数。这是不是也可以从信息量的角度看呢？一个函数在定义域上每点取一个值，看成一个向量的话，维数是不可数的，因为实数不可数。但是傅里叶展开后，这些项是可数的。所以就损失了信息，某些点的值也就无法还原了。