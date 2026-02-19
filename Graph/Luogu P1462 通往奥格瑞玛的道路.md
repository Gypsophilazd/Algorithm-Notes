## 题目背景

在艾泽拉斯大陆上有一位名叫歪嘴哦的神奇术士，他是部落的中坚力量。

有一天他醒来后发现自己居然到了联盟的主城暴风城。

在被众多联盟的士兵攻击后，他决定逃回自己的家乡奥格瑞玛。

## 题目描述

在艾泽拉斯，有 $n$ 个城市。编号为 $1,2,3,\ldots,n$。

城市之间有 $m$ 条双向的公路，连接着两个城市，从某个城市到另一个城市，会遭到联盟的攻击，进而损失一定的血量。

每次经过一个城市，都会被收取一定的过路费（包括起点和终点）。路上并没有收费站。

假设 $1$ 为暴风城，$n$ 为奥格瑞玛，而他的血量最多为 $b$，出发时他的血量是满的。如果他的血量降低至负数，则他就无法到达奥格瑞玛。

歪嘴哦不希望花很多钱，他想知道，在所有可以到达奥格瑞玛的道路中，对于每条道路所经过的城市收费的最大值，最小值为多少。

## 输入格式

第一行 $3$ 个正整数，$n,m,b$。分别表示有 $n$ 个城市，$m$ 条公路，歪嘴哦的血量为 $b$。

接下来有 $n$ 行，每行 $1$ 个正整数，$f_i$。表示经过城市 $i$，需要交费 $f_i$ 元。

再接下来有 $m$ 行，每行 $3$ 个正整数，$a_i,b_i,c_i$（$1\leq a_i,b_i\leq n$）。表示城市 $a_i$ 和城市 $b_i$ 之间有一条公路，如果从城市 $a_i$ 到城市 $b_i$，或者从城市 $b_i$ 到城市 $a_i$，会损失 $c_i$ 的血量。

## 输出格式

仅一个整数，表示歪嘴哦交费最多的一次的最小值。

如果他无法到达奥格瑞玛，输出 `AFK`。

## 输入输出样例 #1

### 输入 #1

```
4 4 8
8
5
6
10
2 1 2
2 4 1
1 3 4
3 4 3
```

### 输出 #1

```
10
```

## 说明/提示

对于 $60\%$ 的数据，满足 $n\leq 200$，$m\leq 10^4$，$b\leq 200$；

对于 $100\%$ 的数据，满足 $1\leq n\leq 10^4$，$1\leq m\leq 5\times 10^4$，$1\leq b\leq 10^9$；

对于 $100\%$ 的数据，满足 $1\leq c_i\leq 10^9$，$0\leq f_i\leq 10^9$，可能有两条边连接着相同的城市。

### Solution
图论中,二分+最短路的最经典题目之一.
总结来说就是有n个点,m条无向边,要求从1号点到n号点,初始血量为b
然后给定点权(费用)$fi$, 给定点a与b链接的边,以及边权$ci$ 需要先判定最短路的血量代价是否大于b,是则输出"AFK", 否则求出费用最大的最小值--二分判定条件

那么 思路便存在了
1. 我们打算二分答案,即二分城市所有需要缴交的费用,二分的左边界left = max(f\[1], f\[n]) 即起点与终点必须包含, 右边界为当前枚举路径中费用的最大值即right = max(f\[1, 2, ..., n]);
2. 我们先建图,以Edge作为结构体,把当前边所包含两点中费用最大值塞进Edge中,随后进行二分,二分过程check的条件为从1~n, 经过属于当前f\[mid]的点,使用dijkstra最短路算法检验其血量剩余是否大于0即可.
3. 值得注意的是,每一次二分需要重新构建子图,重新遍历图,如果当前图中有更大的f,我们就跳过他(因为求最小值),然后把费用小于等于mid的点塞进子图做最短路即可.
### AC Code
```
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 1e4 + 5;
const int INF = 1e18;
int n, m, b, f[N];
struct Edge {
    int u, v, w, max_f;
    Edge(int u = 0, int v = 0, int w = 0, int f1 = 0, int f2 = 0)
        : u(u), v(v), w(w), max_f(max(f1, f2)) {}
};
vector<pair<int, int>> g[N];
vector<Edge> edges;
bool check(int mid) {
    if (f[1] > mid || f[n] > mid) return false;
    for (int i = 1; i <= n; ++i) g[i].clear();
    for (auto &e : edges) {
        if (e.max_f > mid) continue;
        g[e.u].emplace_back(e.v, e.w);
        g[e.v].emplace_back(e.u, e.w);
    }
    vector<int> dist(n + 1, INF);
    bitset<N> vis;
    using pii = pair<int, int>;
    priority_queue<pii, vector<pii>, greater<>> pq;//大顶堆
    pq.emplace(0,  1);
    dist[1] = 0;
    while (!pq.empty())  {
        auto [dis, u] = pq.top();  pq.pop();
        if (vis[u]) continue;
        vis[u] = true;
        for (auto &[v, w] : g[u]) { //结构化绑定 可以改C++11
            if (dist[v] > dist[u] + w) {
                dist[v] = dist[u] + w;
                pq.emplace(dist[v], v);
            }
        }
    }
    return dist[n] <= b;
}
signed main() {
    ios::sync_with_stdio(0), cin.tie(0),  cout.tie(0);
    cin >> n >> m >> b;
    for (int i = 1; i <= n; ++i) cin >> f[i];
    for (int i = 1; i <= m; ++i) {
        int u, v, w; cin >> u >> v >> w;
        edges.emplace_back(u, v, w, f[u], f[v]);
    }
    int l = max(f[1], f[n]);
    int r = *max_element(f + 1, f + n + 1);
    int ans = -1;
    while (l <= r) {
        int mid = (l + r) >> 1;
        if (check(mid)) ans = mid, r = mid - 1;
        else l = mid + 1;
    }
    cout << (ans == -1 ? "AFK" : to_string(ans)) << '\n';
    return 0;
}
```
#### 总时间复杂度为$O((m + nlogn)log(fmax - fmin))$ 