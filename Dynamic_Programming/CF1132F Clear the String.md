`rating = 2000`
## 题目描述

给定一个长度为 $n$ 的字符串 $s$，该字符串仅由小写拉丁字母组成。你可以对该字符串进行如下操作：每次操作可以删除一个连续的子串，前提是该子串中的所有字母都相同。例如，从字符串 abbbbaccdd 删除子串 bbbb 后，得到字符串 aaccdd。

请计算删除整个字符串 $s$ 所需的最少操作次数。

## 输入格式

第一行包含一个整数 $n$（$1 \le n \le 500$），表示字符串 $s$ 的长度。

第二行包含字符串 $s$（$|s| = n$），该字符串仅由小写拉丁字母组成。

## 输出格式

输出一个整数，表示删除字符串 $s$ 所需的最少操作次数。

## 输入输出样例 #1

### 输入 #1

```
5
abaca

```

### 输出 #1

```
3

```

## 输入输出样例 #2

### 输入 #2

```
8
abcddcba

```

### 输出 #2

```
4

```

## 说明/提示

由 ChatGPT 4.1 翻译

## Solution

区间 DP 典题。考虑状态定义 `dp[l][r]`：将子串 `s[l...r]` 全部删除所需的最小操作数。
状态转移：面对区间 `[l, r]` 的首个字符 `s[l]`，我们有两种策略：
	1. **策略 A（孤立删除）**： 不考虑与后面的字符产生联系，直接花 1 步删掉 `s[l]`。 转移：`dp[l][r] = dp[l+1][r] + 1` 
	2. **策略 B（合并）**： 在区间内寻找一个位置 `k` ($l < k \le r$)，使得 `s[l] == s[k]`。 这意味着我们可以**先清空它们中间的障碍 `[l+1, k-1]`**，让 `s[l]` 和 `s[k]` 贴在一起，然后在删除 `[k, r]` 的时候，顺带把 `s[l]` 免费删掉。 $$dp[l][r] = min(dp[l][r], dp[l+1][k-1] + dp[k][r])$$

坑点在于
	1. **循环顺序**：必须先枚举区间长度 `len`，再枚举左端点 `l`。这是保证子问题无后效性的基石。也是区间 DP 的一般规范。
	2. **初始化边界**：`len=1` 时，单个字符需要 1 步 (`dp[i][i] = 1`)。当 `k = l+1` 时，`[l+1, k-1]` 会形成 `l > r` 的非法区间，此时默认 `dp` 值为 0 恰好符合逻辑（中间没有障碍需要清空，代价为 0）。

## Code

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 505;
int dp[N][N] = {0};

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n; cin >> n;
    string s; cin >> s;
    for(int i = 1; i <= n; ++i) dp[i][i] = 1;
    for(int len = 2; len <= n; ++len){
        for(int l = 1; l + len - 1 <= n; ++l){
            int r = l + len - 1;
            if(l != n) dp[l][r] = dp[l + 1][r] + 1;
            for(int k = l + 1; k <= r; ++k){
                if(s[l - 1] == s[k - 1]) dp[l][r] = min(dp[l][r], dp[l + 1][k - 1] + dp[k][r]);
            }
        } 
    }
    cout << dp[1][n] << endl;
    return 0;
}
```
[Open: Gemini_Generated_Image_e2vac4e2vac4e2va.png](Note/ACM%E8%AE%AD%E7%BB%83/Dynamic_Programming/images/586851bd71212eea48115d69b2cb349d_MD5.jpg)
![](Note/ACM%E8%AE%AD%E7%BB%83/Dynamic_Programming/images/586851bd71212eea48115d69b2cb349d_MD5.jpg)