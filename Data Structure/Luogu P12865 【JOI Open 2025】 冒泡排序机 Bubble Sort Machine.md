![](images/Pasted image 20250819165000.png)
## 题目描述

JOI 君——一名算法工程师，开发了冒泡排序机。

冒泡排序机操作长为 $N$ 的整数序列 $a=(a_1,a_2,\ldots,a_N)$。为了让机器能工作，给定 $A_i$ 作为 $a_i$（$1\le i\le N$）的初值。每当机器上的**按钮壹**被按下时，机器会按照如下方式修改序列 $a$：

> 对于 $i=1,2,\ldots,N-1$（按此顺序），若 $a_i\gt a_{i+1}$，交换 $a_i,a_{i+1}$ 的值。

为了使冒泡排序机更博人眼球，JOI 君决定加入以下功能：

> 当**按钮贰**被按下时，给定整数 $l,r$ 作为输入（须满足 $1\le l\le r\le N$），机器会输出 $a_{l}+a_{l+1}+\cdots+a_r$ 的值。

给定整数序列的初值和冒泡排序机的操作序列，编程计算按钮贰的输出值。

## 输入格式

输入格式如下所示：

> $N$ \
> $A_1$ $A_2$ $\cdots$ $A_N$ \
> $Q$ \
> $(\text{Query }1)$ \
> $(\text{Query }2)$ \
> $\vdots$ \
> $(\text{Query }Q)$

这里，$Q$ 指的是冒泡排序机的操作数。每个 $(\text{Query }j)$（$1\le j\le Q$）由若干个空格分隔的数字组成。令 $T_j$ 为 $(\text{Query }j)$ 的首个数字。这行的内容为以下二者之一：

- 若 $T_j=1$，这行再没有其他整数了。这意味着冒泡排序机的第 $j$ 次操作按下了按钮壹。
- 若 $T_j=2$，接下来还有两个整数，依次是 $L_j,R_j$。这意味着冒泡排序机的第 $j$ 次操作按下了按钮贰，给定的输入为 $L_j,R_j$。

## 输出格式

对每个按下按钮贰的操作［意思是，对每个满足 $T_j=2$ 的 $j$（$1\le j\le Q$）］，输出一行一个整数，表示冒泡排序机的输出。你的输出应与询问的顺序相符。

## 输入输出样例 #1

### 输入 #1

```
4
5 3 5 2
6
2 1 3
1
2 1 1
2 2 4
1
2 1 2
```

### 输出 #1

```
13
3
12
5
```

## 输入输出样例 #2

### 输入 #2

```
5
1 1 2 1 2
5
2 2 3
1
2 2 4
1
2 2 4
```

### 输出 #2

```
3
4
4
```

## 说明/提示

### 样例解释

#### 样例 $1$ 解释

初值为 $a_1=5,a_2=3,a_3=5,a_4=2$，$a=(5,3,5,2)$。接下来在冒泡排序机上操作：

1. 按下按钮贰，输入 $l=1,r=3$。机器输出 $a_1+a_2+a_3=13$。
2. 按下按钮壹。对 $i=1,2,3$，按此顺序进行如下操作：
    - $i=1$：由于 $a_1\gt a_2$，交换二者的值，操作后 $a=(3,5,5,2)$。
    - $i=2$：由于并没有 $a_2\gt a_3$，不操作 $a$。
    - $i=3$：由于 $a_3\gt a_4$，交换二者的值，操作后 $a=(3,5,2,5)$。
3. 按下按钮贰，输入 $l=1,r=1$。机器输出 $a_1=3$。
4. 按下按钮贰，输入 $l=2,r=4$。机器输出 $a_2+a_3+a_4=12$。
5. 按下按钮壹。对 $i=1,2,3$，按此顺序进行如下操作：
    - $i=1$：由于并没有 $a_1\gt a_2$，不操作 $a$。
    - $i=2$：由于 $a_2\gt a_3$，交换二者的值，操作后 $a=(3,2,5,5)$。
    - $i=3$：由于并没有 $a_3\gt a_4$，不操作 $a$。
6. 按下按钮贰，输入 $l=1,r=2$。机器输出 $a_1+a_2=5$。

样例 $1$ 满足子任务 $1,5,6$ 的限制。

#### 样例 $2$ 解释

样例 $2$ 满足子任务 $1,3,5,6$ 的限制。


### 数据范围

- $2\le N\le 500\, 000$；
- $1\le A_i\le 10^9\, (1\le i\le N)$；
- $1\le Q\le 500\, 000$；
- $T_j\in \{1,2\}\, (1\le j\le Q)$；
- 若 $T_j=2$，有 $1\le L_j\le R_j\le N\, (1\le j\le Q)$；
- 输入的值都是整数。

### 子任务

- $\text{Subtask 1 (5 pts)}$：满足 $T_j=1$ 的 $j\,(1\le j\le Q)$ 至多有 $10$ 个；
- $\text{Subtask 2 (11 pts)}$：$N,Q\le 150\, 000$，当 $T_j=2$ 时 $L_j=R_j=1\, (1\le j\le Q)$；
- $\text{Subtask 3 (15 pts)}$：$N,Q\le 150\, 000$，$1\le A_i\le 2\, (1\le i\le N)$；
- $\text{Subtask 4 (23 pts)}$：$N,Q\le 150\, 000$，当 $T_j=2$ 时 $L_j=R_j\, (1\le j\le Q)$；
- $\text{Subtask 5 (29 pts)}$：$N,Q\le 150\, 000$；
- $\text{Subtask 6 (17 pts)}$：无额外限制。

## Solution

有一个关键的观察，即冒泡排序若干轮的结果是可以直接刻画的。具体来说，前缀 $[1,l]$ 在经过 $k$ 轮冒泡排序之后的数字构成是原先区间 $[1,l+k]$ 中的前 $l$ 小的数，这个很好理解，经过 $k$ 轮冒泡排序之后，某个数最多往前平移的位置数就是 $k$，所以范围是 $[1,l+k]$，而小的数会往前走，所以就是前 $l$ 小。

按 $k$ 次后，$[1,r]$ 的和 = **初始数组中** $[1,\min(r+k,n)]$ 里**最小的 r 个数的和**。

于是一般区间 $[l,r$] 的答案就是 $F(k,r) - F(k,l-1)$，其中 $F(k,t)= [1,\min(t+k,n)]$ 中**最小的 t 个数之和**。

所以只需要做一个静态操作：**对每个前缀长度 m，快速求出其中最小的 t 个数的和**。

做法是按下标从 $1$ 到 $n$ 依次把 $A_i​$ 插入一棵值域主席树里，得到每个前缀的根 `rt[i]`。每个节点同时维护：

`cnt`：这个值域段内出现了多少个数；
`sum`：这个值域段内所有数值数值和。

询问前缀 $m$ 中最小的 $t$ 个数的和时，沿树走：看左儿子的 $cnt$ 是否 $\ge t$。如果是就全在左边；否则答案先加上左儿子的 $sum$，然后去右儿子找剩下的 $t-\text{cnt(left)}$ 个。复杂度 $O(\log M)$，$M$ 为值域大小（离散化后 $\le n$）。

有一个小细节：如果叶子节点的 $cnt \ge k$，可以先截断当前叶子节点的值，然后再乘 $k$。具体可以看代码。

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;

const int N = 5e5 + 5;
int a[N], k = 0, n, B;
vector<int> X;
int pos[N];
int getpos(const vector<int> &v, int x){
    return lower_bound(v.begin(), v.end(), x) - v.begin() + 1;
}

struct Persistent_Segment_Tree{
    struct Node{int l, r, sum, cnt;} t[N * 25];
    int rt[N], tot = 0;
    int cpy(int from = 0){t[++tot] = t[from]; return tot;}

    int update(int pre, int l, int r, int pos, int val){
        int o = ++tot;
        t[o] = t[pre]; t[o].cnt++, t[o].sum += val;
        if(l == r) return o;
        int mid = (l + r) >> 1;
        if(pos <= mid) t[o].l = update(t[pre].l, l, mid, pos, val);
        else t[o].r = update(t[pre].r, mid + 1, r, pos, val);
        return o;
    }

    int query(int o, int l, int r, int pos){
        if(l == r) return t[o].sum;
        int mid = (l + r) >> 1;
        if(pos <= mid) return query(t[o].l, l, mid, pos);
        else return query(t[o].r, mid + 1, r, pos);
    }

    int ksum(int o, int l, int r, int k){
        if(k <= 0 || o == 0) return 0ll;
        if(l == r){
            if(t[o].cnt == 0) return 0ll;
            int v = t[o].sum / t[o].cnt;
            return v * k;
        }
        int mid = (l + r) >> 1;
        int L = t[o].l, R = t[o].r;
        int Lcnt = t[L].cnt;
        if(k <= Lcnt) return ksum(L, l, mid, k);
        else return t[L].sum + ksum(R, mid + 1, r, k - Lcnt);
    }
    int build0(int l, int r){
        int o = ++tot;
        t[o] = {0, 0, 0, 0};
        if(l == r) return o;
        int mid = (l + r) >> 1;
        t[o].l = build0(l, mid), t[o].r = build0(mid + 1, r);
        return o;
    }
}PST;

int calc(int t){
    if(t <= 0) return 0ll;
    int m = t + k;
    if(m > n) m = n;
    return PST.ksum(PST.rt[m], 1, B, t);
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    cin >> n;
    X.reserve(n);
    for(int i = 1; i <= n; ++i){
        cin >> a[i];
        X.push_back(a[i]);
    }
    sort(X.begin(), X.end()); X.erase(unique(X.begin(), X.end()), X.end());
    B = X.size();
    for(int i = 1; i <= n; ++i) pos[i] = getpos(X, a[i]);
    PST.rt[0] = 0;
    for(int i = 1; i <= n; ++i) PST.rt[i] = PST.update(PST.rt[i - 1], 1, B, pos[i], a[i]);
    int q; cin >> q;
    while(q--){
        int op; cin >> op;
        if(op == 1) ++k;
        else{
            int l, r; cin >> l >> r;
            int ans = calc(r) - calc(l - 1);
            cout << ans << '\n';
        }
    }

    return 0;
}
```