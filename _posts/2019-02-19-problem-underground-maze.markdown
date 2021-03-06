---
layout: post
comments: true
title:  "地下迷宫 2017校招真题 滴滴"
date:   2019-02-19 22:42:30 +0800
tags: 滴滴 笔试题 校招真题 原创
lang: zh
---

<!--引用数学表达式js脚本-->
{% include mathjax_support.html %}

`原创文章`

[原题链接](https://www.nowcoder.com/practice/571cfbe764824f03b5c0bfd2eb0a8ddf)

`题目描述`
小青蛙有一天不小心落入了一个地下迷宫,小青蛙希望用自己仅剩的体力值P跳出这个地下迷宫。为了让问题简单,假设这是一个n*m的格子迷宫,迷宫每个位置为0或者1,0代表这个位置有障碍物,小青蛙达到不了这个位置;1代表小青蛙可以达到的位置。小青蛙初始在(0,0)位置,地下迷宫的出口在(0,m-1)(保证这两个位置都是1,并且保证一定有起点到终点可达的路径),小青蛙在迷宫中水平移动一个单位距离需要消耗1点体力值,向上爬一个单位距离需要消耗3个单位的体力值,向下移动不消耗体力值,当小青蛙的体力值等于0的时候还没有到达出口,小青蛙将无法逃离迷宫。现在需要你帮助小青蛙计算出能否用仅剩的体力值跳出迷宫(即达到(0,m-1)位置)。

`输入描述`
输入包括n+1行:
 第一行为三个整数n,m(3 <= m,n <= 10),P(1 <= P <= 100)
 接下来的n行:
 每行m个0或者1,以空格分隔

`输出描述`
如果能逃离迷宫,则输出一行体力消耗最小的路径,输出格式见样例所示;如果不能逃离迷宫,则输出"Can not escape!"。 测试数据保证答案唯一

`示例输入`
{% highlight cpp %}
4 4 10
1 0 0 1
1 1 0 1
0 1 1 1
0 0 1 1
{% endhighlight %}

`示例输出`
{% highlight cpp %}
[0,0],[1,0],[1,1],[2,1],[2,2],[2,3],[1,3],[0,3]
{% endhighlight %}

解题
--

`思路`

先手动运行。

`遍历图`

1. 输入将可走的`1`标记为32位int的最大值即`0x7fffffff`，将不可走的`0`标记为`-1`
2. 然后将起点标记为`0`，即所需的体力值为`0`
3. 从起点出发遍历地图，每次尝试上下左右方向走，若可走并且新路线比原路线消耗体力值更少则走新路线
4. 完成遍历地图，得到走到每一点所需的最小的体力值

示例输入遍历完后如下所示：（`-1`用`#`表示了）

{% highlight go %}
0 # # 9
0 1 # 6
# 1 2 3
# # 2 3
{% endhighlight %}

完成后发现终点（右上角）为9，而该输入示例中体力值为10可以走到终点，故开始寻找最短路

`找最短路`

从终点向起点走。因为是最短路，那么每一步肯定就是按照题意所说`上3下0左1右1`的反向来的，所以按照这个反向走一定就是终点到起点的最短路

然后将思路转换成代码即可

`注意事项`

从终点向起点找最短路的时候注意，一定要先尝试往左走。因为`起点`在`左上角`，`终点`在`右上角`，一开始我把向左走放在最后一个判断，超时，通过率只有50%，然后把最可能的，向左走放到第一个判断，就AC了

{% highlight cpp %}
#include <iostream>
#include <stack>
#include <vector>

using namespace std;
typedef pair<int, int> point;

int main() {
    int x[10][10];
    int n, m, p;
    cin >> n >> m >> p;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> x[i][j];
            if (x[i][j])// 能走
                x[i][j] = 0x7fffffff;
            else//不能走
                x[i][j] = -1;
        }
    }
    x[0][0] = 0;//起点耗费体力0
    stack<point> s;
    s.push(point(0, 0));
    while (!s.empty()) {
        point c = s.top();
        s.pop();
        int cv = x[c.first][c.second];
        //向上走 其实是向右
        point up = point(c.first, c.second + 1);
        if (up.second < m) {
            if (x[up.first][up.second] > cv + 1 &&
                x[up.first][up.second] != -1) {
                x[up.first][up.second] = cv + 1;
                s.push(up);
            }
        }
        //向左走 其实是向上
        point left = point(c.first - 1, c.second);
        if (left.first > -1) {
            if (x[left.first][left.second] > cv + 3 &&
                x[left.first][left.second] != -1) {
                x[left.first][left.second] = cv + 3;
                s.push(left);
            }
        }
        //向右走 其实是向下
        point right = point(c.first + 1, c.second);
        if (right.first < n) {
            if (x[right.first][right.second] > cv &&
                x[right.first][right.second] != -1) {
                x[right.first][right.second] = cv;
                s.push(right);
            }
        }
        //向下走 其实是向左
        point down = point(c.first, c.second - 1);
        if (down.second > -1) {
            if (x[down.first][down.second] > cv + 1 &&
                x[down.first][down.second] != -1) {
                x[down.first][down.second] = cv + 1;
                s.push(down);
            }
        }
    }
    if (x[0][m - 1] > p) {
        cout << "Can not escape!";
    } else {
        vector<point> path;
        path.emplace_back(point(0, m - 1));
        point current, dest(0, 0);
        while ((current = path.back()) != dest) {
            // 向下 其实是向左
            point down = point(current.first, current.second - 1);
            if (down.second > -1) {
                if (x[down.first][down.second] ==
                    x[current.first][current.second] - 1) {
                    path.emplace_back(down);
                    continue;
                }
            }
            // 向右 其实是向下
            point right = point(current.first + 1, current.second);
            if (right.first < n) {
                if (x[right.first][right.second] ==
                    x[current.first][current.second] - 3) {
                    path.emplace_back(right);
                    continue;
                }
            }
            // 向左 其实是向上
            point left = point(current.first - 1, current.second);
            if (left.first > -1) {
                if (x[left.first][left.second] ==
                    x[current.first][current.second]) {
                    path.emplace_back(left);
                    continue;
                }
            }
            // 向上 其实是向右
            point up = point(current.first, current.second + 1);
            if (up.second < m) {
                if (x[up.first][up.second] ==
                    x[current.first][current.second] - 1) {
                    path.emplace_back(up);
                    continue;
                }
            }
        }
        // 输出path
        for (int i = path.size() - 1; i > 0; i--) {
            cout << "[" << path[i].first << "," << path[i].second << "],";
        }
        cout << "[" << path[0].first << "," << path[0].second << "]";
    }

    // 输出遍历图
//    cout << endl;
//    for (int i = 3; i > -1; i--) {
//        for (int j = 0; j < 4; j++) {
//            if (x[j][i] != -1)
//                cout << x[j][i] << ' ';
//            else
//                cout<<"# ";
//        }
//        cout << endl;
//    }
    return 0;
}
{% endhighlight %}

`运行结果`

{% highlight cpp %}
运行时间：4ms
占用内存：476k
{% endhighlight %}


