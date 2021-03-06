---
layout: post
comments: true
title:  " 山石网科面试"
date:   2019-03-15 22:17:20 +0800
tags: 面试 山石网科 原创
lang: zh
---

<!--引用数学表达式js脚本-->
{% include mathjax_support.html %}

`原创文章`

上午面试山石网科。

面试前先给了一套卷子做，先问我更熟悉C++还是Java呀，当然是C++。

本来看这公司名字，来之前把计算机网络给复习了一下，结果那卷子上的题好多操作系统的，计算机网络的也有，少。可惜操作系统没复习。（以后应该先认真仔细看看公司简介的

印象比较深的一题是C++的，给定i为6，求代码的输出结果，大概长这样：

{% highlight cpp %}
void f(int i){
    if(++i<=6||i++<7||i++<8||++i<9){
        //...
    }
    i++;
    cout<<i;
}
{% endhighlight %}

当然是11了。

如果长这样呢：

{% highlight cpp %}
void f(int i){
    if(++i<=6||i++<7||i++<=8||++i<9){
        //...
    }
    i++;
    cout<<i;
}
{% endhighlight %}

那就变成10了，因为到i++<=8时候判断为真后边就不判断了，因为逻辑运算是或，一个为真就可以了。

卷子上还有两道写代码的，第一题是给定编码方式让写一个解码功能，题目大概是这样：

{% highlight cpp %}
'1'=>'3'
'2'=>'5'
...
{% endhighlight %}

我就写了个这样的。

{% highlight cpp %}
string decode(const string & s) {
    string ans = "";
    for (char x : s) {
        switch (x)
        {
        case '3':ans += '1'; break;
        case '5':ans += '2'; break;
        // ...
        default:
            ans += x;
            break;
        }
    }
    return ans;
}
{% endhighlight %}

感觉没啥问题。

第二个写代码的题目是判断一个链表存储的字符串是不是回文序列，要求时间复杂度为O(n)，空间复杂度为O(1)。

不知道怎么将时间复杂度控制到O(n)，就按照常规思路一个一个判断了，时间复杂度为O(n^2)，就不上代码了。

比较诧异的是最后一题竟然是初等数论的题，之前校招题里边还没见过有这种题呢。不过考虑到这是个安全公司应该挺合理的。

题目说一个监狱管理员管理监狱里的囚犯，给他们吃饭时一个桌子坐三个人会多两个，坐5个人多4个，坐7个人多6个，坐9个人多8个，坐11个人正好，求囚犯最少多少人。

题目抽象成数学公式就是这样子的：

$$
\left\{ 
\begin{array}\\
3a+2=n\\
5b+4=n\\
7c+6=n\\
9d+8=n\\
11e=n
\end{array}
\right.
$$

其中a、b、c、d、e、n均为正整数，求最小的n

先简化条件了当然：

$$
\left\{ 
\begin{array}\\
3(a+1)=n+1\\
5(b+1)=n+1\\
7(c+1)=n+1\\
9(d+1)=n+1\\
11e=n
\end{array}
\right.
$$

那么进一步有：

$$
\left\{ 
\begin{array}\\
3*5*7*9*f=n+1\\
11e=n
\end{array}
\right.
$$

即有：

$$
945*f=11*e+1
$$

很显然了，用欧拉定理：

$$
945=85*11+10\\
11=1*10+1
$$

然后倒着推：

$$
1=11-1*10\\
1=11-1*(945-85*11)\\
1=-1*945+86*11\\
945*(-1)=11*86-1\\
945*1=11*(-86)+1\\
945*11+945*1=945*11+11*(-86)+1\\
945*12=11*859+1
$$

那么答案就是$$11*e=11*859=9449$$人

然后面试，先自我介绍，讲了一下大学几年自己都干了啥，做了啥，学了啥。

完事问了我做的校园网客户端的一些东西，都是自己一行一行敲出来的当然问啥答啥。

接着考了一道题：

求一个集合的所有子集，比如说{1,2}有四个子集：空集、{1}、{2}、{1,2}。

先画了一个树，表示遍历过程：

![AVGr4I.png](https://s2.ax1x.com/2019/03/15/AVGr4I.png)

然后写代码：

{% highlight cpp %}
void loop(int* array, int index, const set<int> &s) {
    set<int> s1 = s;
    set<int> s2 = s;
    s2.insert(array[index]);
    loop(array, index + 1, s1);
    loop(array, index + 1, s2);
}
{% endhighlight %}

然后面试官问：递归出口呢？

（尴尬。太紧张给忘了。

然后加上递归出口：

{% highlight cpp %}
void loop(int* array, int index, const set<int> &s) {
    if(index >= n) return;
    set<int> s1 = s;
    set<int> s2 = s;
    s2.insert(array[index]);
    loop(array, index + 1, s1);
    loop(array, index + 1, s2);
}
{% endhighlight %}

面试官：我要的结果呢？你怎么不输出啊？

{% highlight cpp %}
void loop(int* array, int index, const set<int> &s) {
    if (index >= n) {
        // cout<< ...... 
        return;
    }
    set<int> s1 = s;
    set<int> s2 = s;
    s2.insert(array[index]);
    loop(array, index + 1, s1);
    loop(array, index + 1, s2);
}
{% endhighlight %}

然后就完事了，面试官说我思路很清晰。

其实还能改，当时紧张没想起来。

空间复杂度降低：（不过这样时间复杂度就提升了耶，好像没什么实质性改进

{% highlight cpp %}
void loop(int* array, int index, bool *b) {
    if (index >= n) {
        cout << "result: ";
        for (int i = 0; i < n; i++) {
            if (b[index])cout << array[index];
        }
        cout << endl;
    }
    b[index] = false;
    loop(array, index + 1, b);
    b[index] = true;
    loop(array, index + 1, b);
}
{% endhighlight %}


