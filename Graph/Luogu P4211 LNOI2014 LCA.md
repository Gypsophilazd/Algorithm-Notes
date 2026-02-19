## 题目描述

给出一个 $n$ 个节点的有根树（编号为 $0$ 到 $n-1$，根节点为 $0$）。

一个点的深度定义为这个节点到根的距离 $+1$。 

设 $dep[i]$ 表示点 $i$ 的深度，$\operatorname{LCA}(i, j)$ 表示 $i$ 与 $j$ 的最近公共祖先。 

有 $m$ 次询问，每次询问给出 $l, r, z$，求 $\sum_{i=l}^r dep[\operatorname{LCA}(i,z)]$ 。

## 输入格式

第一行 $2$ 个整数，$n, m$。

接下来 $n-1$ 行，分别表示点 $1$ 到点 $n-1$ 的父节点编号。

接下来 $m$ 行，每行 $3$ 个整数，$l, r, z$。

## 输出格式

输出 $m$ 行，每行表示一个询问的答案。每个答案对 $201314$ 取模输出。

## 输入输出样例 #1

### 输入 #1

```
5 2
0
0
1
1
1 4 3
1 4 2
```

### 输出 #1

```
8
5
```

## 说明/提示

### 数据范围及约定

- 对于 $20\%$ 的数据，$n\le 10000,m\le 10000$；
- 对于 $40\%$ 的数据，$n\le 20000,m\le 20000$；
- 对于 $60\%$ 的数据，$n\le 30000,m\le 30000$；
- 对于 $80\%$ 的数据，$n\le 40000,m\le 40000$；
- 对于 $100\%$ 的数据，$1\le n\le 50000,1\le m\le 50000$。

## Solution

题意简述：多次询问 $\sum_{i=l}^{r}\mathrm{dep}(\mathrm{LCA}(i,z))$。对固定 $z$，枚举 $i\in[l,r]$。对 $i$ 与 $z$ 的最近公共祖先的深度求和。

对样例进行打表+模拟，可以得出一个关键观察： $$\sum_{i=l}^r dep[\operatorname{LCA}(i,z)]= u∈path(root,z)\space∑​\#\{i∈[l,r]∣i∈subtree(u)\}.$$
即$\mathrm{dep}(\mathrm{LCA}(i,z))$ 等价于根到 $z$ 的路径上，有多少个点同时是 $i$ 的祖先。

简单来说，就是**每次把询问区间 $[l,r]$ 里的点到根节点路径上的点权值（包括自己）加一，最后询问 $z$ 到根节点的权值和。**

$m$ 与 $n$ 同阶，面对大量询问，考虑离线排序询问，进行重链剖分，并对树的 $dfn$  建立线段树维护区间和，如此以来，单次跳链的时间复杂度为 $O(logn)$，区间修改的时间复杂度为 $O(logn)$，但每次修改不同的 $[l,r]$ 并要进行回滚，朴素修改下总的时间复杂度为 $O(n^2log^2n)$，离线下对询问使用差分进行 $O(1)$ 修改进行优化。

接下来推导一些公式：

深度为“**从根到该点的节点个数**”，根恒为 $root=0$，深度从 $1$ 开始计。

**结论 $1$**：
对任意两个点 $i,z$，

$$\mathrm{dep}(\mathrm{LCA}(i,z))=|\text{Anc}(i)\cap \text{Anc}(z)|$$

其中 $\text{Anc}(x)$ 表示从根到 $x$ 的所有节点集合（含根和 $x$）。

$\text{Anc}(i)\cap\text{Anc}(z)$ 正是一条从根向下的公共前缀链，它的最后一个点就是 $\mathrm{LCA}(i,z)$，长度正是该 LCA 的深度。

---

**结论 $2$**：

对每个 $i$ 的公共祖先个数在按祖先 $u$ 逐个计数时被累加，交换求和顺序即可。
$$\sum_{i=l}^{r}\mathrm{dep}(\mathrm{LCA}(i,z)) =\sum_{i=l}^{r}|\text{Anc}(i)\cap\text{Anc}(z)| =\sum_{u\in \text{Anc}(z)}\#\{\,i\in[l,r]\mid u\in\text{Anc}(i)\,\}.$$  
---
**结论 $3$**：  

$u\in \text{Anc}(i)$ 当且仅当 $i$ 在 $u$ 的子树里。  

$$\sum_{i=l}^{r}\mathrm{dep}(\mathrm{LCA}(i,z)) =\sum_{u\in \text{path}(root,z)} \#\big([l,r]\cap \text{subtree}(u)\big)$$

很显然，这一步把祖先关系改成了子树包含，后面就可以用树的 DFS 序把子树变成一个**连续区间**来计数。可以使用 HLD 一贯的分析方法在纸上画出来。

---

**结论 4**：  

定义：
$$P_x(u)=\#\{\,i\le x\mid i\in \text{subtree}(u)\,\}$$
再令：
$$F_z(x)=\sum_{u\in \text{path}(root,z)} P_x(u).$$
那么：
$$\sum_{i=l}^{r}\mathrm{dep}(\mathrm{LCA}(i,z)) =F_z(r)-F_z(l-1)$$
 
$P_x(u)$ 是在点编号 $\le x$ 的集合里，落在 $u$ 子树内的个数。因此：

$$\sum_{u\in \text{path}(root,z)}\#\big([l,r]\cap \text{subtree}(u)\big) =\sum_{u}\big(P_r(u)-P_{l-1}(u)\big) =F_z(r)-F_z(l-1).$$

---
- 求 $F_z(x)$：**查询路径 $root\to z$** 的当前和；

- 求答案：在 $x=R$ 时取一次 $+F_z(R)$，在 $x=L-1$ 时取一次 $-F_z(L-1)$，相减即可。

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
#define ls o << 1
#define rs o << 1 | 1
using namespace std;
const int N = 1e5 + 5;
const int mod = 201314;
int n, m;
int sz[N], top[N], id[N], idx[N], dfn;
int son[N], fa[N], dep[N], ans[N];
vector<int> g[N];
struct Q{
    int L, R, z;
}q[N];
struct OP{
    int x, z, id, s;
}op[N << 1];
struct Segment_Tree{
    struct Node{int l,r,sum,lazy;}t[N<<2];
    void apply(int o, int v){
        v %= mod; if(v < 0) v += mod;
        t[o].sum  = (t[o].sum + (t[o].r - t[o].l + 1) * v) % mod;
        t[o].lazy = (t[o].lazy + v) % mod;
    }
    void pushup(int o){
        t[o].sum=(t[ls].sum+t[rs].sum)%mod;
    }
    void build(int l,int r,int o){
        t[o].l=l,t[o].r=r;
        if(l==r){
            t[o].sum=0;
            return ;
        }
        int mid=(l+r)>>1;
        build(l,mid,ls);build(mid+1,r,rs);
        pushup(o);
    }
    void pushdown(int o){
        if(t[o].lazy){
            apply(ls, t[o].lazy);
            apply(rs, t[o].lazy);
            t[o].lazy = 0;
        }
    }
    void add(int x,int y,int k,int o){
        if(t[o].r<x||t[o].l>y) return ;
        if(x<=t[o].l&&t[o].r<=y){
            t[o].sum=(t[o].sum + (t[o].r-t[o].l+1)*k)%mod;
            t[o].lazy=(t[o].lazy + k) % mod;
            return ;
        }
        pushdown(o);
        add(x,y,k,ls),add(x,y,k,rs);
        pushup(o);
    }

    int query(int x,int y,int o){
        if(t[o].r<x||t[o].l>y) return 0;
        if(x<=t[o].l&&t[o].r<=y){
            pushdown(o);
            return t[o].sum % mod;
        }
        pushdown(o);
        return (query(x,y,ls)+query(x,y,rs)) % mod;
    }

}ST;


void dfs1(int x, int p){
    sz[x] = 1; son[x] = -1;
    for(const auto &y : g[x]){
        if(y == p) continue;
        fa[y] = x; dep[y] = dep[x] + 1;
        dfs1(y, x);
        sz[x] += sz[y];
        if(son[x] == -1 || sz[y] > sz[son[x]]) son[x] = y;
    }
}

void dfs2(int x, int tp){
    top[x] = tp;
    dfn++, id[x] = dfn;
    idx[dfn] = x;
    if(son[x] != -1) dfs2(son[x], tp);
    for(const auto &y : g[x]){
        if(y == son[x] || y == fa[x]) continue;
        dfs2(y, y);
    }
    return ;
}

void modify(int u, int v, int k){
    k %= mod;
    while(top[u] != top[v]){
        if(dep[top[u]] < dep[top[v]]) swap(u, v);
        ST.add(id[top[u]], id[u], k, 1);
        u = fa[top[u]];
    }
    if(dep[u] < dep[v]) swap(u, v);
    ST.add(id[v], id[u], k, 1);
}

int query(int u, int v){
    int res = 0;
    while(top[u] != top[v]){
        if(dep[top[u]] < dep[top[v]]) swap(u, v);
        res = (res + ST.query(id[top[u]], id[u], 1)) % mod;
        u = fa[top[u]];
    }   
    if(dep[u] < dep[v]) swap(u, v);
    res = (res + ST.query(id[v], id[u], 1) % mod);
    return res;
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    cin >> n >> m;
    for(int i = 1; i < n; ++i){
        int f; cin >> f;
        g[f].push_back(i);
        g[i].push_back(f);
    }
    int tot = 0;
    for(int i = 1; i <= m; ++i){
        cin >> q[i].L >> q[i].R >> q[i].z;
        op[++tot] = {q[i].R, q[i].z, i, 1};
        if(q[i].L > 0) op[++tot] = {q[i].L - 1, q[i].z, i, -1};
    }
    dep[0] = 1;
    dfs1(0, -1), dfs2(0, 0);
    ST.build(1, n, 1);
    sort(op + 1, op + 1 + tot, [](const OP &A, const OP &B){
        return A.x < B.x;
    });
    int cur = -1;
    for(int i = 1; i <= tot; ++i){
        while(cur < op[i].x){++cur, modify(cur, 0, 1);}
        int val = query(op[i].z, 0);
        ans[op[i].id] = (ans[op[i].id] + op[i].s * val) % mod;
        if(ans[op[i].id] < 0) ans[op[i].id] += mod;
    }
    for(int i = 1; i <= m; ++i) cout << ans[i] % mod << '\n';
    return 0;
}
```