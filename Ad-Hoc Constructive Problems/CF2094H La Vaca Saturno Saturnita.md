
## 题目描述

Saturnita 的情绪取决于一个长度为 $n$ 的数组 $a$（只有他知道其含义）以及一个函数 $f(k, a, l, r)$（只有他知道如何计算）。以下是该函数的伪代码实现：

```
function f(k, a, l, r):
   ans := 0
   for i from l to r (inclusive):
      while k is divisible by a[i]:
         k := k/a[i]
      ans := ans + k
   return ans
```

给定 $q$ 个查询，每个查询包含整数 $k$、$l$ 和 $r$。对于每个查询，请输出 $f(k,a,l,r)$ 的值。

## 输入格式

第一行包含一个整数 $t$（$1 \leq t \leq 10^4$）——测试用例的数量。

每个测试用例的第一行包含两个整数 $n$ 和 $q$（$1 \leq n \leq 10^5$，$1 \leq q \leq 5 \cdot 10^4$）。

第二行包含 $n$ 个整数 $a_1, a_2, \ldots, a_n$（$2 \leq a_i \leq 10^5$）。

接下来的 $q$ 行，每行包含三个整数 $k$、$l$ 和 $r$（$1 \leq k \leq 10^5$，$1 \leq l \leq r \leq n$）。

保证所有测试用例的 $n$ 之和不超过 $10^5$，且所有测试用例的 $q$ 之和不超过 $5 \cdot 10^4$。

## 输出格式

对于每个查询，在新的一行输出答案。

## 输入输出样例 #1

### 输入 #1

```
2
5 3
2 3 5 7 11
2 1 5
2 2 4
2310 1 5
4 3
18 12 8 9
216 1 2
48 2 4
82944 1 4
```

### 输出 #1

```
5
6
1629
13
12
520
```

## 说明/提示

翻译由 DeepSeek V3 完成


## 题目概述

给定长度为 $n$ 的数组 $a[1…n]$（每个元素 $\le10^5$），和若干查询 (k,l,r)(k,l,r)，需要计算一个隐藏函数

> **f(k,a,l,r)**
> 
>    - 维护当前值 $K$（初始 $K=k$）和答案累加器 $ans=0$。
>    
>    - 从下标 $i=li=l$ 开始，重复：
>     
>     1. 在区间 $[i..r]$ 找到第一个位置 $j$ 使得 $a_j$ 整除 $K$；
>         
>     2. 若找不到或 $K=1$，则把剩余 $[i..r]$ 都累加上 $K$：  
>         $ans+=K×(r−j+1))$，退出；
>         
>     3. 否则，先把 $[i..j-1]$ 累加上当前 KK：  
>         $\text{ans} += K\times (j-i)$；
>         
>     4. 再不断除去所有因子 $a_j$
>        
```cpp
while (K % a[j] == 0) K /= a[j];
```
5. 将新的 $K$ 再加一次：  $ans += K$；
6. 设置 $i=j+1$，继续。
对每个查询输出最终 $ans$。

---

## 关键观察

1. **只有因子会“改变”$K$**  
    只有当 $ai∣K$ 时，才会执行“除因子”逻辑；否则只是在累加 $ans$。
    
2. **每个因子只会使用一次**  
    若某个因子 $d$ 在位置 $i_1$ 被用掉，$K$ 中不再包含 $d$，后续相同的值不会再次触发。  
    因此整除操作的次数 $≤k$ 的正因子个数 $d(k)$。
    
3. **因子枚举和首次出现定位**
    
    - 预处理所有数 $\ldots10^5$的因子列表 `divs[x]`。
        
    - 存储每个值 $v$ 在数组中出现的位置列表 `pos[v]`（升序）。
        
    - 对每个因子$d∈divs[K]$，通过二分在 `pos[d]` 上找下一个 $≥i$ 的下标。
        

---

## 算法流程

```
1. 预处理阶段：
   - 上限 A = 10⁵，建立 `divs[1..A]`：
     for d in [1..A]:
       for multiple m = d; m ≤ A; m += d:
         divs[m].push_back(d)
   - 每次读入新测试：
     · 读入 n, q, 数组 a[1..n]
     · 清空并重建 `pos[v]`：遍历 i=1..n，执行 `pos[a[i]].push_back(i)`

2. 处理每个查询 (k, l, r)：  
   K ← k;  
   i ← l;  
   ans ← 0;  

   while i ≤ r:
     cur ← +∞
     for each d in divs[K]:
       // 在 pos[d] 上二分找第一个 ≥ i
       idx ← lower_bound(pos[d].begin(), pos[d].end(), i)
       if idx exists and *idx ≤ r:
         cur ← min(cur, *idx)

     if cur == +∞ or K == 1:
       // 剩余整段都加 K
       ans += K * (r - i + 1)
       break

     // 在 cur 处触发因子
     ans += K * (cur - i)
     // 除尽所有 a[cur] 的因子
     while(K % a[cur] == 0): K /= a[cur]
     ans += K
     i = cur + 1

   输出 ans
```

---

## 复杂度分析

- **因子预处理**：$O(Alog⁡A),A=10^5$。
    
- **位置表构建**：$O(n)$。
    
- **单次查询**：
    
    - 枚举因子$divs[K]$（$d(K)$ 个），对每个做一次二分 $O(log⁡n)$。
        
    - 外层循环次数 $≤d(K)$。
        
    - 总计 $O(d(K)log⁡n)$。
        

在题目给定的 $∑n\le10^5、∑q\le5\times10^4$ 下足够高效。

---

## 参考代码（C++）

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

static const int A = 100000;
vector<int> divs[A+1];
vector<int> pos[A+1];
int a[200005];

int main(){
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // 预处理：枚举因子
    for(int d = 1; d <= A; d++){
        for(int m = d; m <= A; m += d){
            divs[m].push_back(d);
        }
    }

    int T; cin >> T;
    while(T--){
        int n, q;
        cin >> n >> q;
        for(int i = 1; i <= n; i++){
            cin >> a[i];
            pos[a[i]].push_back(i);
        }

        while(q--){
            ll K; int L, R;
            cin >> K >> L >> R;

            ll ans = 0;
            int i = L;
            while(i <= R){
                int cur = INT_MAX;
                for(int d : divs[K]){
                    auto &v = pos[d];
                    auto it = lower_bound(v.begin(), v.end(), i);
                    if(it != v.end() && *it <= R){
                        cur = min(cur, *it);
                    }
                }
                if(cur == INT_MAX || K == 1){
                    ans += K * (R - i + 1);
                    break;
                }
                ans += K * (cur - i);
                while(K % a[cur] == 0) K /= a[cur];
                ans += K;
                i = cur + 1;
            }

            cout << ans << "\n";
        }

        // 清理 pos
        for(int i = 1; i <= n; i++){
            pos[a[i]].clear();
        }
    }
    return 0;
}
```

---
