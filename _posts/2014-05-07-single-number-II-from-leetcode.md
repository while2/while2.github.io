---
layout: post
tags: code
title: Signel Number II from Leetcode
---

[leetcode一道水题](http://oj.leetcode.com/problems/single-number-ii/): 从一堆数中间找到唯一一个出现次数不是3的数。这一类问题可以这样表述：有x个数构成的长度为n的数组，其中有x-1个数，其出现次数是K的整数倍，剩下一个特殊的数出现次数不是k的整数倍。求这个特殊的数。
直观的方法是开一个数组进行计数，但是空间复杂度就是O(n)。
有两点可以改进，
1. 对于出现次数，我们只关心对K取模的结果。同一个数a出现m次和出现m+k次对我们不产生影响，这样可以消除一些纵向的冗余。
2. 对于大部分的数，我们并不关心数值。在计数的任何一个阶段，如果有两个数a和b，都出现了K的整数倍次，那他们都不会再对未来产生影响。
我们只关心出现次数不是K的正可以把计数放到每一位上，如果一个数出现了n次，这个数中为1的位，在这个数中也出现了n次。对每一位进行计数，等效于对数字进行计数，关键
int当中同一位b，在任何一个数字中出现次数，只要满了3次，就可以归0，对最后结果也不产生影响。
方法就是利用位运算和模，把计数变成几个很小的int之间的位操作。

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