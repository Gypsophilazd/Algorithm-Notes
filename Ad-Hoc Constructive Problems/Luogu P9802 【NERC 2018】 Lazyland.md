## 题目背景

翻译自 [NERC 2018](https://neerc.ifmo.ru/archive/2018/neerc-2018-statement.pdf) L 题。

## 题目描述

一个城市里有 $n$ 个人，和 $k$ 种职业，其中，每个人都有一个现在正在从事的职业 $a_i$，但是很遗憾，由于某些职业的空缺，使得这个城市根本无法继续正常生存下去。

你作为一城之主，自然不希望看到自己的城市这样子下去，你决定说服其中的某些人，使得每种职业都有人在做，对于每个人 $i$，你需要耗费 $b_i$ 的时间去说服。

你需要求出你去说服的时间的最小值。

## 输入格式

第一行两个整数 $n$ 和 $k (1 \leq k \leq n \leq 10^5)$，分别表示 $n$ 个人和 $k$ 种职业。

第二行 $n$ 个整数 $a_i (1 \leq a_i \leq k)$，表示第 $i$ 个人正在从事的职业 。

第三行 $n$ 个整数 $b_i (1 \leq b_i \leq 10^9)$，表示第 $i$ 个人被说服去做别的职业的时间。

## 输出格式

输出去说服市民的最小时间。

## 输入输出样例 #1

### 输入 #1

```
8 7
1 1 3 1 5 3 7 1
5 7 4 8 1 3 5 2
```

### 输出 #1

```
10
```

## 输入输出样例 #2

### 输入 #2

```
3 3
3 1 2
5 3 4
```

### 输出 #2

```
0
```

## 说明/提示

对于所有的数据，保证 $1 \leq k \leq n \leq 10^5$，$1 \leq a_i \leq k$，$1 \leq b_i \leq 10^9$。

对于样例一，分别令 $1$，$6$，$8$ 号市民去从事 $2$，$4$，$6$ 号职业。


## Solution

### 题意重述

有 $n$ 个人，$k$ 种职业，每个人的职业是 $a_i$，为了使得职业没有空缺，每个人又可以被花费 $b_i$ 的时间说服去做需要做的职业，求总花费时间最小值。

### 解法

考虑贪心+排序。将每个人的两种属性按 $b_i$ 排序。

开一个桶记录所有职业的人数，记录 `cur` 为现在不同的职业数，随后按说服时间升序遍历，如果该职业中已经存在 $2$ 个及以上的人，那么可以把他加入到 `cur` 中，最后判断 `cur` 和 `k` 的关系即可。

## Code

```cpp
#include <bits/stdc++.h>
#define int long long 
using namespace std;
const int N = 1e5 + 5;
struct Node{
    int m, t;
    bool operator <(Node &A){
        return t < A.t;
    }
}a[N];
int cnt[N], cur;

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n, k; cin >> n >> k;
    for(int i = 1; i <= n; ++i){
        cin >> a[i].m;
        if(cnt[a[i].m] == 0) cur++;
        cnt[a[i].m]++;
    }
    for(int i = 1; i <= n; ++i) cin >> a[i].t;
    sort(a + 1, a + 1 + n);
    int ans = 0, idx = 1;
    while(cur < k){
        if(cnt[a[idx].m] > 1) cnt[a[idx].m]--, cur++, ans += a[idx].t;
        idx++;
    }
    cout << ans << endl;
    return 0;
}
```
