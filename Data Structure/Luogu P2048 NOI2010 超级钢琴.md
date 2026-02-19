## 题目描述

小 Z 是一个小有名气的钢琴家，最近 C 博士送给了小 Z 一架超级钢琴，小 Z 希望能够用这架钢琴创作出世界上最美妙的音乐。

这架超级钢琴可以弹奏出 $n$ 个音符，编号为 $1$ 至 $n$。第 $i$ 个音符的美妙度为 $A_i$，其中 $A_i$ 可正可负。

一个“超级和弦”由若干个编号连续的音符组成，包含的音符个数不少于 $L$ 且不多于 $R$。我们定义超级和弦的美妙度为其包含的所有音符的美妙度之和。两个超级和弦被认为是相同的，当且仅当这两个超级和弦所包含的音符集合是相同的。

小 Z 决定创作一首由 $k$ 个超级和弦组成的乐曲，为了使得乐曲更加动听，小 Z 要求该乐曲由 $k$ 个不同的超级和弦组成。我们定义一首乐曲的美妙度为其所包含的所有超级和弦的美妙度之和。小 Z 想知道他能够创作出来的乐曲美妙度最大值是多少。

## 输入格式

输入第一行包含四个正整数 $n, k, L, R$。其中 $n$ 为音符的个数，$k$ 为乐曲所包含的超级和弦个数，$L$ 和 $R$ 分别是超级和弦所包含音符个数的下限和上限。

接下来 $n$ 行，每行包含一个整数 $A_i$，表示按编号从小到大每个音符的美妙度。

## 输出格式

输出只有一个整数，表示乐曲美妙度的最大值。

## 输入输出样例 #1

### 输入 #1

```
4 3 2 3
3
2
-6
8
```

### 输出 #1

```
11
```
## 说明/提示

### 样例解释

共有 $5$ 种不同的超级和弦：

1. 音符 $1 \sim 2$，美妙度为 $3+2=5$；
2. 音符 $2 \sim 3$，美妙度为 $2+(-6)=-4$；
3. 音符 $3 \sim 4$，美妙度为 $(-6)+8=2$；
4. 音符 $1 \sim 3$，美妙度为 $3+2+(-6)=-1$；
5. 音符 $2 \sim 4$，美妙度为 $2+(-6)+8=4$。

最优方案为：乐曲由和弦 $1,3,5$ 组成，美妙度为 $5+2+4=11$。

 ![](https://cdn.luogu.com.cn/upload/pic/2609.png) 

所有数据满足：$-1000 \leq A_i \leq 1000$，$1 \leq L \leq R \leq n$ 且保证一定存在满足要求的乐曲。
## Solution

题意简述：给定一个长度为 $n$ 的整数序列 $a$，需要选出 $k$ 个长度在 $[L, R]$ 范围内的连续子序列，使得这 $k$ 个子序列的和的总和最大。

静态问题，考虑预处理前缀和 `prefix[i]` 这样区间 $[L,R]$ 中的 $sum$ 可以使用 `prefix[R] - prefix[L-1]` 表示。找出静态前 $k$ 大，可以使用主席树（方便动态开点），也可以直接使用堆进行多路归并。

固定左端点 $i$，找到最大的 $prefix[R]$ 下标 $j$，利用 `ST` 表，快速查找 $[i+L-1,i+R-1]$ 中最大的下标位置 $j$。先将所有不同左端点的最大区间放入堆，每次取出堆顶，再取 $[i+L-1,j),[j,i+R-1)$ 范围中的最大加入队列。

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 5e5 + 5;
struct Node{
    int i, l ,r, j, val;
    bool operator<(const Node A) const{
        return val < A.val;
    }
};
priority_queue<Node> pq;
int st[N][22], pos[N][22], prefix[N], ans = 0;

int query(int l, int r){
    int j = log2(r - l + 1);
    return st[l][j] > st[r - (1 << j) + 1][j] ? pos[l][j] : pos[r - (1 << j) + 1][j];
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n, k, L, R; cin >> n >> k >> L >> R;
    for(int i = 1; i <= n; ++i){
        cin >> prefix[i]; prefix[i] += prefix[i - 1];
        st[i][0] = prefix[i], pos[i][0] = i;
    }
    for(int j = 1; j <= 20; ++j){
        for(int i = 1; i <= n - (1 << j) + 1; ++i){
            if(st[i][j - 1] > st[i + (1 << (j - 1))][j - 1]) st[i][j] = st[i][j - 1], pos[i][j] = pos[i][j - 1];
            else st[i][j] = st[i + (1 << (j - 1))][j - 1], pos[i][j] = pos[i + (1 << (j - 1))][j - 1];
        }
    }
    for(int i = 1; i <= n - L + 1; ++i){
        int idx = query(i + L - 1, min(n, i + R - 1));
        pq.push({i, i + L - 1, min(i + R - 1, n), idx, prefix[idx] - prefix[i - 1]});
    }
    while(k--){
        Node tp = pq.top();
        pq.pop(); ans += tp.val;
        if(tp.l < tp.j) pq.push({tp.i, tp.l, tp.j - 1, query(tp.l, tp.j - 1), prefix[query(tp.l, tp.j - 1)] - prefix[tp.i - 1]});
        if(tp.r > tp.j) pq.push({tp.i, tp.j + 1, tp.r, query(tp.j + 1, tp.r), prefix[query(tp.j + 1, tp.r)] - prefix[tp.i - 1]});
    }
    cout << ans << endl;
    return 0;
}
```