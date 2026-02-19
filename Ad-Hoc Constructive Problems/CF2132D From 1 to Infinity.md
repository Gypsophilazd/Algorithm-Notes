`rating = 1600`
## 题目描述

Vadim wanted to understand the infinite sequence of digits that consists of the positive integers written consecutively from $1$ to infinity. That is, this sequence looks like: $$123456789101112131415 \ldots$$

To avoid looking into infinity, Vadim cut this sequence at the $k$ -th digit and discarded everything after it. Thus, exactly $k$ digits remained in the sequence. Help him find the sum of the digits in the remaining sequence.

## 输入格式

Each test consists of several test cases. The first line contains a single integer $t$$(1\le t \le 2 \cdot 10^4)$ — the number of test cases. The following lines describe the test cases.

In a single line of each test case, there is an integer $k$ — the number of digits in the remaining sequence $(1 \le k \le 10^{15})$.

## 输出格式

For each given $k$, output the sum of the digits in the sequence of length $k$.

## 输入输出样例 #1

### 输入 #1

```
6
5
10
13
29
1000000000
1000000000000000
```

### 输出 #1

```
15
46
48
100
4366712386
4441049382716054
```

## 说明/提示

In the first sample, the remaining sequence will be $12345$.

In the second sample, the remaining sequence will be $1234567891$.

In the third sample, the remaining sequence will be $1234567891011$.

## Solution

题意重述：把所有正整数 $1\dots n$ 依次拼接在一起，给定 $k$，求前 $k$ 位数位和。

### 观察与思路

注意到：数字是按照长度进行分段，即：

- 一位数：从 $1$ 到 $9$，共 $9$ 个数，总位数 $9⋅1=9$。

- 两位数：从 $10$ 到 $99$，共 $90$ 个数，总位数 $90⋅2=180$。

- 三位数：从 $100$ 到 $999$，共 $900$ 个数，总位数 $900⋅3=2700$。

推广来说：$d$ 位数有 $9⋅10^{d−1}$ 个，总位数 $9\cdot 10^{d-1}\cdot d$。

**考虑分块的思想，预处理某个整段的数字和，找出 $k$ 属于哪一位数的段中，在最后一个不完整的段中，利用拆位求贡献快速求解。**

1. 如何预处理 $d$ 位数所有的数位和？

考虑最高位贡献：从 $1$ 到 $9$，各出现 $10^{d-1}$ 次。总和为 $(1+2+…+9)\cdot 10^{d-1} = 45 \cdot 10^{d-1}$。

其他 $d-1$ 位：每个位置 $0 \dots 9$ 平均出现次数 $9\cdot 10^{d-2}$。总和为 $45 \cdot 9 \cdot 10^{d-2}$。

总贡献：$45 \cdot 10^{d-1} + (d-1)\cdot 45 \cdot 9 \cdot 10^{d-2}$。

2. 那么，我们如何快速求出前 $m$ 个整数拼接后所有数字的和？

这里仔细说明。设我们正在看某一位，位权是：个位：$f=1$，十位：$f=10$，百位：$f=100$，以此类推。

假设总数是 $n$，我们想知道这一位上所有数字的贡献。

考虑每 $10f$ 个数是一个大循环。

例如看十位：从 $0$ 到 $99$，十位刚好跑完 $0..9$ 各 $10$ 次。从 $100$ 到 $199$，又跑完一轮。因此：每个长度为 $10f$ 的区间里，$0..9$ 在该位各出现 $f$ 次。总和就是 $45 \cdot f$。

那么在 $1..n$ 中，循环次数是 $\lfloor \frac{n}{10f} \rfloor$。所以完整循环贡献是：$\left\lfloor \frac{n}{10f} \right\rfloor \cdot 45$。

除了完整循环，还会多出来一部分，长度为 $(n \bmod 10f) + 1$。在这一段里，当前位的数字只到一部分： 当前位数字为 $(\frac{n}{f}) \bmod 10$，记作 $cur$。
 
在这一轮中，小于 $cur$ 的数字 $0..cur-1$，每个都会完整走 $f$ 次。贡献为 $\frac{cur \cdot (cur-1)}{2} \cdot f$。

这一位等于 $cur$ 的部分还要覆盖尾数：余数长度为 $(n \bmod f)+1$。所以再加上：$cur \cdot \bigl((n \bmod f)+1\bigr)$。

把三部分加起来，就是这一位的贡献。

$$\operatorname{digit\_sum}(n) = \sum_{f=1,10,100,\dots} \Bigg[ \left(\frac{n}{10f}\right)\cdot 45f + \frac{cur \cdot (cur-1)}{2}\cdot f + cur \cdot \bigl(n \bmod f + 1\bigr) \Bigg]$$

其中 $cur = (n/f) \bmod 10$。

时间复杂度 $O(t· \log k)$。

## Code

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;

int calc(int x){
    if(x == 1) return 45;
    int p = 1; for(int i = 1; i < x; ++i) p *= 10;
    int q = p / 10, sum = 45 * p;
    return sum + (x - 1) * 45 * 9 * q;
}

int sum(int n){
    if(n <= 0) return 0;
    int res = 0, f = 1;
    while(f <= n){
        int cur  = (n / f) % 10;
        res += (n / (f * 10)) * 45 * f;
        res += cur * (cur - 1) / 2 * f;
        res += cur * (n % f + 1);
        f *= 10;
    }
    return res;
}

signed main() {
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int t; cin >> t;
    while(t--){
        int k; cin >> k;
        int ans = 0, d = 1, cnt = 9, len = 1;
        while(1){
            int tot = cnt * len;
            if(k > tot) ans += calc(len), k -= tot, len++, cnt *= 10;
            else{
                int st = pow(10ll, len - 1);
                int a = k / len, b = k % len;
                 // a 表示完整数字，b 表示余数
                int l = st, r = st + a - 1;
                ans += sum(r) - sum(l - 1);
                if(b > 0){
                    int x = st + a;
                    string s = to_string(x);
                    for(int i = 0; i < b; ++i) ans += s[i] - '0';
                }
                break;
            }
        }
        cout << ans << '\n';
    }
    return 0;
}