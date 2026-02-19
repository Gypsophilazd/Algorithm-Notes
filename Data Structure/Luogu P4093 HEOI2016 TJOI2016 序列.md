![](images/Pasted image 20250819165000.png)
## 题目描述

佳媛姐姐过生日的时候，她的小伙伴从某宝上买了一个有趣的玩具送给他。

玩具上有一个数列，数列中某些项的值可能会变化，但同一个时刻最多只有一个值发生变化。现在佳媛姐姐已经研究出了所有变化的可能性，她想请教你，能否选出一个子序列，使得在**任意一种变化和原序列**中，这个子序列都是不降的？请你告诉她这个子序列的最长长度即可。

## 输入格式

输入的第一行有两个正整数 $n,m$，分别表示序列的长度和变化的个数。

接下来一行有 $n$ 个整数，表示这个数列原始的状态。

接下来 $m$ 行，每行有 $2$ 个整数 $x,y$，表示数列的第 $x$ 项可以变化成 $y$ 这个值。

## 输出格式

输出一个整数，表示对应的答案。

## 输入输出样例 #1

### 输入 #1

```
3 4 
1 2 3 
1 2 
2 3 
2 1 
3 4
```

### 输出 #1

```
3
```

## 说明/提示

注意：每种变化最多只有一个值发生变化。

在样例输入中，所有的变化是：
```plain
1 2 3
2 2 3
1 3 3
1 1 3
1 2 4
```
选择子序列为原序列，即在任意一种变化中均为不降子序列。

对于 $20\%$ 数据，所有数均为正整数，且小于等于 $300$。

对于 $50\%$ 数据，所有数字均为正整数，且小于等于 $3000$。

对于 $100\%$ 数据，所有数字均为正整数，且小于等于 $10^5$。$1\le x\le n$。


## Solution

题意简述：数据规模增大的带修最长非降子序列长度问题。设原序列 $a$，对每个位置 $i$ 有可选改值集合（最多只改一个位置）。要找一个下标递增的子序列，使它在**原序列**和**任一单点改值的序列**里都不降。

同时也是**三维偏序**问题，考虑转化为二维偏序然后使用 **CDQ** 分治。

CDQ 分治的一般思考方式：把问题离线，同时把**相邻两项**作为稳定性的最小单元。即满足以下偏序条件：
$$\begin{equation}
\begin{cases}
i < j \\
a_i \le a_j \\
y_1 \le a_j & \text{ $ (a_i \to y_1) $ } \\
a_i \le y_2 & \text{ $ (a_j \to y_2) $ }
\end{cases}
\end{equation}$$
合并一下条件，因为带修改，故设 $$mx[i] = \max(a[i], mx[i])$$ $$mn[i] = \min(a[i], mn[i])$$
则二维偏序为$$\begin{equation}
\begin{cases}
mx[i] \le a_j \\
a_i \le mn[j] \\
\end{cases}
\end{equation}$$
设位置 $j$ 合法的所有合法前驱为集合 $pred$，那么$$Pred(j) = \{i | i < j, mx[i]\le a[j], a[i] \le mn[j]\}$$
设 $dp[i]$ 为**在下标 $i$ 的范围内，以 $i$ 为结尾的稳定不降子序列的最大长度**，状态转移方程即为：$$dp[j] = 1 +\max(dp[i])\space [i∈Pred(j)]$$
根据 CDQ 分治的特点，又如本题的状态转移过程必须从前往后，故必须采用中序遍历，先遍历左区间的 $dp$ 值，以此更新跨区间部分，再更新右区间。CDQ 分治本身可以自然处理时间戳 $i < j$ 的问题，那么考虑第一个约束条件用排序+双指针筛掉，第二个约束条件前缀最大值使用树状数组（BIT）解决。

记住：**左半更新，右半查询**。

CDQ 分治还有一个特点，在类似于归并排序的过程中，同为子区间的左右区间中，左区间用于更新，而右区间用于查询，这是基于**从下标有序到值有序的过程**。

具体操作十分灵活，在代码中给出注释。

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int inf = 1e18;
const int N = 2e5 + 5;
int a[N], mx[N], mn[N], dp[N], id[N];
int ans = 0;
int n, m, lim; 
vector<int> vis;

int lowbit(int x){return x & -x;}
int t[N];
void update(int k, int x){
    for(int i = k; i <= lim; i += lowbit(i)){
        if(t[i] < x) t[i] = x;
        vis.push_back(i);
    }
}

int query(int k){
    int res = 0;
    for(int i = k; i > 0; i -= lowbit(i)) res = max(res, t[i]);
    return res;
}

void Clear(){
    for(int i : vis) t[i] = 0;
    vis.clear();
}

void CDQ(int l, int r){
    if(l == r){
        dp[l] = max(dp[l], 1ll); // 递归出口
        return ;
    }
    int mid = (l + r) >> 1;
    CDQ(l, mid); // 中序遍历，先遍历左子树
    // 处理跨区间
    // 处理下标数组
    for(int i = l; i <= r; ++i) id[i] = i;
    // 归并也可，这里直接按约束条件sort 记住mx[i]<=a[j]
    sort(id + l, id + mid + 1, [&](int i, int j){return mx[i] < mx[j];});
    sort(id + mid + 1, id + r + 1, [&](int i, int j){return a[i] < a[j];});
    // 双指针扫描 这里j < i
    int j = l;
    for(int i = mid + 1; i <= r; ++i){
        while(j <= mid && mx[id[j]] <= a[id[i]]){
            update(a[id[j]], dp[id[j]]);
            ++j;
        }
        // 在已经满足 mx[j] ≤ a[i] 的左点里，找出所有还满足 a[j] ≤ mn[i] 的 j 的 dp[j] 最大值 这里j < i
        dp[id[i]] = max(dp[id[i]], query(mn[id[i]]) + 1);
    }
    Clear();
    CDQ(mid + 1, r);
}   


signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    cin >> n >> m; lim = 0;
    for(int i = 1; i <= n; ++i){
        cin >> a[i]; mx[i] = a[i]; mn[i] = inf;
        lim = max(lim, a[i]);
    }
    for(int i = 1; i <= m; ++i){
        int x, y; cin >> x >> y;
        mx[x] = max(mx[x], y);
        mn[x] = min(mn[x], y);
        lim = max(lim, y);
    }
    for(int i = 1; i <= n; ++i) if(mn[i] == inf) mn[i] = lim;
    CDQ(1, n);
    for(int i = 1; i <= n; ++i) ans = max(ans, dp[i]);
    cout << ans << endl;
    return 0;
}
```