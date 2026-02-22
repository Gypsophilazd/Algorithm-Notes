`rating = 1800`
## 题目描述

给定一棵包含 $n$ 个顶点的树。树是一个包含 $n-1$ 条边的连通无向图。树上的每个顶点 $v$ 都被赋予了一种颜色（如果顶点 $v$ 是白色，则 $a_v = 1$，如果顶点 $v$ 是黑色，则 $a_v = 0$）。

你需要对于每个顶点 $v$，解决如下问题：在所有包含顶点 $v$ 的子树中，白色顶点数与黑色顶点数的最大差值是多少？树的子树是原树的一个连通子图。更正式地说，如果你选择的子树包含 $cnt_w$ 个白色顶点和 $cnt_b$ 个黑色顶点，你需要最大化 $cnt_w - cnt_b$。

## 输入格式

输入的第一行包含一个整数 $n$（$2 \le n \le 2 \cdot 10^5$），表示树的顶点数。

第二行包含 $n$ 个整数 $a_1, a_2, \dots, a_n$（$0 \le a_i \le 1$），其中 $a_i$ 表示第 $i$ 个顶点的颜色。

接下来的 $n-1$ 行，每行描述一条树的边。第 $i$ 条边由两个整数 $u_i$ 和 $v_i$ 表示，表示该边连接顶点 $u_i$ 和 $v_i$（$1 \le u_i, v_i \le n, u_i \ne v_i$）。

保证给定的边构成一棵树。

## 输出格式

输出 $n$ 个整数 $res_1, res_2, \dots, res_n$，其中 $res_i$ 表示在所有包含顶点 $i$ 的子树中，白色顶点数与黑色顶点数的最大差值。

## 输入输出样例 #1

### 输入 #1

```
9
0 1 1 1 0 0 0 0 1
1 2
1 3
3 4
3 5
2 6
4 7
6 8
5 9
```

### 输出 #1

```
2 2 2 2 2 1 1 0 2
```

## 输入输出样例 #2

### 输入 #2

```
4
0 0 1 0
1 2
1 3
1 4
```

### 输出 #2

```
0 -1 1 -1
```

## 说明/提示

第一个样例如下图所示：

![](Note/ACM%E8%AE%AD%E7%BB%83/Dynamic_Programming/images/69af08669a0526392387a3f58522c033_MD5.webp)

黑色顶点用粗边框表示。

在第二个样例中，顶点 $2, 3, 4$ 的最优子树分别是顶点 $2, 3, 4$ 本身。顶点 $1$ 的最优子树是包含顶点 $1$ 和 $3$ 的子树。

由 ChatGPT 4.1 翻译

## Solution

题意简述：一棵树，节点有黑有白，从某节点出发，遇到黑 $-1$，遇到白 $+1$，求从每个节点分别出发，得到的最大值是多少。

同样考虑换根 DP。

第一次 DFS：
先假设 $1$ 号节点为根，预处理每个节点仅往子树走的最大解，此时根节点的解就是答案。设 `dp[u]` 表示从 $u$ 走完其所有子树获得的最大值。

第二次 DFS：
以 $u$ 为根转换到以 $v$ 为根，考察 `dp[v]` 的变化：
	（1）$v$ 子树对 $u$ 有贡献（$dp[v] > 0$）：`dp[u]`一定已经加了 `dp[v]`的值。如果 $dp[v]-dp[u] \le 0$，就取 `dp[v]`，否则取 `dp[v] + (dp[u] - dp[v]) = dp[u]`。
	（2）$v$ 子树对 $u$ 没有贡献（$dp[v] \le 0$）：`dp[u]` 一定没加 `dp[v]`的值。如果 $dp[u] \le 0$，则取 `dp[v]`，否则取 `dp[v] + dp[u]`

## Code

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 2e5 + 5;
int a[N], dp[N];
vector<int> g[N];

void dfs1(int u, int fa){
    dp[u]= a[u];
    for(const auto &v : g[u]){
        if(v == fa) continue;
        dfs1(v, u);
        if(dp[v] > 0) dp[u] += dp[v];
    }
}

void dfs2(int u, int fa){
    for(const auto &v : g[u]){
        if(v == fa) continue;
        if(dp[v] > 0) dp[v] = max(dp[u], dp[v]);
        else dp[v] = max(dp[v], dp[u] + dp[v]);
        dfs2(v, u);
    }
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n; cin >> n;
    for(int i = 1; i <= n; ++i){
        cin >> a[i];
        if(!a[i]) a[i] = -1;
    }
    for(int i = 1; i < n; ++i){
        int u, v; cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u);
    }
    dfs1(1, 0); dfs2(1, 0);
    for(int i = 1; i <= n; ++i) cout << dp[i] << " ";
    return 0;
}

```