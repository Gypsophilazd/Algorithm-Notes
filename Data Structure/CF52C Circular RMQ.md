## 题目描述

给定一个环形数组 $a_{0},a_{1},...,a_{n-1}$。有两种操作：

- $inc(lf,rg,v)$——该操作将区间 $[lf,rg]$（包括两端）内的每个元素加上 $v$；
- $rmq(lf,rg)$——该操作返回区间 $[lf,rg]$（包括两端）内的最小值。

假设区间为环形区间，因此若 $n=5$ 且 $lf=3,rg=1$ ，则所指索引依次为： $3,4,0,1$。

请编写程序处理给定的操作序列。

## 输入格式

第一行为整数 $n$（$1 \leq n \leq 200000$）。  
第二行为数组的初始状态：$a_{0},a_{1},...,a_{n-1}$（$-10^6 \leq a_{i} \leq 10^6$），$a_{i}$ 为整数。  
第三行为整数 $m$（$0 \leq m \leq 200000$），表示操作的个数。  
接下来的 $m$ 行，每行为一个操作。如果该行有两个整数 $lf,rg$（$0 \leq lf,rg \leq n-1$），则表示 $rmq$ 操作；如果有三个整数 $lf,rg,v$（$0 \leq lf,rg \leq n-1 ; -10^6 \leq v \leq 10^6$），则表示 $inc$ 操作。

## 输出格式

对于每个 $rmq$ 操作，输出其结果。请不要在 C++ 中使用 %lld 格式符读取或输出 64 位整数，建议使用 cout（也可以使用 %I64d）。

## 输入输出样例 #1

### 输入 #1

```
4
1 2 3 4
4
3 0
3 0 -1
0 1
2 1
```

### 输出 #1

```
1
0
0
```

## Solution

很显然，区间加操作+区间求 $\operatorname{min}$ 操作，可以直接套线段树板子。

环形数组换汤不换药，只要把区间拆成两段就可以满足要求。

这里建议默写线段树板子喵~

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
#define ls o << 1
#define rs o << 1 | 1
using namespace std;
const int N = 2e5 + 5;
int space;
int a[N] = {0};
struct Segment_Tree{
    struct Node{int l,r,sum,mn,lazy;}t[N<<2];

    void lazy_union(int o, int v){
        t[o].sum += (t[o].r - t[o].l + 1) * v;
        t[o].mn += v; t[o].lazy += v;
    }
    void pushup(int o){
        t[o].sum=t[ls].sum+t[rs].sum;
        t[o].mn=min(t[ls].mn,t[rs].mn);
    }

    void build(int l,int r,int o){
        t[o].l=l,t[o].r=r;
        if(l==r){
            t[o].sum=a[l];
            t[o].mn=a[l];
            return ;
        }
        int mid=(l+r)>>1;
        build(l,mid,ls);build(mid+1,r,rs);
        pushup(o);
    }
    void pushdown(int o){
        if(t[o].lazy){
            lazy_union(ls,t[o].lazy),lazy_union(rs,t[o].lazy);
            t[o].lazy = 0;
        }
    }
    void add(int x,int y,int k,int o){
        if(t[o].r<x||t[o].l>y) return ;
        if(x<=t[o].l&&t[o].r<=y){
            lazy_union(o, k);
            return ;
        }
        pushdown(o);
        add(x,y,k,ls),add(x,y,k,rs);
        pushup(o);
    }

    int query(int x,int y,int o){
        if(t[o].r<x||t[o].l>y) return 1e18;
        if(x<=t[o].l&&t[o].r<=y) return t[o].mn;
        pushdown(o);
        return min(query(x,y,ls), query(x,y,rs));
    }

}ST;

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n; cin >> n;
    for(int i = 0; i < n; ++i) cin >> a[i];
    ST.build(0, n - 1, 1);
    int m; cin >> m;
    while(m--){
        int l, r; cin >> l >> r;
        if(cin.peek() == ' '){
            int v; cin >> v;
            if(l <= r) ST.add(l, r, v, 1);
            else ST.add(l, n - 1, v, 1), ST.add(0, r, v, 1);
        }
        else{
            if(l <= r) cout << ST.query(l, r, 1) << "\n";
            else cout << min(ST.query(l, n - 1, 1), ST.query(0, r, 1)) << "\n";
        } 
    }
    return 0;
}
```