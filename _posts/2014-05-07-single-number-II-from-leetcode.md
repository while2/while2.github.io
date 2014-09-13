---
layout: post
tags: c++ leetcode online-judge
categories: code
title: Signel Number II from Leetcode
---

[leetcode一道水题](http://oj.leetcode.com/problems/single-number-ii/): 从一堆数中间找到唯一一个出现次数不是3的数。这一类问题可以这样表述：有x个数构成的长度为n的数组，其中有x-1个数，其出现次数是K的整数倍，剩下一个特殊的数出现次数不是k的整数倍。求这个特殊的数。
直观的方法是开一个数组进行计数，但是空间复杂度就是O(n)。
有两点可以改进，

1. 对于出现次数，我们只关心对K取模的结果。同一个数a出现m次和出现m+k次对我们不产生影响，这样可以消除一些纵向的冗余。或者说计数只需logK个bit就足够了，当K等于三的时候只需要2位，真棒。
2. 对于大部分的数，我们并不关心具体数值。这使得我们可以把数字的计数转化为对位的计数。反正对于那些出现K次的数，他们中任何一位出现1的次数永远是K的整数倍。由于只有一个outlier，只要把计数后非零的位合并起来就可以得到那个特殊的数了。

利用位运算和模，把计数变成几个很小的int之间的位操作。

{% highlight cpp %}
int singleNumber(int A[], int n) {
	int count[2] = {0, 0};
	for (int i = 0; i < n; ++i) {
		int carry = count[0] & A[i];
		count[0] ^= A[i];
		count[1] |= carry;
		int full = count[1] & count[0];
		count[0] &= ~full;
		count[1] &= ~full;
	}
	return count[0] | count[1];
}
{% endhighlight %}