---
layout: post
tags: c++ linked-list sort
categories: code
abstract: This post shows how to swap two nodes in a Single-Linked-List and how to apply quick sort for lists. Although it's not recommended.
---

Quick sort is a good algorithm that every notebook take it as an example for Divide and Conquer. Conquer means to recursively call itself, while Division is carried out by partition.

There are various partition methods, I prefer the one from "Introduction to Algorithms". The greatest thing is that this one scan the sequence once from start to the end, which makes it suitable for Single-Linked-Lists. A straight forward partition may scan the sequence from two ends to the middle. But a Single-Linked-List cannot be scanned reversely, not mention that this method takes more steps than the preferred one.

As described in "Introduction to Algorithms", given an array with indices [s, e) and a pivot value p, to partition it to two parts, those less than p and the others, maintain tow cursors `i`, `j` so that:

1. Values in [s, i] is less than p,
2. Values in (i, j] greater or equal to p. 

Initialise i and j as s-1 so the properties hold at the beginning.
Then iterate j from s to e. After ++j, if the value of j is less than p, then the 2nd property broke. This can be fixed by swapping j with the first element in (i, j], then increase i. When j reaches e, the partition is done.

Although this is quite easy for arrays, its not straightforward for a Single-Linked-List, when you want to swap nodes instead of their data.

Given `ListNode` defined as:
{% highlight cpp %}
struct ListNode {
	int val;	
	ListNode *next;
	ListNode(int x) : val(x), next(nullptr) {}
};
{% endhighlight %}

The swap of nodes is surprisingly simple:
{% highlight cpp %}
void Swap(ListNode **p1, ListNode **p2) {
	swap(*p1, *p2);
	swap((*p1)->next, (*p2)->next);
}
{% endhighlight %}
Since the swapping also affects previous nodes (the value of their next pointer should point to a new node after swap), pointer's pointer was used to simplify the process.

Given 2 nodes as:
<pre>
      *p1
       +---+-+
p1---->| a | |---->
       +---+-+
	
      *p2
       +---+-+
p2---->| b | |---->
       +---+-+
</pre>

The status after swap(\*p1, \*p2):
<pre>
            *p2
             +---+--+
p1--+    +-->| a |pa|---->
     \  /    +---+--+
      \/
      /\    *p1
     /  \    +---+--+
p2--+    +-->| b |pb|---->
             +---+--+
</pre>

The status after swap((\*p1)->next, (\*p2)->next):
<pre>
            *p2
             +---+--+
p1--+    +-->| a |pa|--+    +-->
     \  /    +---+--+   \  /
      \/                 \/
      /\    *p1          /\
     /  \    +---+--+   /  \
p2--+    +-->| b |pb|--+    +-->
             +---+--+
</pre>
Note that pa always follows node a, swapping can change it's value but can never change its address.

This simple swapping also works for neighboring nodes.
Given 2 nodes as:
<pre>
       *p1          *p2
       +---+--+     +---+--+
p1---->| a |p2|---->| b |pb|---->
       +---+--+     +---+--+
</pre>
After the first swap there will be a circle:
<pre>
        +-------------+
       /   *p2         \     *p1
       \    +---+--+   /      +---+--+
p1--+   +-->| a |p2|--+   +-->| b |pb|---->
     \      +---+--+     /    +---+--+
      \                 /
       +---------------+
</pre>
But it breaks after the second swap:
<pre>
        +--------------------------------+
       /                      *p1         \
       \    +---+--+           +---+--+   /  *p2
p1--+   +-->| a |p2|--+    +-->| b |pb|--+    +-->
     \      +---+--+   \  /    +---+--+      /
      \                 \/                  /
       \                /\                 /
        +--------------+  +---------------+
</pre>
`(*p1)->next` used to refer to pb, swap it with `(*p2)->next`, which is node a itself, breaks the circle and gives the right result.

The only problem is that the resulting `p1` and `p2` differs in these two cases.

Replace `i`, `j` with `p1` and `p2` in the partition formula, in the general case, p2 points to the swapped greater node, so (p1, p2] holds the 2nd property after the swap. However, in the neighbouring case, p2 moves one step ahead, points to the next node to be considered. To fix this, we set `p2 = &(*p1)->next` in this case.
 
So the final QuickSort comes like this, with the first node selected as pivot:

{% highlight cpp %}
/*
Pick head as the pivot.
Any node n in (head, p1], n < head.
Any node n in (p1, p2), n >= head.
Scan with p2, if p2 < head, swap it with p1->next.
When p2 reaches tail, the partition is down, and
p1 is at the pivot place, so swap it with head.
*/
ListNode *QuickSort(ListNode *head, ListNode *tail) {
	if (head == tail || head->next == tail)
		return head;

	ListNode *pivot = head, **p1 = &head;
	for (ListNode **p2 = &head->next; *p2 != tail; p2 = &(*p2)->next) {
		if ((*p2)->val < pivot->val) {	
			p1 = &(*p1)->next;
			if ((*p1)->next == *p2) {				Swap(p1, p2);
				p2 = &(*p1)->next;
			}
			else {
				Swap(p1, p2);
			}
		}
	}

	Swap(&head, p1);
	pivot->next = QuickSort(pivot->next, tail);
	return QuickSort(head, pivot); // tail recursion
}
{% endhighlight %}

QuickSort is notorious for its degeneration when the sequence has already some sort of order so that the partition cannot divide the problem normally. Since Linked-List can not be randomly accessed efficiently, it is more difficult to design a pivot selecting method.

That's why I would not recommend to quick sort a Linked-List, in any case an array solution sounds better. In case your data is not efficient enough to be swapped, use a pointer array. 