`rating = 2100`
## 题目描述

给定一棵包含 $n$ 个顶点的树（无向连通无环图）。你要在这棵树上玩一个游戏。

最开始所有顶点都是白色的。在游戏的第一回合，你选择一个顶点并将其染成黑色。之后的每一回合，你都可以选择一个与任意黑色顶点相邻的白色顶点，并将其染成黑色。

每当你选择一个顶点（包括第一回合），你获得的分数等于包含该顶点的、仅由白色顶点组成的连通块的大小。游戏在所有顶点都被染成黑色时结束。

来看下面这个例子：

![](Note/ACM%E8%AE%AD%E7%BB%83/Dynamic_Programming/images/db03f887791ac6dd2466978bc5517fc6_MD5.webp)

顶点 $1$ 和 $4$ 已经被染成黑色。如果你选择顶点 $2$，你将获得 $4$ 分，因为包含顶点 $2, 3, 5, 6$ 的连通块的大小为 $4$。如果你选择顶点 $9$，你将获得 $3$ 分，因为包含顶点 $7, 8, 9$ 的连通块的大小为 $3$。

你的任务是最大化你获得的分数。

## 输入格式

第一行包含一个整数 $n$，表示树的顶点数（$2 \le n \le 2 \times 10^5$）。

接下来的 $n-1$ 行，每行描述树的一条边。第 $i$ 条边由两个整数 $u_i$ 和 $v_i$ 表示，表示该边连接顶点 $u_i$ 和 $v_i$（$1 \le u_i, v_i \le n$，$u_i \ne v_i$）。

保证给定的边构成一棵树。

## 输出格式

输出一个整数，表示如果你采取最优策略，最多可以获得多少分。

## 输入输出样例 #1

### 输入 #1

```
9
1 2
2 3
2 5
2 6
1 4
4 9
9 7
9 8

```

### 输出 #1

```
36

```

## 输入输出样例 #2

### 输入 #2

```
5
1 2
1 3
2 4
2 5

```

### 输出 #2

```
14

```

## 说明/提示

第一个示例的树如题目描述中的图片所示。

由 ChatGPT 4.1 翻译


## Solution

确定某点为根后，依次染色，累计个子树的大小，即得以该点为根的答案，如果以每个点为根，跑树形 DP，$O(n^2)$ 会超时，考虑换根 DP。

第一次 DFS：
用 `sz[u]` 表示以 $u$ 为根的子树大小。`dp[1]` 表示以 $1$ 为根获得的总点数。

第二次 DFS：
当前已经推出以 $u$ 为根时获得的总点数 `dp[u]`，换成以 $v$ 为根的时候，$v$ 子树的节点已不属于 $u$，必然需要先减掉`sz[v]`；同时，$v$ 子树以外的节点变成 $v$ 的子树，必然要加上 `n - sz[v]`，$v$ 子树内的贡献以及 $u$ 的其他子树的贡献保持不变
故转移方程:$$dp[v] = dp[u] - sz[v] + n - sz[v]$$

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 2e5 + 5;
int dp[N], sz[N], ans = 0, n;
vector<int> g[N];

void dfs1(int u, int fa){
    sz[u] = 1;
    for(const auto &v : g[u]){
        if(v == fa) continue;
        dfs1(v, u);
        sz[u] += sz[v];
    }
    dp[1] += sz[u];
}

void dfs2(int u, int fa){
    for(const auto &v : g[u]){
        if(v == fa) continue;
        dp[v] = dp[u] + n - 2 * sz[v]; 
        dfs2(v, u);
    }
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    cin >> n;
    for(int i = 1; i < n; ++i){
        int u, v; cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u);
    }
    dfs1(1, 0); dfs2(1, 0);
    for(int i = 1; i <= n; ++i) ans = max(dp[i], ans);
    cout << ans << endl;
    return 0;
}
```