## 题目描述

lxhgww 最近收到了一个 $01$ 序列，序列里面包含了 $n$ 个数，下标从 $0$ 开始。这些数要么是 $0$，要么是 $1$，现在对于这个序列有五种变换操作和询问操作：

- `0 l r` 把 $[l, r]$ 区间内的所有数全变成 $0$；
- `1 l r` 把 $[l, r]$ 区间内的所有数全变成 $1$；
- `2 l r` 把 $[l,r]$ 区间内的所有数全部取反，也就是说把所有的 $0$ 变成 $1$，把所有的 $1$ 变成 $0$；
- `3 l r` 询问 $[l, r]$ 区间内总共有多少个 $1$；
- `4 l r` 询问 $[l, r]$ 区间内最多有多少个连续的 $1$。

对于每一种询问操作，lxhgww 都需要给出回答，聪明的程序员们，你们能帮助他吗？

## 输入格式

第一行两个正整数 $n,m$，表示序列长度与操作个数。  
第二行包括 $n$ 个数，表示序列的初始状态。  
接下来 $m$ 行，每行三个整数，表示一次操作。

## 输出格式

对于每一个询问操作，输出一行一个数，表示其对应的答案。

## 输入输出样例 #1

### 输入 #1

```
10 10
0 0 0 1 1 0 1 0 1 1
1 0 2
3 0 5
2 2 2
4 0 4
0 3 6
2 3 7
4 2 8
1 0 5
0 5 6
3 3 9
```

### 输出 #1

```
5
2
6
5
```

## 说明/提示

【数据范围】  
对于 $30\%$ 的数据，$1\le n,m \le 1000$；  
对于$100\%$ 的数据，$1\le n,m \le 10^5$。

## Solution

注意到本题有两种操作，两种查询。其一是区间 `set` 操作，另一种是区间 `rev`操作，查询则分别为统计区间 $1$ 的个数和最长连续 $1$ 的个数。那么核心思路就是对每个节点维护一个区间 `sum`，以及有关 $1/0$ 的三个特性，即前缀，后缀（从当前区间的左端/右端开始，最多连续的 $1/0$ 的个数），**整个区间**包含连续 $1$ 的最大长度，再加上两种懒标记即 `set` ($-1$ 表示无标记，$0$ 表示全 $0$，$1$ 表示全 $1$)，和 `rev` 区间翻转。

- 合并（`pushup`）：
    
    - `sum = sumL + sumR`（1 的个数）；
    
    - `l1 = (L.l1 == L.len ? L.len + R.l1 : L.l1)`；`r1`/`m1` 同理；
    
- 标记作用（`apply`）：
    
    - `set = 0/1`：该段变全 0/1，同时清掉 `rev`；
    
    - `rev`：交换“关于 1 的三件套”与“关于 0 的三件套”，`sum = len - sum`；
     
    - 下传（`pushdown`）时，先下放 `set`，再下放 `rev`；
    
    **Tips：标签优先级 `set` > `rev/xor/chmin/chmax(区间取min，max)` > `add`。**
    
- 查询：
    
    - `op=3` 返回 `sum`；
    
    - `op=4` 返回 `m1`（最长连续 1）。
#### 进阶合并操作

在线段树的 `pushup/pushdown` 中，懒标记和区间信息的合并总是令人头疼，于是可以写一个区间合并函数多次复用。
```cpp
// 合并两个区间信息
Node unite(const Node& L, const Node& R){
    Node x;
    x.len = L.len + R.len;
    x.setv = -1; x.rev = false;

    x.sum = L.sum + R.sum;

    // 关于 1
    x.l1 = (L.l1 == L.len) ? (L.len + R.l1) : L.l1;
    x.r1 = (R.r1 == R.len) ? (R.len + L.r1) : R.r1;
    x.m1 = max({L.m1, R.m1, L.r1 + R.l1});

    // 关于 0
    x.l0 = (L.l0 == L.len) ? (L.len + R.l0) : L.l0;
    x.r0 = (R.r0 == R.len) ? (R.len + L.r0) : R.r0;
    x.m0 = max({L.m0, R.m0, L.r0 + R.l0});

    return x;
}
```

## Code

```cpp

#include <bits/stdc++.h>
#define int long long
#define ls o << 1
#define rs o << 1 | 1
using namespace std;
const int N = 1e5 + 5;

int a[N] = {0};

struct Segment_Tree{
    struct Node{int l, r, len, sum, l1, r1, m1, l0, r0, m0, setv, rev;}t[N<<2];
    Node unite(const Node& L, const Node& R){
        Node x;
        x.len = L.len + R.len; x.setv = -1; x.rev = 0;
        x.sum = L.sum + R.sum;
        x.l1 = (L.l1 == L.len) ? (L.len + R.l1) : L.l1;
        x.r1 = (R.r1 == R.len) ? (R.len + L.r1) : R.r1;
        x.m1 = max({L.m1, R.m1, L.r1 + R.l1});

        x.l0 = (L.l0 == L.len) ? (L.len + R.l0) : L.l0;
        x.r0 = (R.r0 == R.len) ? (R.len + L.r0) : R.r0;
        x.m0 = max({L.m0, R.m0, L.r0 + R.l0});
        return x;
    }

    void apply_rev(int o){
        if(t[o].setv != -1) t[o].setv ^= 1;
        else t[o].rev ^= 1;
        swap(t[o].l1, t[o].l0);
        swap(t[o].r1, t[o].r0);
        swap(t[o].m1, t[o].m0);
        t[o].sum = t[o].len - t[o].sum;
    }
    void apply_set(int o, int v){
        t[o].setv = v, t[o].rev = 0;
        if(v == 1){
            t[o].sum = t[o].len;
            t[o].l1 = t[o].r1 = t[o].m1 = t[o].len;
            t[o].l0 = t[o].r0 = t[o].m0 = 0;
        }
        else{
            t[o].sum = 0;
            t[o].l0 = t[o].r0 = t[o].m0 = t[o].len;
            t[o].l1 = t[o].r1 = t[o].m1 = 0;
        }
    }
    void pushup(int o){
        Node x = unite(t[ls], t[rs]);
        t[o].sum = x.sum;
        t[o].l1 = x.l1; t[o].r1 = x.r1; t[o].m1 = x.m1;
        t[o].l0 = x.l0; t[o].r0 = x.r0; t[o].m0 = x.m0;
    }

    void build(int l,int r,int o){
        t[o].l=l,t[o].r=r, t[o].len = r - l + 1;
        t[o].setv = -1, t[o].rev = 0;
        if(l==r){
            t[o].sum=a[l];
            t[o].l1 = t[o].r1 = t[o].m1 = a[l] ? 1 : 0;
            t[o].l0 = t[o].r0 = t[o].m0 = a[l] ? 0 : 1;
            return ;
        }
        int mid=(l+r)>>1;
        build(l,mid,ls);build(mid+1,r,rs);
        pushup(o);
    }
    void pushdown(int o){
        if(t[o].setv != -1){
            apply_set(ls, t[o].setv);
            apply_set(rs, t[o].setv);
            t[o].setv = -1;
        }
        if(t[o].rev){
            apply_rev(ls), apply_rev(rs);
            t[o].rev = 0;
        }
    }
    void range_set(int l, int r, int v, int o = 1){
        if(r < t[o].l || t[o].r < l) return ;
        if(l <= t[o].l && t[o].r <= r){apply_set(o, v); return ;}
        pushdown(o);
        range_set(l, r, v, ls), range_set(l, r, v, rs);
        pushup(o);
    }
    void range_rev(int l, int r, int o = 1){
        if(r < t[o].l || t[o].r < l) return ;
        if(l <= t[o].l && t[o].r <= r){apply_rev(o); return ;}
        pushdown(o);
        range_rev(l, r, ls), range_rev(l, r, rs);
        pushup(o);
    }
    int query(int x,int y,int o = 1){
        if(t[o].r<x||t[o].l>y) return 0;
        if(x<=t[o].l&&t[o].r<=y) return t[o].sum;
        pushdown(o);
        return query(x,y,ls)+query(x,y,rs);
    }
    Node query_node(int L, int R, int o = 1){
        if(L <= t[o].l && t[o].r <= R) return t[o];
        pushdown(o);
        int mid = (t[o].l + t[o].r) >> 1;
        if(R <= mid) return query_node(L, R, ls);
        if(L > mid) return query_node(L, R, rs);
        Node left = query_node(L, R, ls);
        Node right = query_node(L, R, rs);
        return unite(left, right);
    }
    int query_max1(int l, int r){
        return query_node(l, r).m1;
    }
}ST;

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n, q; cin >> n >> q;
    for(int i = 1; i <= n; ++i) cin >> a[i];
    ST.build(1, n, 1);
    while(q--){
        int op, l, r; cin >> op >> l >> r; ++l, ++r;
        if(op == 0) ST.range_set(l, r, 0);
        else if(op == 1) ST.range_set(l, r, 1);
        else if(op == 2) ST.range_rev(l, r);
        else if(op == 3) cout << ST.query(l, r) << "\n";
        else cout << ST.query_max1(l, r) << "\n";
    }
    return 0;
}
```