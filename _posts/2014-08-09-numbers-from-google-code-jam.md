---
layout: post
---

A very funny problem from [Google Code Jam](https://code.google.com/codejam/contest/32016/dashboard#s=p2).

Briefly:

> Find the last three digits before the decimal point for the number $$ (3 + \sqrt{5})^n $$.

This remind me of the closed form formula of [Fibonacci series][fib], if you don't have patients, jump to [Proof](#induction-proof).

###Start from Fibonacci

The closed form formula:

$$
F_n= {1\over \sqrt5}({1+\sqrt5 \over 2})^n-
{1\over \sqrt5}({1-\sqrt5 \over 2})^n
$$

It is interesting to have an irrational expression derived to an integral result. The trick is that there are two parts aside the subtraction, both of which have irrational components, but cancel with each other. 

Moreover, the two parts are not equally important, the first one is the main part, while the second one is always less than 1 because $$ 1-\sqrt5 \over 2 $$ is. For the nth Fibonacci number, the first part already have a very good guess. The second one is a residual to pull the values from irrational to an integer, the accurate one.

Inspired by this, I claim that

$$
A_n = (a+b\sqrt c)^n + (a-b\sqrt c)^n
$$

must always be an integer, with a,b,c integers. It's easy to prove that via induction, or you can think of this as a complex number with a $$\sqrt5$$ took the place of $$i$$.

###Induction Proof
If we represent:

$$
B_n = (a+b\sqrt c)^n=a_n+b_n\sqrt c,\\
C_n = (a-b\sqrt c)^n=a_n-b_n\sqrt c,\\
a_1 = a, b_1 = b
$$

then we have:

$$
A_n = B_n + C_n = 2a_n
$$

is an integer. 
So we are to prove that the $$a_n,b_n$$ representation exists.
This is obvious when $$n=1$$, assume this holds for $$n-1$$, then

$$
B_n = B_ {n-1} B_1 = 
(a_ {n-1}+b_ {n-1} \sqrt c)(a_1+b_1 \sqrt c) \\
=(a_1 a_ {n-1} + b_1 b_ {n-1}c) +
 (a_1 b_ {n-1} + b_1 a_ {n-1})\sqrt c \\
C_n = C_ {n-1} C_1 = 
(a_ {n-1}-b_ {n-1} \sqrt c)(a_1-b_1 \sqrt c) \\
=(a_1 a_ {n-1} + b_1 b_ {n-1}c) -
 (a_1 b_ {n-1} + b_1 a_ {n-1})\sqrt c
$$

also holds for $$n$$. 

And we have the transformation:

$$
a_n = a_1 a_ {n-1} + b_1 b_ {n-1}c \\
b_n = a_1 b_ {n-1} + b_1 a_ {n-1}
$$

Now it's easy to solve $$A_n$$ through $$a_n, b_n$$.

Put in $$ a = 3, b = 1$$, we know that the second part  $$(3-\sqrt5)^n$$ is always less than 1, it makes up to 1 with a minor part from $$(3+\sqrt5)^n$$. Thus the integral part of $$(3+\sqrt5)^n$$ is exactly 1 less than $$A_n$$.

###Code
The code is simple compare to the deducing:

~~~ cpp
#include <stdio.h>

int main() {
	int T; scanf("%d", &T);
	for (int t = 1; t <= T; ++t) {
		int n; scanf("%d", &n);
		int a = 1, b = 0;
		for (unsigned bit_flat = 1 << 31; bit_flat > 0; bit_flat >>= 1) {
			// A_n = (A_{n/2})^2
			int a2 = a*a + 5*b*b;
			int b2 = 2*a*b;
			a = a2 % 1000, b = b2 % 1000;

			if (bit_flat & n) {
				// A_n = A_{n-1} * (3 + \sqrt 5)
				int a2 = 3*a + 5*b;
				int b2 = a + 3*b;
				a = a2 % 1000, b = b2 % 1000;
			}
		}

		printf("Case #%d: %03d\n", t, (2*a-1)%1000);
	}
	return 0;
}
~~~

[fib]: http://en.wikipedia.org/wiki/Fibonacci_number#Closed-form_expression "Fibonacci Series"