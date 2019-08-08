---
layout: post
title:  "暗黑的字符串 2017校招真题 网易"
date:   2019-02-17 16:24:13 +0800
tags: 网易 笔试题 校招真题 原创
lang: zh
---

<!--引用数学表达式js脚本-->
{% include mathjax.html %}

`原创文章`

[原题链接](https://www.nowcoder.com/practice/7e7ccd30004347e89490fefeb2190ad2)

`题目描述`
一个只包含'A'、'B'和'C'的字符串，如果存在某一段长度为3的连续子串中恰好'A'、'B'和'C'各有一个，那么这个字符串就是纯净的，否则这个字符串就是暗黑的。例如：
BAACAACCBAAA 连续子串"CBA"中包含了'A','B','C'各一个，所以是纯净的字符串
AABBCCAABB 不存在一个长度为3的连续子串包含'A','B','C',所以是暗黑的字符串
你的任务就是计算出长度为n的字符串(只包含'A'、'B'和'C')，有多少个是暗黑的字符串。

`输入描述`
输入一个整数n，表示字符串长度(1 ≤ n ≤ 30)

`输出描述`
输出一个整数表示有多少个暗黑字符串

`示例输入`
{% highlight cpp %}
2 3
{% endhighlight %}

`示例输出`
{% highlight cpp %}
9 21
{% endhighlight %}

解题
=

`思路`
由题意可知A、B、C是完全对等的，比如我算出来一个解为AABBCC，

那么将A、B互换，得BBAACC也是一个解，

将A、C互换，得CCBBAA也是一个解，

将B、C互换，得AACCBB也是一个解。

因此解的个数一定是3的倍数。

一、穷举 从前往后挨个试
====

{% highlight cpp %}
#include <iostream>

using namespace std;

bool ok(const int *const a, int x) {
    if (x <= 1)return true;
    if (a[x] == a[x - 1])return true;
    if (a[x - 1] == a[x - 2])return true;
    if (a[x] == a[x - 2])return true;
    return false;
}

int countBlack(int *a, int n, int index) {
    if (index == n) {
        return 1;
    }
    int ans = 0;
    a[index] = 1;
    if (ok(a, index)) {
        ans += countBlack(a, n, index + 1);
    }
    a[index] = 2;
    if (ok(a, index)) {
        ans += countBlack(a, n, index + 1);
    }
    a[index] = 3;
    if (ok(a, index)) {
        ans += countBlack(a, n, index + 1);
    }
    return ans;
}

int main() {
    int a[31] = {0};
    int n;
    a[0] = 1;
    while (cin >> n) {
        cout << 3 * countBlack(a, n, 1);
    }
    return 0;
}
{% endhighlight %}

`运行结果`
{% highlight cpp %}
运行超时:您的程序未能在规定时间内运行结束，请检查是否循环有错或算法复杂度过大。
case通过率为70.00%
{% endhighlight %}

意料之中的超时，开始优化。

二、第一次优化
====

注意到代码中每次往后填充新的元素的时候，只需比较前边两个元素即可。

那么前边两个元素有两种情况：

+ AA 或 BB 或 CC 即相同
+ AB 或类似 即不同

对于相同的情况，如AA，那么后边一共有三种情况符合题意：

AA A、 AA B、 AA C

对于他后边接B或C两种情况，两种情况的解的个数一定是相等的，因此有下列代码：

{% highlight cpp %}
#include <iostream>

using namespace std;

bool ok(const int *const a, int x) {
    if (x <= 1)return true;
    if (a[x] == a[x - 1])return true;
    if (a[x - 1] == a[x - 2])return true;
    if (a[x] == a[x - 2])return true;
    return false;
}

int countBlack(int *a, int n, int index) {
    if (index == n) {
        return 1;
    }
    int ans = 0;

    if (index >= 2) {
        if (a[index - 2] == a[index - 1]) {
            a[index] = (a[index - 1] + 1) % 3 + 1;
            ans += 2 * countBlack(a, n, index + 1);
            a[index] = a[index - 1];
            ans += countBlack(a, n, index + 1);
            return ans;
        }
    }

    a[index] = 1;
    if (ok(a, index)) {
        ans += countBlack(a, n, index + 1);
    }
    a[index] = 2;
    if (ok(a, index)) {
        ans += countBlack(a, n, index + 1);
    }
    a[index] = 3;
    if (ok(a, index)) {
        ans += countBlack(a, n, index + 1);
    }
    return ans;
}

int main() {
    int a[31] = {0};
    int n;
    a[0] = 1;
    while (cin >> n) {
        cout << 3 * countBlack(a, n, 1);
    }
    return 0;
}
{% endhighlight %}

`运行结果`

{% highlight cpp %}
运行超时:您的程序未能在规定时间内运行结束，请检查是否循环有错或算法复杂度过大。
case通过率为80.00%
{% endhighlight %}

还是超时，继续优化。

三、最终版本
====

注意到前边两个元素不同的时候还没有进行优化，尝试优化此部分：

假设不同的那俩元素为AB（反正设啥都一样，A、B、C地位完全一样），然后继续按照题意要求向后延伸

![延伸后续元素](https://s2.ax1x.com/2019/02/17/kyKyP1.md.png)

两个死节点当然就不用算了，

ABAA和ABBB最后俩元素一样，因此可以只算一个然后*2，

ABAB和ABBC和ABBA最后俩元素不一样，因此可以只算一个然后*3，

由此得到最终版

{% highlight cpp %}
#include <iostream>
typedef long long ll;

using namespace std;

bool ok(const int *const a, int x) {
    if (x <= 1)return true;
    if (a[x] == a[x - 1])return true;
    if (a[x - 1] == a[x - 2])return true;
    if (a[x] == a[x - 2])return true;
    return false;
}

ll countBlack(int *a, int n, int index) {
    if (index == n) {
        return 1;
    }
    ll ans = 0;

    if (index >= 2) {
        if (a[index - 2] == a[index - 1]) {
            // 前边两个数一样
            a[index] = (a[index - 1] + 1) % 3 + 1;
            ans += 2 * countBlack(a, n, index + 1);
            a[index] = a[index - 1];
            ans += countBlack(a, n, index + 1);
            return ans;
        } else if (index < n - 2) {
            // 前边两个数不一样
            a[index] = 1;
            a[index + 1] = 1;
            ans += 2 * countBlack(a, n, index + 2);
            a[index + 1] = 2;
            ans += 3 * countBlack(a, n, index + 2);
            return ans;
        }
    }

    a[index] = 1;
    if (ok(a, index)) {
        ans += countBlack(a, n, index + 1);
    }
    a[index] = 2;
    if (ok(a, index)) {
        ans += countBlack(a, n, index + 1);
    }
    a[index] = 3;
    if (ok(a, index)) {
        ans += countBlack(a, n, index + 1);
    }
    return ans;
}

int main() {
    int a[31] = {0};
    int n;
    a[0] = 1;
    while (cin >> n) {
        cout << 3 * countBlack(a, n, 1);
    }
    return 0;
}
{% endhighlight %}

`运行结果`

{% highlight cpp %}
运行时间：88ms
占用内存：992k
{% endhighlight %}

