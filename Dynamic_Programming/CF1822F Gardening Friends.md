`rating = 1700`
## 题目描述

#### 题意翻译
有一棵有 $n$ 个节点的树，根节点为节点 $1$，每条边的权值为 $k$。现在可以进行挪根操作，每次耗费 $c$ 价值，将树的根转移到与原来根结点相邻的点上。

定义这棵树的价值为根节点到子节点的最远距离 $a$ 与挪根耗费总价值 $b$ 之差（可能不会挪根）。求这棵树经过挪根操作后的最大价值

## 输入格式

第一行一个正整数 $t$（$1\le t\le10^4$）为数据组数。

每组数据的第一行包括整数 $n$，$k$，$c$（$2\le n\le2\cdot10^5$，$1\le k,c\le10^9$），分别为节点数、边权和操作耗费价值。

接下来 $n-1$ 行每行描述一条边，节点为 $u_i,v_i$（$ 1\le u_i,v_i\le n$）。

## 输出格式

一行一个正整数表示最大价值。

## 输入输出样例 #1

### 输入 #1

```
4
3 2 3
2 1
3 1
5 4 1
2 1
4 2
5 4
3 4
6 5 3
4 1
6 1
2 6
5 1
3 2
10 6 4
1 3
1 9
9 7
7 6
6 4
9 2
2 8
8 5
5 10
```

### 输出 #1

```
2
12
17
32
```

## Solution

考虑换根 DP。需要对每个节点 $u$ 维护三个极其重要的核心数据：

1. **`down1[u]`**：从 $u$ 往下走，能到达的**最远**距离。
2. **`down2[u]`**：从 $u$ 往下走，能到达的**次远**距离。（次远距离必须来自与最远距离**不同的子树分支**）。
3. **`up[u]`**：从 $u$ 往**上**走（经过它的父节点），能到达的最远距离。

第一次 DFS：
自底向上算 `down1` 和 `down2`。就像普通的树形 DP，合并子节点的 `down1[v] + 1`，去更新 $u$ 的最大值和次大值。

第二次 DFS：
自顶向下算 `up`，现在要把根从父节点 $u$ 换到子节点 $v$。
$v$ 往上走的最远距离 `up[v]` 怎么算？它有两个选择：

- 往上走到 $u$ 之后，继续**往上**走 $\rightarrow$ 对应 `up[u]`。
- 往上走到 $u$ 之后，往下拐进 $u$ 的**其他子树** $\rightarrow$ 对应 $u$ 的向下最远距离。

**这里的坑点来了：**
如果拐进的其他子树，恰好就是 $v$ 自己所在的这棵子树呢？那就相当于走回头路。

- 所以如果 $v$ 所在的子树恰好贡献了 $u$ 的最远距离 `down1[u]`，那么 $v$ 往上走就只能借用 $u$ 的**次远距离** `down2[u]`。  
- 如果 $v$ 不在 $u$ 的最远分支上，那 $v$ 就可以放心地借用 $u$ 的**最远距离** `down1[u]`。

最终，对于任意一个节点 $i$，它作为整棵树的根时，最远距离就是 $\max(down1[i], up[i])$。
这道题的最终答案，就是去遍历所有的 $i$，计算：
$$Ans = \max(Ans, \quad k \times \max(down1[i], up[i]) - c \times dep[i])$$
(注意：$dep[i]$ 是节点 $i$ 到原始节点 1 的真实距离，在 `dfs1` 里顺手求出来即可)

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 2e5 + 5;
int n, k, c;
int down1[N], down2[N], mx[N], up[N], dep[N];
vector<int> g[N];
void dfs1(int u, int fa){
    down1[u] = down2[u] = mx[u] = 0;
    dep[u] = dep[fa] + 1;
    for(const auto &v : g[u]){
        if(v == fa) continue;
        dfs1(v, u);
        int val = down1[v] + 1;
        if(val > down1[u]) down2[u] = down1[u], down1[u] = val, mx[u] = v;
        else if(val > down2[u]) down2[u] = val;
    }
}

void dfs2(int u, int fa){
    for(const auto &v : g[u]){
        if(v == fa) continue;
        up[v] = 1;
        if(mx[u] == v) up[v] += down2[u];
        else up[v] += down1[u];
        up[v] = max(up[u] + 1, up[v]);
        dfs2(v, u);
    }
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int t; cin >> t;
    while(t--){
        cin >> n >> k >> c;
        for(int i = 1; i <= n; ++i){
            dep[i] = 0; g[i].clear();
        }
        for(int i = 1; i < n; ++i){
            int u, v; cin >> u >> v;
            g[u].push_back(v);
            g[v].push_back(u);
        }
        dep[0] = -1, up[1] = 0;
        dfs1(1, 0); dfs2(1, 0);
        int ans = 0;
        for(int i = 1; i <= n; ++i) ans = max(ans, k * max(down1[i], up[i]) - c * dep[i]);
        cout << ans << '\n';
    }
    return 0;
}
```