## 题目描述

**译自 [BalticOI 2009](http://www.csc.kth.se/contest/boi/tasks.php) Day2 T1「[Rectangle](https://www.csc.kth.se/contest/boi/rectangle.pdf)」**

你被给定 $n$ 个在平面坐标上的点。

写一个程序计算出一个最大的矩形区域，使它的每一个顶点都是给定的点之一。你可以假设至少存在一个这样的区域。

## 输入格式

第一行，一个整数 $n$，表示点的数量。

以下 $n$ 行，每行两个整数，表示一个点的坐标。
坐标值在 $-10^8$ 与 $10^8$ 之间。

没有两个点位于同一个坐标。

## 输出格式

一个整数，表示最大的矩形区域面积。

## 输入输出样例 #1

### 输入 #1

```
8
-2 3
-2 -1
0 3
0 -1
1 -1
2 1
-3 1
-2 1
```

### 输出 #1

```
10
```

## 说明/提示

![](https://cdn.luogu.com.cn/upload/image_hosting/x481b6hi.png)

### 数据范围：

| 数据百分比 | 限制 |
| :----------: | :----------: |
| $20\%$ | $n \le 500$ |
| $100\%$ | $4 \le n \le 1500$ |

## Solution

题意已经给的很明白了，给定 $n$ 个点，求这些点可以形成的最大的矩形面积。
一个 naive 的想法是直接 $O(n^4)$ 的枚举每个点对，先判定是否为矩形再计算最大面积，固然可以成立，但是必然超时。

但是，我们发现，枚举出一条对角线后，不一定要枚举第二条对角线。因为第二条符合要求的对角线一定和第一条对角线的长度相同，而且中心点的横纵坐标相同。由此，我们可以想到通过排序来解决。

先枚举任意两点找到所有对角线，然后以对角线长度为的第一关键字，对角线的中心的横纵坐标为第二第三关键字。排完序后的对角线数组满足一个规律，就是每次遍历到一个对角线，然后往后遍历的时候，如果有一个对角线的长度或者中心点都不匹配，那么后面绝对不会再遇到匹配的，而且遍历开始的那一条对角线到临界的对角线中间的这一部分都是可以互相匹配的。

由此，我们就可以先枚举对角线，然后排序，然后再遍历这个对角线。虽然上面的文字看起来需要双重循环，但是有一个小小的优化。就是每次遍历到一个临界值后，通过上面的性质，在这一个集合的点两两配对计算后第一轮循环就可以直接跳过这个集合。

而且我们还可以证明，如果 $n$ 个点共圆，那么对角线只有 $2n$​ 条，不会超时。

## Code

```
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 3e6 + 5;
int x[N], y[N], tot, ans;
struct Edge{
    int x, y, i, j;
    long double len;
    bool operator <(const Edge& A){
        if(len != A.len) return len > A.len;
        if(x != A.x) return x < A.x;
        return y < A.y;
    }
}e[N];

long double calc(int i, int j){
    return (x[i] - x[j]) * (x[i] - x[j]) + (y[i] - y[j]) * (y[i] - y[j]);
}

void Insert(int i, int j){
    e[++tot].len = calc(i, j);
    e[tot].x = (x[i] + x[j]), e[tot].y = (y[i] + y[j]);
    e[tot].i = i, e[tot].j = j;
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n; cin >> n;
    for(int i = 1; i <= n; ++i) cin >> x[i] >> y[i];
    for(int i = 1; i <= n; ++i){
        for(int j = i + 1; j <= n; ++j) Insert(i, j);
    }
    sort(e + 1, e + 1 + tot);
    for(int i = 1; i <= tot; ++i){
        int lst = i;
        while(e[i].x == e[i + 1].x && e[i].y == e[i + 1].y && e[i].len == e[i + 1].len){
            for(int j = lst; j <= i; ++j){
                long double s1 = sqrt(calc(e[j].i, e[i + 1].i));
                long double s2 = sqrt(calc(e[j].j, e[i + 1].i));
                int s = 0;
                s = round(s1 * s2);
                ans = max(s, ans);
            }
            i++;
        }
    }
    cout << ans << endl;
    return 0;
}
```