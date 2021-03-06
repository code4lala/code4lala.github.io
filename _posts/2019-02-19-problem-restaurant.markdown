---
layout: post
comments: true
title:  "餐馆 2017校招真题 滴滴"
date:   2019-02-19 13:59:07 +0800
tags: 滴滴 笔试题 校招真题 原创
lang: zh
---

<!--引用数学表达式js脚本-->
{% include mathjax_support.html %}

`原创文章`

[原题链接](https://www.nowcoder.com/practice/d2cced737eb54a3aa550f53bb3cc19d0)

`题目描述`
某餐馆有n张桌子，每张桌子有一个参数：a 可容纳的最大人数； 有m批客人，每批客人有两个参数:b人数，c预计消费金额。 在不允许拼桌的情况下，请实现一个算法选择其中一部分客人，使得总预计消费金额最大

`输入描述`
输入包括m+2行。 第一行两个整数n(1 <= n <= 50000),m(1 <= m <= 50000) 第二行为n个参数a,即每个桌子可容纳的最大人数,以空格分隔,范围均在32位int范围内。 接下来m行，每行两个参数b,c。分别表示第i批客人的人数和预计消费金额,以空格分隔,范围均在32位int范围内。

`输出描述`
输出一个整数,表示最大的总预计消费金额

`示例输入`
{% highlight cpp %}
3 5
2 4 2
1 3
3 5
3 7
5 9
1 10
{% endhighlight %}

`示例输出`
{% highlight cpp %}
20
{% endhighlight %}

解题
=

`思路`

1. 人数过多以至于最大的桌子都坐不下的直接走
2. 谁钱多谁先上座
3. 找空桌的时候找能放下这群人的最小容量的桌子

`找桌子的技巧`

先新增一种容量正好等于这桌人的人数的桌子，这种桌子的数量为0，然后就可以进入循环了。完事把这个桌子删了就ok了。

{% highlight cpp %}
#include<iostream>
#include<map>
#include<vector>
#include<algorithm>

typedef long long ll;
using namespace std;
typedef struct tagCustomer {
    int people;
    int money;

    bool operator<(const tagCustomer &o) const {
        return money < o.money;
    }

    bool operator>(const tagCustomer &o) const {
        return money > o.money;
    }

    tagCustomer(int t1, int t2) {
        people = t1;
        money = t2;
    }
} customer;

int main() {
    ios::sync_with_stdio(false);
    int n, m;
    map<int, int> tables;//桌子容量 该容量桌子的个数
    vector<customer> c;//每批客人的 人数 金额
    cin >> n >> m;
    int tableCount = n;
    int aMax = 0x80000000;// 初始化为 int 的最小值
    int t1, t2;
    for (int i = 0; i < n; i++) {
        cin >> t1;
        tables[t1]++;
        if (t1 > aMax)aMax = t1;
        // 统计出最大的桌子容量
    }
    for (int i = 0; i < m; i++) {
        cin >> t1;
        if (t1 > aMax) {
            cin >> t1;
            // 不合并桌子的话不可能坐下，直接排除
            continue;
        }
        cin >> t2;
        c.emplace_back(customer(t1, t2));
    }
    sort(c.rbegin(), c.rend());// 降序排列
    // 按金额降序排列，因为要求的是最大金额
    ll ans = 0;
    // 对每组人，找能容下这组人的最小容量的桌子
    for (customer cust:c) {
        // 所有桌子都用完了直接退出
        if (tableCount <= 0)break;
        // 使用技巧找空桌子
        // 先创建一个正好能坐所要求的人的个数的桌子，这个桌子的数量为0
        tables[cust.people] += 0;
        // 然后就可以用这个桌子开始用iterator遍历了，不然for循环进不去
        // 注意，此处用的是find函数而不是判断这个key对应的值是不是0
        // 如果用后一种方法，那么会增加好多key对应value为0的无效元素，无故增加运行时间
        for (auto it = tables.find(cust.people); it != tables.end(); it++) {
            if (it->second > 0) {
                tables[it->first]--;
                ans += cust.money;
                // 找到了所要求的的桌子，坐完这桌人之后这种桌子就没有了，那么就删掉
                if (tables[it->first] == 0) {
                    tables.erase(it->first);
                }
                // 桌子总数--
                tableCount--;
                break;
            }
        }
        // 完事后删除那个不存在的桌子
        if (tables[cust.people] == 0) {
            tables.erase(cust.people);
        }
    }
    cout << ans;
    return 0;
}
{% endhighlight %}

`运行结果`

{% highlight cpp %}
运行时间：63ms
占用内存：2920k
{% endhighlight %}


