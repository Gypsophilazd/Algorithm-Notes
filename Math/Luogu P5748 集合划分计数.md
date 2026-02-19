![[Pasted image 20250819165000.png]]
## 题目描述

一个有 $n$ 个元素的集合，将其分为任意个非空子集，求方案数。  
注意划分出的集合间是无序的，即 $\{\{1,2\},\{3\}\}$ 和 $\{\{3\},\{2,1\}\}$ 算作一种方案。

由于答案可能会很大，所以要对 $998244353$ 取模。

## 输入格式

第一行一个正整数 $T$，表示数据组数。  
接下来 $T$ 行，每行一个正整数 $n$。

## 输出格式

输出 $T$ 行，每行一个整数表示答案。

## 输入输出样例 #1

### 输入 #1

```
5
2
3
7
9
233
```

### 输出 #1

```
2
5
877
21147
53753544
```

## 说明/提示

【数据范围】  
$T = 1000$，$1\le n \le 10^5$。

【样例解释】  
对于 $n=3$，有五种方案：$\{\{1,2,3\}\},\{\{1,2\},\{3\}\},\{\{1\},\{2,3\}\},\{\{1\},\{2\},\{3\}\},\{\{1,3\},\{2\}\}$。

本题只有一个测试点，假设你答对了 $x$ 组数据，你将得到 $\lfloor x/(T/100) \rfloor$ 分。   
如果你不能解决所有数据，也请输出 $T$ 个整数。

## Solution

题意重述：$n$ 个有标号的球放到无标号的盒子中，每个盒子至少放一个球，不同形态算不同方案，求分配方案数。

### 思路

精炼来说即求 $1e5$ 以内的 $\operatorname{bell}$ 数。设装有 $i$ 个球的盒子有 $f_i$ 种形态，则 $\{f_i\}_{i \ge 1}$ 的常生成函数为 $F(x) = \sum_{i \ge 1}f_i x^i$ 指数生成函数为 $E(x) = \sum_{i \ge 1}f_i \frac{x^i}{i!}$。

先给出结论，不限盒子数量的情况下，$\operatorname{bell}$ 数的公式如下（注意盒子非空需要减一）：$$\operatorname{bell}(x)=e^{E(x)-1}=\exp(E(x)-1)=e^{(\sum_{i \ge 1}f_i \frac{x^i}{i!})-1}$$
直观来讲：我们设一个非空集合的指数生成函数为 $E(x)$，设答案的指数生成函数为 $G(x)$，枚举一共用多少个集合，将这些集合组合起来，而集合之间不可区分。故：$$G(x)=\sum_{i=0}^{\infty}\frac{E^i(x)}{i!}$$
即多项式 $\exp$：$$G(x)=e^{E(x)}$$
而：$$E(x)=e^x-1$$
最后需要记得乘以 $n!$， 因为多项式 $\exp$ 本身会先除以 $n!$。

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 3e5 + 5, mod = 998244353, g = 3;
int rev[N], inv[N];
int f[N], q[N], h[N], tmp1[N], tmp2[N], tmp3[N], tmp4[N], tmp5[N], tmp6[N], tmp7[N];
int fac[N], query[N], ans[N];
int qpow(int a, int b){
    int res = 1;
    while(b){
        if(b & 1) res = res * a % mod;
        a = a * a % mod, b >>= 1;
    }
    return res;
}

void initinv(int N){
    inv[0] = inv[1] = 1;
    for(int i = 2; i < N; ++i) inv[i]= 1ll * inv[mod % i] * (mod - mod / i) % mod;
}

void NTT(int a[], int n, int op){
    int log2n = 0; while((1 << log2n) < n) log2n++;
    for(int i = 0; i < n; ++i){
        rev[i] = 0;
        for(int j = 0; j < log2n; ++j) if(i & (1 << j)) rev[i] |= 1 << (log2n - 1 - j);
    }
    for(int i = 0; i < n; ++i) if(i < rev[i]) swap(a[i], a[rev[i]]);
    for(int h = 2; h <= n; h <<= 1){
        int wn = qpow(g, (mod - 1) / h);
        if(op == -1) wn = qpow(wn, mod - 2);
        for(int j = 0; j < n; j += h){
            int w = 1;
            for(int k = j; k < j + h / 2; ++k){
                int u = a[k] % mod, t = w * a[k + h / 2] % mod;
                a[k] = (u + t) % mod, a[k + h / 2] = (u - t + mod) % mod;
                w = w * wn % mod;
            }
        }
    }
    if(op == -1){
        int inv = qpow(n, mod - 2);
        for(int i = 0; i < n; ++i) a[i] = a[i] * inv % mod;
    }
}

void poly_inv(int F[], int G[], int n){
    if(n == 1) return (void)(G[0] = qpow(F[0], mod - 2));
    int mid = (n + 1) >> 1;
    int lim = 1; while(lim < (n << 1)) lim <<= 1;
    poly_inv(F, G, mid);
    for(int i = 0; i < mid; ++i) tmp1[i] = G[i]; for(int i = mid; i < lim; ++i) tmp1[i] = 0;
    for(int i = 0; i < n; ++i) tmp2[i] = F[i]; for(int i = n; i < lim; ++i) tmp2[i] = 0;
    NTT(tmp1, lim, 1), NTT(tmp2, lim, 1);
    for(int i = 0; i < lim; ++i) tmp1[i] = tmp1[i] * (2 - tmp1[i] * tmp2[i] % mod + mod) % mod;
    NTT(tmp1, lim, -1);
    for(int i = 0; i < n; ++i) G[i] = tmp1[i];
    for(int i = 0; i < lim; ++i) tmp1[i] = tmp2[i] = 0;
}

void poly_dev(int F[], int G[], int n){
    for(int i = 1; i < n; ++i) G[i - 1] = i * F[i] % mod;
    G[n - 1] = 0;
}

void poly_invdev(int F[], int G[], int n){
    for(int i = 1; i < n; ++i) G[i] = F[i - 1] * qpow(i, mod - 2) % mod;
    G[0] = 0;
}

void poly_ln(int F[], int G[], int n){
    poly_dev(F, tmp3, n), poly_inv(F, tmp4, n);
    int lim = 1; while(lim < (n << 1)) lim <<= 1;
    NTT(tmp3, lim, 1), NTT(tmp4, lim, 1);
    for(int i = 0; i < lim; ++i) tmp3[i] = tmp3[i] * tmp4[i] % mod;
    NTT(tmp3, lim, -1); poly_invdev(tmp3, G, n);
}

void poly_exp(int F[], int G[], int n) {
    if (n == 1) return (void)(G[0] = 1);
    int mid = (n + 1) >> 1;
    poly_exp(F, G, mid); poly_ln(G, tmp5, n);
    for (int i = 0; i < n; ++i) tmp5[i] = (F[i] - tmp5[i] + mod) % mod;
    tmp5[0] = (tmp5[0] + 1) % mod;
    int lim = 1; while (lim < (n << 1)) lim <<= 1;
    for (int i = 0; i < lim; ++i) tmp6[i] = (i < n) ? tmp5[i] : 0;
    for (int i = 0; i < lim; ++i) tmp7[i] = (i < mid) ? G[i] : 0;
    NTT(tmp6, lim, 1); NTT(tmp7, lim, 1);
    for(int i = 0; i < lim; ++i) tmp6[i] = tmp6[i] * tmp7[i] % mod;
    NTT(tmp6, lim, -1);
    for(int i = 0; i < n; ++i) G[i] = tmp6[i];
    for(int i = 0; i < lim; ++i) tmp6[i] = tmp7[i] = 0;
}


signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int t; cin >> t;
    fac[0] = 1;
    for(int i = 1; i <= 100000; ++i) fac[i] = (fac[i - 1] * i) % mod;
    int m = 0;
    for(int i = 1; i <= t; ++i){ cin >> query[i]; if(query[i] > m) m = query[i]; }
    m++;
    static int invnum[N];
    invnum[1] = 1;
    for(int i = 2; i < m; ++i) invnum[i] = (mod - mod / i) * 1ll * invnum[mod % i] % mod;
    inv[0] = 1;
    for(int i = 1; i < m; ++i) inv[i] = inv[i - 1] * 1ll * invnum[i] % mod;
    for(int i = 0; i < m; ++i) f[i] = 0;
    f[0] = 0;
    for(int k = 1; k < m; ++k) f[k] = inv[k];
    for(int i = 0; i < m; ++i) q[i] = 0;
    poly_exp(f, q, m);
    for(int i = 1; i <= t; ++i){
        int x = query[i];
        cout << (q[x] * fac[x] % mod) << '\n';
    }
    return 0;
}
```