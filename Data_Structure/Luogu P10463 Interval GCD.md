![](images/Pasted image 20250823141015.png)
## 题目描述

给定一个长度为 $N$ 的数列 $a$，以及 $M$ 条指令，每条指令可能是以下两种之一：

1. `C l r d`，表示把 $a_l,a_{l+1},…,a_r$ 都加上 $d$。
2. `Q l r`，表示询问 $a_l,a_{l+1},…,a_r$ 的最大公约数（$\gcd$）。

对于每个询问，输出一个整数表示答案。

## 输入格式

第一行两个整数 $N,M$。

第二行 $N$ 个整数，分别表示 $a_1,a_2,\dots,a_N$。

接下来 $M$ 行表示 $M$ 条指令，每条指令的格式如题目描述所示。

## 输出格式

对于每个询问，输出一个整数表示答案，每个答案占一行。

## 输入输出样例 #1

### 输入 #1

```
5 5
1 3 5 7 9
Q 1 5
C 1 5 1
Q 1 5
C 3 3 6
Q 2 4
```

### 输出 #1

```
1
2
4
```

## 说明/提示

对于 $100\%$ 的测试数据，$N \le 5\times10^5$，$M \le 10^5$，$1 \le a_i \le 10^{18}$，$|d| \le 10^{18}$，保证数据在计算过程中不会超过 long long 范围。

### Solution
区间修改+区间查询,很明显的线段树模版,但是需要作一些改动。

由最大公因数性质：

 $$\gcd(a) = \gcd\{\gcd(a1,a2),\gcd(a2,a3)...\gcd(a_{n-1},a_n)\}$$  
从而由 $gcd(x, y) = gcd(y, x\bmod{y})$ 可得：

$$\gcd\{a\} = \gcd\{gcd(a1,a2 - a1),\gcd(a2,a3 - a2)...\gcd(a_{n-1},a_n - a_{n-1})\}$$ 
由 $\gcd(x,y)=\gcd(x,y−kx)(∀k∈Z)$ 可以消去冗余项 从而得到：

$$\gcd\{a\} = \gcd\{a1,a2 - a1, a3 - a2...a_n - a_{n-1})\}$$ 
即 $a$ 的数组的 $\gcd$ 等同于 $a$ 差分数组的 $\gcd$。

因此,我们操作步骤如下：

1. 使用线段树维护 $a$ 的差分数组，便于完成单点修改。

2. 单点修改后，维护一个区间 $\operatorname{sum}$ 即原数组的 $a[x]$，在线段树维护差分数组 $\gcd$ 的情况下，$\gcd(a[x], \operatorname{query}(x + 1, y))$即可。

优化卡常部分，可以考虑势能操作。

即根据 $\gcd$ 的*线性组合不变性*,原数组的 $\gcd$ 等同于首项 $a[x]$ 和区间差分数组的 $\gcd$。

### AC Code
```
#include <bits/stdc++.h>
#define int long long
#define ls (o << 1)
#define rs (o << 1 | 1)
using namespace std;
const int N = 5e5 + 5;
int a[N], n, m;
struct Tree{
    int sum, gcd;
}t[N << 2];

int _gcd(int x, int y){return (y == 0 ? x : _gcd(y, x % y));}

void pushup(int o){t[o] = {t[ls].sum + t[rs].sum, _gcd(t[ls].gcd, t[rs].gcd)};}

void build(int st = 1, int ed = n, int o = 1){
    if(st == ed) return t[o] = {a[st] - a[st - 1], a[st] - a[st - 1]}, void();
    int mid = (st + ed) >> 1;
    build(st, mid, ls);
    build(mid + 1, ed, rs);
    pushup(o);
}
  
void update(int pos, int d, int st = 1, int ed = n, int o = 1){
    if(st == ed){
        t[o].sum += d;
        t[o].gcd += d;
        return ;
    }

    int mid = (st + ed) >> 1;
    if(pos <= mid) update(pos, d, st, mid, ls);
    else update(pos, d, mid + 1, ed, rs);
    pushup(o);
}

int sum(int l, int r, int st = 1, int ed = n, int o = 1){
    if(ed < l || st > r) return 0;
    if(st >= l && ed <= r) return t[o].sum;
    int mid = (st + ed) >> 1;
    return sum(l, r, st, mid, ls) + sum(l, r, mid + 1, ed, rs);
}

int query(int l, int r, int st = 1, int ed = n, int o = 1){
    if(ed < l || st > r) return 0;
    if(st >= l && ed <= r) return abs(t[o].gcd);
    int mid = (st + ed) >> 1;
    return _gcd(query(l, r, st, mid, ls), query(l, r, mid + 1, ed, rs));
}

  

signed main(){
    ios::sync_with_stdio(0), cin.tie(0);
    cin >> n >> m;
    for(int i = 1; i <= n; ++i) cin >> a[i];
    build();
    while(m--){
        char op; cin >> op;
        if(op == 'C'){
            int x, y, d; cin >> x >> y >> d;
            update(x, d);
            if(y != n) update(y + 1, -d);
        }
        else{
            int x, y; cin >> x >> y;
            if(x == y) cout << sum(1, x) << endl;
            else{
                int ax = sum(1, x);
                int g = query(x + 1, y);
                cout << _gcd(ax, g) << endl;
            }
        }
    }
    return 0;
}
```