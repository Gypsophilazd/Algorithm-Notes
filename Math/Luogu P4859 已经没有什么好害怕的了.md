## 题目描述

已经使 Madoka 有签订契约，和自己一起战斗的想法后，Mami 忽然感到自己不再是孤单一人了呢。

于是，之前的谨慎的战斗作风也消失了，在对 Charlotte 的傀儡使用终曲——Tiro Finale 后，Mami 面临着即将被 Charlotte 的本体吃掉的局面。

这时，已经多次面对过 Charlotte 的 Homura 告诉了学 OI 的你这样一个性质：Charlotte 的结界中有两种具有能量的元素，一种是“糖果”，另一种是“药片”，各有 $n$ 个。在 Charlotte 发动进攻前，“糖果”和“药片”会两两配对，若恰好糖果比药片能量大的组数比“药片”比“糖果”能量大的组数多 $k$ 组，则在这种局面下，Charlotte 的攻击会丟失，从而 Mami 仍有消灭 Charlotte 的可能。

你必须根据 Homura 告诉你的“糖果”和“药片”的能量的信息迅速告诉 Homura 这种情况的个数.

## 输入格式

第一行两个整数 $n,k$，含义如题目所描述。

接下来一行 $n$ 个整数，第 $i$ 个数表示第 $i$ 个糖果的能量。

接下来 $n$ 个整数，第 $j$ 个数表示第 $j$ 个药片的能量。

保证上面两行不会有重复的数字。

## 输出格式

一个答案，表示消灭 Charlotte 的情况个数，需要对 $10^9+9$ 取模。

## 输入输出样例 #1

### 输入 #1

```
4 2
5 35 15 45
40 20 10 30
```

### 输出 #1

```
4
```

## 说明/提示

【样例解释】

正确的组合为：

(5-40,35-20,15-10,45-30)，    
(5-40,45-20,15-10,35-30)，   
(45-40,5-20,15-10,35-30)，   
(45-40,35-20,15-10,5-30).   

【数据范围】
对于 $100\%$ 的数据，$1 \le n \le 2000$，$0 \le k \le n$。

## Solution

题意简述：给定 $n, k$ 与包含 $n$ 个数的两个数组 $a, b$，对于 $i,j∈[1, n]$，设 $a_i > b_j$ 的组数为 $cnt_a$，$a_i \le b_j$ 的组数为 $cnt_b$， 求恰好使得 $cnt_a - cnt_b = k$ 的 $a_i, b_j$ 配对情况数。

### 思路

注意到数组的顺序对解决问题没有影响，考虑先对数组排序。

其次注意到，**恰好**等于 $k$ 并不好解决，那么我们暂时扩大问题规模，看能不能先解决**至少**等于 $k$。

那么，可以考虑二项式反演。

设 $g_m$ 表示 $n$ 个数中，**恰好**有 $m$ 对 $a > b$ 的方案数。

设 $f_m$ 表示 $n$ 个数中，**至少**有 $m$ 对 $a > b$ 的方案数。

$$f_m = \sum_{i=m}^{n} \binom{i}{m} g_i$$
$$g_m = \sum_{i=m}^{n}(-1)^{i-m} \binom{i}{m}f_i$$
举例：
$a: 2\space 4\space 6$
$b: 1\space 3\space 5$
$g_2:\{(2,1)\space (6,3)\space (4,5)\space\}\{(4,1)\space (6,3)\space (2,5)\space\}\{(4,3)\space (6,1)\space (2,5)\space\}\{(6,5)\space (4,1)\space (2,3)\space\}$
$g_3:\{(2,1)\space (4,3)\space (6,5)\space\}\{(2,1)\space (6,5)\space (4,3)\space\}\{(4,3)\space (6,5)\space (2,1)\space\}$
$$f_2=\binom{2}{2}g_2+\binom{3}{2}g_3=7$$
接下来就可以考虑如何求出 $f_m$。$a, b$ 数组都已排好序，考虑动态规划，设 $F_{i,j}$ 表示从 $a$ 中前 $i$ 个数，选了 $j$ 个 $a > b$ 的方案数。

(1) 不考虑 $a_i$，前 $i - 1$ 个数选 $j$ 个，即 $F_{i-1,j}$。

(2) 考虑 $a_i$，前 $i - i$ 个数选 $j - 1$ 个，即 $F_{i-1,j-1}$，对于其中没中方案，$a_i$ 都可以和 $b$ 中 $(p-j+1)$ 个空位配对，$p$ 表示在 $b$ 中 $a_i>b$ 的个数。

$$F_{i, j} = F_{i-1, j} + (p-j+1)F_{i-1,j-1}$$
$F_{n,m}$ 表示选了 $n$ 个数中有 $m$ 对 $a > b$，那么剩下的 $(n-m)$ 对可以任意配对，即全排列。故：$$f_m=(n-m)!\space F_{n,m}$$
## Code
```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 2e3 + 5;
const int mod = 1e9 + 9;
int a[N], b[N], C[N][N], dp[N][N];
int f[N], g[N];

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n, k; cin >> n >> k;
    for(int i = 1; i <= n; ++i) cin >> a[i];
    for(int i = 1; i <= n; ++i) cin >> b[i];
    for(int i = 0; i <= n; ++i) C[i][0] = 1, dp[i][0] = 1;
    for(int i = 1; i <= n; ++i){
        for(int j = 1; j <= i; ++j) C[i][j] = (C[i - 1][j] + C[i - 1][j - 1]) % mod;
    }
    sort(a + 1, a + 1 + n); sort(b + 1, b + 1 + n);
    for(int i = 1, p = 0; i <= n; ++i){
        while(p + 1 <= n && b[p + 1] < a[i]) p++;
        for(int j = 1; j <= n; ++j) dp[i][j] = (dp[i - 1][j] + dp[i - 1][j - 1] * (p - j + 1) % mod) % mod;
    }
    int s = 1, ans = 0, x = (n + k) / 2;
    for(int j = n; j >= 1; --j){
        f[j] = dp[n][j] * s % mod;
        s = s * (n - j + 1) % mod;
    }
    for(int i = x; i <= n; ++i) ans = ((ans + ((i - x) & 1 ? -1ll : 1ll) * C[i][x] * f[i] % mod) + mod) % mod;
    cout << ans << endl;
    return 0;
}
```