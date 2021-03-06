---
layout: post
comments: true
title:  "跳石板 2017校招真题 网易"
date:   2019-02-17 12:05:12 +0800
tags: 网易 笔试题 校招真题 动态规划 原创
lang: zh
---

<!--引用数学表达式js脚本-->
{% include mathjax_support.html %}

`原创文章`

[原题链接](https://www.nowcoder.com/practice/4284c8f466814870bae7799a07d49ec8)

`题目描述`
小易来到了一条石板路前，每块石板上从1挨着编号为：1、2、3.......
这条石板路要根据特殊的规则才能前进：对于小易当前所在的编号为K的 石板，小易单次只能往前跳K的一个约数(不含1和K)步，即跳到K+X(X为K的一个非1和本身的约数)的位置。 小易当前处在编号为N的石板，他想跳到编号恰好为M的石板去，小易想知道最少需要跳跃几次可以到达。
例如：
N = 4，M = 24：
4->6->8->12->18->24
于是小易最少需要跳跃5次，就可以从4号石板跳到24号石板

`输入描述`
输入为一行，有两个整数N，M，以空格隔开。 (4 ≤ N ≤ 100000) (N ≤ M ≤ 100000)

`输出描述`
输出小易最少需要跳跃的步数,如果不能到达输出-1

`示例输入`
{% highlight cpp %}
4 24
{% endhighlight %}

`示例输出`
{% highlight cpp %}
5
{% endhighlight %}

解题
=

`思路`
先在草纸上画个开头

![遍历图](https://s2.ax1x.com/2019/02/17/kySnwF.png)

然后发现依次遍历即可

`注意事项`

值得注意的一点，程序中两个循环，一个是外层的遍历从n到m的循环，另一个内层的求当前数的所有因数的循环，内层循环的循环范围应为从$$2$$到$$\sqrt{i}$$。

此处有一个疑点，例如12的除了1和本身的所有因数分别为2、3、4、6，而$$\sqrt{12}=3.464$$，那么从4开始往后的因数就遍历不到了，因此循环中要进行两次运算，比如遍历到因数为2时，将6也顺便算了；遍历到因数为3时，将4也顺便算了。

因为我本来是从$$i$$遍历到$$i/2$$，结果一直超时，case通过率只有70%。然后参考了一下别人的思路，[讨论区](https://www.nowcoder.com/questionTerminal/4284c8f466814870bae7799a07d49ec8) 中@`脱缰的哈士奇～`的回答，就这里不一样。换了这个循环范围就可以了。

{% highlight cpp %}
#include <iostream>
#include <algorithm>
#include <cmath>

using namespace std;

int main() {
    int n, m;
    int a[100005] = {0};
    int intMax = 999999999;
    for (int i = 0; i < 100005; i++) {
        a[i] = intMax;
    }
    cin >> n >> m;
    a[n] = 0;
    int iMax = m + (int) (m / 2) + 1;
    iMax = min(iMax, 100004);
    int i;
    // 外层循环遍历从n到m
    for (i = n; i <= iMax; i++) {
        if (a[i] == intMax)continue;
        int xMax = (int) sqrt(i) + 1;
        // 内层循环遍历当前数的所有因数
        for (int j = 2; j <= xMax && i + j < 100005; j++) {
            if (i % j == 0) {
                if (a[i + j] > a[i] + 1) {
                    a[i + j] = a[i] + 1;
                    if (i + j == m) {
                        cout << a[i + j];
                        return 0;
                    }
                }
                int k = i / j;
                if (i + k < 100005) {
                    if (a[i + k] > a[i] + 1) {
                        a[i + k] = a[i] + 1;
                        if (i + k == m) {
                            cout << a[i + k];
                            return 0;
                        }
                    }
                }
            }
        }
    }
    if (a[m] != intMax) {
        cout << a[m];
    } else {
        cout << -1;
    }
    return 0;
}
{% endhighlight %}

`运行结果`

运行时间：88ms

占用内存：992k




