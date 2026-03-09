## 题目背景

萌萌买了一颗字符串树的种子，春天种下去以后夏天就能长出一棵很大的字符串树。字符串树很奇特，树枝上都密密麻麻写满了字符串，看上去很复杂的样
子。

## 题目描述

字符串树本质上还是一棵树，即 $N$ 个节点 $N-1$ 条边的连通无向无环图，节点从 $1$ 到 $N$ 编号。与普通的树不同的是，树上的每条边都对应了一个字符串。萌萌和 JYY 在树下玩的时候，萌萌决定考一考 JYY。每次萌萌都写出一个字符串 $S$ 和两个节点 $U,V$，JYY 需要立即回答 $U$ 和 $V$ 之间的最短路径（即 $U,V$ 之间边数最少的路径，由于给定的是一棵树，这样的路径是唯一的）上有多少个字符串以 $S$ 为前缀。

JYY 虽然精通编程，但对字符串处理却不在行。所以他请你帮他解决萌萌的难题。

## 输入格式

输入第一行包含一个整数 $N$，代表字符串树的节点数量。

接下来 $N-1$ 行，每行先是两个数 $U,V$，然后是一个字符串 $S$，表示节点 $U$ 和节点 $V$ 之间有一条直接相连的边，这条边上的字符串是 $S$。输入数据保证给出的是一棵合法的树。

接下来一行包含一个整数 $Q$，表示萌萌的问题数。

接下来 $Q$ 行，每行先是两个数 $U,V$，然后是一个字符串 $S$，表示萌萌的一个问题是节点 $U$ 和节点 $V$ 之间的最短路径上有多少字符串以 $S$ 为前缀。

## 输出格式

输出 $Q$ 行，每行对应萌萌的一个问题的答案。

## 输入输出样例 #1

### 输入 #1

```
4
1 2 ab
2 4 ac
1 3 bc
3
1 4 a
3 4 b
3 2 ab
```

### 输出 #1

```
2
1
1
```

## 说明/提示

对于 $100\%$ 的数据，$1\leq N,Q\leq 10^5$，输入所有字符串长度不超过 $10$ 且只包含 `a~z` 的小写字母。

## Solution

本题的思维含金量非常高。题意简述下来就是，给定一棵树，树上的每条边的权值是一个字符串，有 $t$ 次查询，每次查询指定两个节点 $u, v$，和一个字符串 $s$，要求出这两个节点 **最短路径** 之间有几个以 $s$ 为 **前缀** 的字符串。

那么查询字符串，而且是树上的字符串问题，一开始的思路是去套 Trie 的模版，但是很快会发现几个问题：
	1. 常规的 Trie 模板中，一般是直接给出字符串，然后求出这个字符串的存在性问题，我们一般会用后缀去判断。而这次是给出 $u, v$ 节点连边，要求的是前缀。
	2. 考虑每次询问遍历一次 Trie 树，必然会出现 $O(NQ)$ 的时间复杂度，不可接受。

针对问题一，需要跳出模板思路，常规建树，然后第一个 trick 是 *把字符串当做边权*。出现树上两点简单路径且要求满足条件的（前缀）计数问题，条件反射的会有 *LCA与树上差分问题*。处理此类字符串问题，依旧需要 Trie 树，但是联动问题二，就可以想到我们必然需要进行第二个 trick，即 *离线询问，离线修改*。如何对时间复杂度进行降阶呢？这个时候就可以考虑第三个 trick，即 *树上差分*。

这里的离线问题不同以往，我们不能让查询去等树的状态，而是需要让查询变成挂载在树节点上的中断事件。我们假设：
1. 萌萌提出了一个问题（原版）：`id = 1, u = 4, v = 5, s = "ab"`
2. 算出 4 和 5 的 LCA 假设是节点 2。
3. 根据差分公式 `ans[i] = query(u) + query(v) - 2 * query(lca(u, v))`，这个问题可以被**拆碎，变成三个独立的中断请求**，扔进对应节点的请求结构体中：

    - 节点 4 请求体新增：`{属于问题 1, 查 "ab", 乘 1}`
        
    - 节点 5 请求体新增：`{属于问题 1, 查 "ab", 乘 1}`
        
    - 节点 2 请求体新增：`{属于问题 1, 查 "ab", 乘 -2}`

离线操作以后，具体如何处理这些操作？这个时候就可以用到 Trie 树，我们不再根据 `id` 轮询询问，而是直接对一整棵树进行 DFS，每次 DFS，让所在节点存在询问的地方，遍历询问的 `id`，进行累加，而这个时候我们的 `Q` 结构体中的符号位就派上用场，只要符号位设定好为 $-2$ 或者 $1$，代表是 LCA 或者节点，就可以直接乘上询问到的答案数量累加到该 `id` 下的询问！

最后一个细节上的问题就是，我们如何在 Trie 树上统计包含这个前缀的字符串数量？
很简单，但是这个 trick 也很巧，我们直接把原来在 `trie_insert()` 中，**for 循环外部原始为了精准匹配的`cnt[p] += val` 放到循环内部**，删除则把 `val` 换成 $-1$ 即可以下是其正确性说明：

当你插入 "abc" 时：

1. **第一步（i=0）**：到了代表 'a' 的节点。你立刻给这个节点加 1（`cnt[p] += val`）。
    - **意义**：路过这里的字符串多了一个，也就是**以 "a" 为前缀的字符串数量 + 1**。
2. **第二步（i=1）**：往下踩到了 'b' 的节点。你给这个节点加 1。
    - **意义**：路过这里的字符串多了一个，也就是**以 "ab" 为前缀的字符串数量 + 1**。
3. **第三步（i=2）**：踩到 'c'。给它加 1。
    - **意义**：**以 "abc" 为前缀的字符串数量 + 1**。

至于求LCA的方法有很多。这里不再赘述。

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 1e5 + 5;
const int M = 1e6 + 5;
int fa[N][21], dep[N], ch[M][26], idx;
int ans[N], cnt[M];

struct Edge{
    int to;
    string s;
};

struct Q{
    int id;
    string s;
    int flag;
};

vector<Edge> es[N];
vector<Q> q[N];

void dfs1(int u, int f){
    dep[u] = dep[f] + 1;
    fa[u][0] = f;
    for(int i = 1; i <= 20; ++i) fa[u][i] = fa[fa[u][i - 1]][i - 1];
    for(const auto &g : es[u]){
        int v = g.to;
        if(v == f) continue;
        dfs1(v, u);
    }
}

int lca(int x, int y){
    if(dep[x] < dep[y]) swap(x, y);
    for(int i = 19; i >= 0; --i){
        if(dep[fa[x][i]] >= dep[y]) x = fa[x][i];
    }
    if(x == y) return x;
    for(int i = 19; i >= 0; --i){
        if(fa[x][i] != fa[y][i]) x = fa[x][i], y = fa[y][i];
    }
    return fa[x][0];
}

void trie_insert(string s, int sign){
    int p = 0;
    for(int i = 0; s[i]; ++i){
        int j = s[i] - 'a';
        if(!ch[p][j]) ch[p][j] = ++idx;
        p = ch[p][j];
        cnt[p] += sign;
    }
}

int trie_output(string s){
    int p = 0;
    for(int i = 0; s[i]; ++i){
        int j = s[i] - 'a';
        if(!ch[p][j]) return 0;
        p = ch[p][j];
    }
    return cnt[p];
}

void dfs2(int u, int fa){
    for(const auto &qi : q[u]){
        ans[qi.id] += qi.flag * trie_output(qi.s);
    }
    for(const auto &g : es[u]){
        int v = g.to;
        string s = g.s;
        if(v == fa) continue;
        trie_insert(s, 1);
        dfs2(v, u);
        trie_insert(s, -1);
    }
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n; cin >> n;
    for(int i = 1; i < n; ++i){
        int u, v; cin >> u >> v;
        string s; cin >> s;
        es[u].push_back({v, s});
        es[v].push_back({u, s});
    }
    dfs1(1, 0);
    int t; cin >> t;
    for(int i = 1; i <= t; ++i){
        int u, v; cin >> u >> v;
        string s; cin >> s;
        int LCA = lca(u, v);
        q[u].push_back({i, s, 1});
        q[v].push_back({i, s, 1});
        q[LCA].push_back({i, s, -2});
    }
    dfs2(1, 0);
    for(int i = 1; i <= t; ++i) cout << ans[i] << "\n";
    return 0;
}
```