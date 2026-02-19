`rating = 1700`
## 题目描述

给定正整数 $k$。称一个 01 字符串是好的当且仅当其可以通过把**连续** $k$ 个 $1$ **全部改成** $0$ 来达到全为 $0$，例如 $k=2$ 时，`000`，`0110`，`110110` 是好的而 `010`，`101` 不是好的。

多次询问，每次给定 $l,r$，询问长度在 $[l,r]$ 内的 01 字符串有多少个是好的。由于结果可能过大，答案对 $10^9+7$ 取模。

## 输入

Input contains several test cases.

The first line contains two integers $t$ and _$k$ ($1 ≤ t, k ≤ 10^5$), where $t$ represents the number of test cases.

The next _t_ lines contain two integers $a_i$ and $b_i$ ($1 ≤ a_i ≤ b_i ≤ 10^5$), describing the $i-th$ test.
## 输入输出样例 #1

### 输入 #1

```
3 2
1 3
2 3
4 4
```

### 输出 #1

```
6
5
5
```

## Solution

注意到询问量和题设条件都非常大，我们需要考虑是否存在 $O(1)$ 得出答案的做法。静态区间查询会想到前缀和，也就可以联想到使用 dp 解决问题。

设 `dp[x]` 表示存在 $x$ 个 $1$ 的方法数。那么，状态转移方程可以很轻松的据题意确定。即：
$$dp[x]=dp[x-1]+dp[x-k]$$
也就是说，我既可以选择不更换 $x-1$，也可以选择从 $x-k$ 转移过来。再预处理一个前缀和，即可在线地更新询问。

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int mod = 1e9 + 7;
const int N = 1e5 + 5;
int k, dp[N], prefix[N];

void solve(){
    int l, r; cin >> l >> r;
    cout << (prefix[r] - prefix[l - 1] + mod) % mod << '\n';
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int t; cin >> t >> k;
    dp[0] = prefix[0] = 1;
    for(int i = 1; i <= N - 5; ++i){
        if(i >= k) dp[i] = (dp[i - 1] + dp[i - k]) % mod;
        else dp[i] = dp[i - 1];
        prefix[i] = (prefix[i - 1] + dp[i]) % mod;
    }
    while(t--) solve();
    return 0;
}
```

