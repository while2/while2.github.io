---
layout: post
tags: scala avl-tree rb-tree 
categories: code

---

The complete code can be found here[^1].

Functional programming languages are usually more about definition than implementation, which makes them perfect to demonstrate data structures.

Scala is such a kind of interesting language, where you can always think in a recursive way, and never bother the detailed processes. 
Once you figure out what exactly you demand, the language synthesis everything for you.

### Binary Search Tree
A simple binary search tree with insert method can be defined as:
{% highlight scala %}
trait Tree {
  def insert(tar: Int): Tree
}

object Empty extends Tree {
  def insert(tar: Int) = NonEmpty(Empty, tar, Empty)
}

case class NonEmpty(left: Tree, key: Int, right: Tree) extends Tree {
  def insert(tar: Int) = {
    if (tar < key)
      NonEmpty(left.insert(tar), key, right).balance
    else if (tar > key)
      NonEmpty(left, key, right.insert(tar)).balance
    else
      this
  }
}
{% endhighlight %}

The recursive definition is exactly the concept of Binary Search Tree.

### AVL-Tree
With the pattern match mechanism, it comes quite straightforward to rotate and balance an AVL-Tree while inserting or deleting.

{% highlight Scala %}
case class NonEmpty(left: Tree, key: Int, right: Tree) extends Tree {
  def rotate_left() = {
    right match {
      case NonEmpty(rl, rk, rr) => NonEmpty(NonEmpty(left, key, rl), rk, rr)
    }
  }
  
  def rotate_right() = {
    left match {
      case NonEmpty(ll, lk, lr) => NonEmpty(ll, lk, NonEmpty(lr, key, right))
    }
  }
}
{% endhighlight %}

The method `rotate_right` says: Hey Scala, when you see something like the following left structure, give me another tree like the right one. Quite different from C++ styles.

<div style="overflow: hidden;">
<pre style="float:left">
          +ll
     +lk--|  
     |    +lr
key--|     
     +right
</pre>
<pre style="float:left">
          +ll
     lk---|
=>        |      +lr
          +key---|
                 +right
</pre>
</div>

The diagram was rotated by 90º, with the left child above the right one, to be consistent with my [display()](https://github.com/while2/bs-tree/blob/master/AVLTree.scala) function.

The balance code is also straightforward, almost exactly translated from the textbook[^2].

{% highlight Scala %}
  def balance() = {
    if (left.height > right.height + 1) {
      left match {
        case NonEmpty(ll, lk, lr) => {
          if (ll.height >= lr.height) rotate_right
          else NonEmpty(left.rotate_left, key, right).rotate_right
        }
      }
    } else if (left.height + 1 < right.height) {
      right match {
        case NonEmpty(rl, rk, rr) => {
          if (rl.height <= rr.height) rotate_left
          else NonEmpty(left, key, right.rotate_right).rotate_left
        }
      }
    } else
      this
  }
{% endhighlight %}
When the left child is more than 2 levels higher than the right one, denote the left child as (ll, lk, lr), the left child's left child, left child's key, left child's right child.
If ll is higher or equal to lr, just right rotate the considering tree. 

Otherwise it's a zigzag situation, left rotate the left child converts it to the previous case, but you'll need to reconstruct the tree with the rotated left child here. Then another right rotate finishes the job.

Now add `balance` after each insertion, when you call `insert` recursively, you know that it returns a balanced tree.

[Here](https://github.com/while2/bs-tree/blob/master/AVLTree.scala) is the complete code.

### Red-Black-Tree
Red-Black-Tree is such a trick that I can never figure out if not read about it from a textbook.

They color the nodes with Red and Black. Only black nodes count for the height of trees. But since they forbid two neighboring red nodes, there can be no more than half of them, so the trees are still balanced.

1. It's either red or black.
2. The root is black.
3. Red nodes have black children.
4. For all nodes, the black height are the same.

After insertion there might be 2 neighboring Red nodes, which breaks the property of Red-Black Trees. But there are only 4 possible structures as the following diagram listed. They are also corresponding to the 4 cases in the code.

<div style="overflow: hidden; width: 80%">
<pre style="float:left">
1.                  +a
             +x(R)--|
             |      +b
      +y(R)--|
      |      +c
z(B)--|
      +d
</pre>
<pre style="float:right">
3.           +a       
      +x(R)--|        
      |      |      +b
      |      +y(R)--|
      |             +c
z(B)--|
      +d
</pre>
</div>
<div style="overflow: hidden; width: 80%">
<pre style="float:left">
      +a
x(B)--|
      |      +b
      +y(R)--|
             |      +c
             +z(R)--| 
2.                  +d
</pre>
<pre style="float:right">
      +a
x(B)--|
      |             +b
      |      +y(R)--|
      |      |      +c
      +z(R)--|
4.           +d
</pre>
</div>

For all of them, the solution is the same:
<pre>
fixed.       
             +a
      +x(B)--|
      |      +b
y(R)--|
      |      +c             
      +z(B)--|
             +d
</pre>

A simple pattern match can fix the broken property:
{% highlight scala %}
  def fix_ins() = {
    this match {
      case NonEmpty(NonEmpty(NonEmpty(a, x, R, b), y, R, c), z, B, d) => NonEmpty(NonEmpty(a, x, B, b), y, R, NonEmpty(c, z, B, d))
      case NonEmpty(a, x, B, NonEmpty(b, y, R, NonEmpty(c, z, R, d))) => NonEmpty(NonEmpty(a, x, B, b), y, R, NonEmpty(c, z, B, d))
      case NonEmpty(NonEmpty(a, x, R, NonEmpty(b, y, R, c)), z, B, d) => NonEmpty(NonEmpty(a, x, B, b), y, R, NonEmpty(c, z, B, d))
      case NonEmpty(a, x, B, NonEmpty(NonEmpty(b, y, R, c), z, R, d)) => NonEmpty(NonEmpty(a, x, B, b), y, R, NonEmpty(c, z, B, d))
      case _ => this
    }
  }
{% endhighlight %}

The deletion of Red-Black-Tree is a little bit complicated, I have not implement it yet.

[^1]: [https://github.com/while2/bs-tree](https://github.com/while2/bs-tree)
[^2]: [Elementary Algorithms](https://sites.google.com/site/algoxy/home)