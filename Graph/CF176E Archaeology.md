`rating = 3100`
## 题目描述

This time you should help a team of researchers on an island in the Pacific Ocean. They research the culture of the ancient tribes that used to inhabit the island many years ago.

Overall they've dug out $n$ villages. Some pairs of villages were connected by roads. People could go on the roads in both directions. Overall there were exactly $n-1$ roads, and from any village one could get to any other one.

The tribes were not peaceful and they had many wars. As a result of the wars, some villages were destroyed completely. During more peaceful years some of the villages were restored.

At each moment of time people used only those roads that belonged to some shortest way between two villages that existed at the given moment. In other words, people used the minimum subset of roads in such a way, that it was possible to get from any existing village to any other existing one. Note that throughout the island's whole history, there existed exactly $n-1$ roads that have been found by the researchers. There never were any other roads.

The researchers think that observing the total sum of used roads’ lengths at different moments of time can help to better understand the tribes' culture and answer several historical questions.

You will be given the full history of the tribes' existence. Your task is to determine the total length of used roads at some moments of time.

## 输入格式

The first line contains an integer $n$ ( $1\le n \le 10^{5}$ ) — the number of villages. The next $n-1$ lines describe the roads. The $i$ -th of these lines contains three integers $a_{i}$ , $b_{i}$ and $c_{i}$ ( $1 \le a_{i},b_{i} \le n$ , $a_{i}≠b_{i}$ , $1 \le c_{i} \le 10^{9}$ , $1 \le i \le n$ ) — the numbers of villages that are connected by the $i$ -th road and the road's length. The numbers in the lines are separated by a space.

The next line contains an integer $q$ ( $1 \le q \le 10^{5}$ ) — the number of queries. Then follow $q$ queries, one per line, ordered by time. Each query belongs to one of three types:

- "+  $x$ " — village number $x$ is restored ( $1 \le x \le n$ ).
- "-   " — village number  is destroyed ( $1 \le x \le n$ ).
- "?" — the archaeologists want to know the total length of the roads which were used for that time period.

It is guaranteed that the queries do not contradict each other, that is, there won't be queries to destroy non-existing villages or restore the already existing ones. It is guaranteed that we have at least one query of type "?". It is also guaranteed that one can get from any village to any other one by the given roads.

At the initial moment of time no village is considered to exist.

## 输出格式

For each query of type "?" print the total length of used roads on a single line. You should print the answers to the queries in the order, in which they are given in the input.

Please do not use the %lld specifier to read or write 64-bit integers in С++. It is preferred to use cin, cout streams or the %I64d specifier.

## 输入输出样例 #1

### 输入 #1

```
6
1 2 1
1 3 5
4 1 7
4 5 3
6 4 2
10
+ 3
+ 1
?
+ 6
?
+ 5
?
- 6
- 3
?
```

### 输出 #1

```
5
14
17
10
```

## Solution

题意简述：一棵 n 个节点的树，边有边权。每个点可能是关键点，每次操作改变一个点是否是关键点。每次改变后求所有关键点形成的极小连通子树的边权和的两倍。

先考虑求树上两点距离，设某点 $u$ 到根节点的距离为 $dis(u)$（可以很方便的使用 `dfs` 预处理），那么很直观的可以得出树上两点距离 $dist(u,v)$ 的计算方法为：$$dist(u,v)=dis(u)+dis(v) - 2dis(lca(u,v))$$
![[Pasted image 20250909113729.png]]

`lca` 可以使用重链剖分速通。

接下来考虑各关键点的最小连通子树的总距离和算法。

稍微画画图，模拟一下可以想到使用 `dfn`，即将各关键点的 `dfs` 序组成一个集合（成环） $\{a_1,a_2,...,a_k,a_1\}$，那么我们所求的和即为 $$\operatorname{Cycle\_sum} = dist(a_1, a_2) + dist(a_2, a_3) +...+dis(a_k, a_1)$$
接下来考虑如何动态维护这个序列，我们有两种运算，即增加点和删除点，每次需要维护$\operatorname{Cycle\_sum}$。使用 `splay` 维护前驱和后继固然可行，但是码量过高易错，故使用 `set` 维护。

考虑每次增加点和删除点对答案的影响。

当我们新增一个点 $u$，求出其前驱和后继分别为 $u_l, u_r$，那么对答案的贡献将是：$$\operatorname{Cycle\_sum} += dist(u,u_l)+dist(u,u_r)-dist(u_l,u_r)$$
当我们删除一个点 $u$，求出其前驱和后继分别为 $u_l, u_r$，那么对答案的贡献将是：$$\operatorname{Cycle\_sum} -= dist(u,u_l)+dist(u,u_r)-dist(u_l,u_r)$$
求前驱和后继的操作在 `set` 上 `find()` 即可。注意成环。

重链剖分预处理时间复杂度为 $O(n)$，单次求 $dist(u, v)$ 使用跳链操作，复杂度 $O(log\ n)$，`set` 上操作 $O(log\ n)$，每次对答案修改为 $O(1)$，总时间复杂度可以到 $O(n + m\ log\ n)$ 

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 1e6 + 5;
vector<pair<int, int>> g[N];
int sz[N], dep[N], son[N], fa[N], top[N], id[N], idx[N], dfn, R[N], L[N];
int dis[N];
set<int> st;
int csum = 0;

void dfs1(int x, int f){
    fa[x] = f, sz[x] = 1;
    son[x] = -1, dep[x] = dep[f] + 1;
    for(const auto &y : g[x]){
        if(y.first == f) continue;
        dis[y.first] = dis[x] + y.second;
        dfs1(y.first, x);
        sz[x] += sz[y.first];
        if(son[x] == -1 || sz[y.first] > sz[son[x]]) son[x] = y.first;
    }
}

void dfs2(int x, int tp){
    top[x] = tp;
    id[x] = ++dfn, idx[dfn] = x, L[x] = id[x];
    if(son[x] != -1) dfs2(son[x], tp);
    for(const auto &y : g[x]){
        if(y.first == son[x] || y.first == fa[x]) continue;
        dfs2(y.first, y.first);
    }
    R[x] = dfn;
}

int lca(int x, int y){
    while(top[x] != top[y]){
        if(dep[top[x]] < dep[top[y]]) swap(x, y);
        x = fa[top[x]];
    }
    return dep[x] < dep[y] ? x : y;
}   

int dist(int x, int y){
    int f = lca(x, y);
    return dis[x] + dis[y] - 2ll * dis[f];
}

set<int>::iterator predeccessor(set<int>::iterator it){
    if(it == st.begin()) it = st.end();
    --it;
    return it;
}

set<int>::iterator successor(set<int>::iterator it){
    ++it;
    if(it == st.end()) it = st.begin();
    return it;
}

void add(int u){
    int k = id[u];
    if(st.empty()){
        st.insert(k); return ;
    }
    auto it = st.lower_bound(k);
    auto itr = (it == st.end() ? st.begin() : it);
    auto itl = (it == st.begin() ? predeccessor(st.end()) : predeccessor(it));
    int vl = idx[*itl], vr = idx[*itr];
    csum += dist(vl, u) + dist(u, vr) - dist(vl, vr);
    st.insert(k);
}

void del(int u){
    int k = id[u];
    if(st.size() <= 1){
        st.clear(); csum = 0; return ;
    }
    auto it = st.find(k);
    auto itl = (it == st.begin() ? predeccessor(st.end()) : predeccessor(it));
    auto itr = next(it);
    if(itr == st.end()) itr = st.begin();
    int vl = idx[*itl], vr = idx[*itr];
    csum -= dist(vl, u) + dist(u, vr) - dist(vl, vr);
    st.erase(it);
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n, m; cin >> n;
    for(int i = 1; i < n; ++i){
        int x, y, z; cin >> x >> y >> z;
        g[x].push_back({y, z}), g[y].push_back({x, z});
    }
    dep[0] = 0;
    dfs1(1, 0), dfs2(1, 1);
    vector<int> c(n + 1, 0);
    cin >> m;
    while(m--){
        char op; cin >> op;
        if(op == '+'){
            int x; cin >> x;
            add(x), c[x] = 1;
        }
        else if(op == '-'){
            int x; cin >> x;
            del(x), c[x] = 0;
        }
        else{
            if(st.size() <= 1) cout << 0 << '\n';
            else cout << csum / 2 << '\n';
        }
    }
    return 0;
}
```