## 题目描述

这个背包最多可以装 $10^5$ 大小的东西

有 $n$ 种商品，她要准备出摊了

每种商品体积为 $v_i$，都有无限件

给定 $m$，对于 $s\in [1,m]$，请你回答用这些商品恰好装 $s$ 体积的方案数

## 输入格式

第一行两个正整数 $n,m$。
第二行 $n$ 个正整数，表示每种商品的体积。

## 输出格式

输出 $m$ 行，第 $i$ 行代表 $s=i$ 时方案数，对 $998244353$ 取模。

## 输入输出样例 #1

### 输入 #1

```
2 4
1 2
```

### 输出 #1

```
1
2
2
3
```

## 说明/提示

【数据范围】  
对于 $30\%$ 的数据，$1\le n,m \le 3000$；  
对于 $60\%$ 的数据，纯随机生成；   
对于 $100\%$ 的数据， $1\le n,m \le 10^5$，$1\le v_i \le m$。



## Solution

  对于一般的完全背包问题，如果采用传统的动态规划解法，其时间复杂度 $O(n^2)$ 在 $n = 10^5$的情况下显然不可接受。注意到题目提及是**恰好**装 $s$ 体积的方案数，即为计数问题，考虑生成函数解法。

  对每个 $v_i$，其生成函数如下：

$$
f(x) = \sum_{i \ge 0,\ i \equiv 0 \pmod{v}} x^i = \sum_{i \ge 0} x^{v_i} = \frac{1}{1 - x^v}
$$

但是，题目求 $m$ 个方案数，对于两个 $n$ 项生成函数卷积，其时间复杂度为 $NTT,O(nlogn)$ ，总的时间复杂度为 $O(nmlogn)$，仍然不可接受。

因此考虑进阶做法，比较自然的联想是考虑多项式 $ln$，其时间复杂度为 $O(nlogn)$，再将各项 $v_i$ 相加，时间复杂度为一个类似级数求和的式子，总体为 $O(nlogn)$，最后再进行多项式 $exp$ 其时间复杂度为 $O(nlogn)$， 总的时间复杂度为 $O(nlogn)$。

具体实现细节上，对如上多项式进行 $ln$ 操作后，可以证明为以下的式子：

令：
$$
G(x) = \ln\left( \frac{1}{1 - x^{V_i}} \right)
$$
则：
$$
G'(x) = \left[\ln\left( \frac{1}{1 - x^{V_i}} \right)\right]' = -\left[\ln(1 - x^{V_i})\right]'
$$

根据对数函数的性质，我们有：
$$
\ln(x^c) = c \cdot \ln(x)
$$
代入得：
$$
G'(x) = \frac{d}{dx} \left(-\ln(1 - x^{V_i}) \right) = \frac{V_i \cdot x^{V_i - 1}}{1 - x^{V_i}} = V_i \cdot \sum_{j=0}^{\infty} x^{V_i \cdot j + V_i - 1}
$$

再将该式积分回去，我们有：
$$
G(x) = \int G'(x)\,dx = V_i \cdot \sum_{j=0}^{\infty} \int x^{V_i \cdot j + V_i - 1}\,dx
= V_i \cdot \sum_{j=0}^{\infty} \frac{x^{V_i \cdot j + V_i}}{V_i \cdot j + V_i}
= \sum_{j=0}^{\infty} \frac{x^{V_i \cdot j + V_i}}{j + 1}
= \sum_{j=1}^{\infty} \frac{x^{V_i \cdot j}}{j}
$$
实现方面，我们先求的是：$$\ln\left({1 - x^{V_i}} \right)$$
再进行多项式 $exp$ 操作，最后进行多项式求逆。

提供代码，多项式全家桶可以自行更改。

## Code
```Cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 3e5 + 5, mod = 998244353, g = 3;
int rev[N], inv[N];
int f[N], q[N], h[N], tmp1[N], tmp2[N], tmp3[N], tmp4[N], tmp5[N], tmp6[N], tmp7[N];

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
    int n, m; cin >> n >> m;
    for(int i = 0; i < n; ++i){
        int x; cin >> x;
        f[x]++;
    }
    int N; for(N = 1; N <= m; N <<= 1);
    initinv(N);
    for(int i = 1; i <= m; ++i) inv[i] = qpow(i, mod - 2);
    for(int i = 1; i <= m; ++i){
        if(f[i]){
            for(int j = i; j < N; j += i) q[j] = (q[j] + 1ll * f[i] * (mod - inv[j / i])) % mod; // 构造生成函数
        }
    }
    poly_exp(q, h, N);
    memset(q, 0, sizeof(f));
    poly_inv(h, q, N);
    for(int i = 1; i <= m; ++i) cout << q[i] << '\n';
    return 0;
}

```