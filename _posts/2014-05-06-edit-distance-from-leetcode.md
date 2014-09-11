---
layout: post
tags: c++ dynamic-programming leetcode online-judge algorithm
categories: code

---

[LeetCode上一个题](http://oj.leetcode.com/problems/edit-distance/): 求两个字符串之间的编辑距离，意思是通过单个字母的添加/删除/替换操作，把一个字符串word1变成另一个字符串word2，求最小操作次数。

用DP来解，把问题分解为求word1的某个子串和word2的某个子串之间的距离。维持一个二维数组dist，dist[i][j]保存的是word1[0, i)到word2[0, j)的距离，也就是两个从头开始，长度分别为i和j的子串。之后分三种情况考虑：

1）替换，替换最简单，首先知道了dist[i-1][j-1]的值，也就是word1[0, i-1)到word2[0, j-1)的距离，再看word1[i-1]和word2[j-1]是否相等（前面两个子串不包含i-1和j-1位），如果不相等就+1，表示一次替换操作。

2）添加，通过添加一个字母把word1[0, i)变成word2[0, j)，那之前的子问题一定是把word[0, i)变成了word2[0, j-1)。word1已经用了i个，word2只用了j-1个，添加一个正好j个。因此这种情况下距离d_add=dist[i][j-1]+1.

3）删除，通过删除一个字母把word1[0, i)变成word2[0, j)，删除的字母就是当前的word1[i]，因此子问题就是把word1[0, i-1)变成word2[0, j)，解放在dist[i-1][j]。这种情况下距离d_del=dist[i-1][j]+1.

递推公式：
\\[
Dist(i, j)=min\{Dist(i-1,j-1)+ \sigma(i-1,j-1), Dist(i-1,j)+1, Dist(i,j-1)+1\}
\\]
其中\\(\sigma\\)判断第i,j位置字母是否相等，是返回0否则返回1。
三种情况分别查看当前元素左上方，上方和左方，计算之后取最小值就可以了。第一行和第一列比较特别，第一行dist[0][j]表示把一个空串变成一个长度j的串，需要j次添加。第一列dist[i][0]表示把一个长i的串变成空串，需要i次删除。最后结果就是矩阵右下角就是最终结果。

然后发现，二维数组是不必要的，每次更新只和上一行有关，所以开出两行来就足够了。每次利用上一行的信息更新下一行，然后swap。由于全部做完之后有一次swap，所以最终结果存放在row1的最后一个元素。

{% highlight cpp %}
int minDistance(string word1, string word2) { 
    vector<int> row1(word2.size()+1), row2(word2.size()+1); 
    for (int i = 0; i <= word2.size(); ++i) { 
        row1[i] = i; 
    } 
    for (int i = 1; i <= word1.size(); ++i) { 
        row2[0] = i; 
        for (int j = 1; j <= word2.size(); ++j) { 
            int d_rep = row1[j-1] + (word1[i-1] != word2[j-1] ? 1 : 0); 
            int d_add = row2[j-1] + 1; 
            int d_del = row1[j] + 1; 
            row2[j] = min(min(d_rep, d_add), d_del); 
        } 
        swap(row1, row2); 
    } 
    return row1.back();
}
{% endhighlight %}