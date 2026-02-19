## 题目描述

在2312年，宇宙中发现了 $n$ 台巨型对撞机，这些对撞机分别用 $1 \sim n$ 的自然数标识。科学家们不知道启动这些对撞机会发生什么危险事故，所以这些机器，刚开始都是出于关闭的状态。

随着科学家们的研究发现，第 $i$ 台对撞机启动是安全的，当且仅当其他已经启动的对撞机的标识数都跟这台对撞机标识数互质。（例如假设前面启动的对撞机标识数是 $j$，如果 $i$ 能启动，那么 $i,j$ 互质，即 $\gcd(i,j) = 1$）。如果两台对撞机的标识数不为互质数就启动，那么就会发生爆炸事故。

基于前面的研究，科学家们准备做各种启动和关闭对撞机的实验。为了确保科学家们的生命安全，你要设计一个远程遥控的软件。

刚开始，所有的对撞机都是关闭状态。你的程序将会收到许多询问，格式为“启动、关闭第 $i$ 台对撞机”。这个程序应该能处理这些询问（根据收到询问的先后顺序处理）。程序按照如下的格式输出处理结果。

如果询问为 `+ i`（表示启动第 $i$ 台对撞机），程序应该按照下面三种情况之一输出结果。

- `Success`，表示启动第 $i$ 台是安全的。
- `Already on`，表示第 $i$ 台在询问之前就已经启动了。
- `Conflict with j`，表示第 $i$ 台与前面已经启动的第 $j$ 台冲突。如果前面有多台对撞机跟 $i$ 冲突，那么只输出其中任何一台即可。

如果询问为 `- i`（表示关闭第 $i$ 台对撞机），程序应该按照下面两种情况之一输出结果。

- `Success`，表示关闭第 $i$ 台对撞机。
- `Already off`，表示第 $i$ 台对撞机在询问之前就已经关闭了。

## 输入格式

第一行输入两个以空格隔开的整数 $n$ 和 $m$，分别表示对撞机的数量和询问数。

接下来 $m$ 行，表示询问，每行仅可能是 `+ i` 或 `- i`，表示开启或关闭第 $i$ 台对撞机。

## 输出格式

输出 $m$ 行，输出结果按照上面题目给定格式输出。

## 输入输出样例 #1

### 输入 #1

```
10 10 
+ 6
+ 10
+ 5
- 10
- 5
- 6
+ 10 
+ 3
+ 6
+ 3
```

### 输出 #1

```
Success
Conflict with 6
Success
Already off
Success
Success
Success
Success
Conflict with 3
Already on
```

## 说明/提示

**数据范围**

$1 \le n,m \le 10^5$，$1 \le i \le n$。

---

## Solution
一道小模拟题,主要优化点在于`op == '+'`操作下,对于质因数是否冲突的判断

##### 代码段逻辑详解（`op == '+'`分支)
##### **功能定位**
这段代码处理用户尝试**启动对撞机**（`+ i`操作）时的逻辑，核心目标是判断启动是否安全。需满足以下条件：
1. 当前对撞机未启动（`vis[x] == 0`）。
2. 当前对撞机与所有已启动对撞机的标识数**互质**（即无共享质因数）。
##### **分步拆解**
##### **1. 状态检查与初始化**
`if (vis[x]) cout << "Already on\n";`
- **目的**：直接过滤已启动的对撞机。`vis[x]`数组记录对撞机状态，`vis[x] == 1`表示已启动。
#### **2. 质因数分解**
`vector<int> prime_num = getPrime(x);`
- **作用**：获取当前对撞机标识数 `x` 的所有**质因数**。
- **示例**：
    - 若 `x = 12`，分解结果为 `{2, 3}`。
    - 若 `x = 7`（质数），分解结果为 `{7}`。
- **技术实现**：通过试除法或更高效算法（如 Pollard-Rho）分解质因数。
#### **3. 冲突检测**
```
else {
	vector<int> prime_num = getPrime(x);
    bool flag = false;
    for (int p : prime_num) {
    if (factor.find(p) != factor.end())  {
	    cout << "Conflict with " << factor[p] << "\n";
        flag = true;
        break;
    }
}
```

- **核心逻辑**：遍历 `x` 的每个质因数，检查是否已被其他对撞机占用。
- **哈希表 `factor` 设计**：
    - **键**（Key）：质因数 `p`。
    - **值**（Value）：第一个占用该质因数的对撞机标识数。
- **冲突判定**：若存在共享质因数，说明 `x` 与 `factor[p]` 不互质（`gcd(x, factor[p]) >= p > 1`）。
#### **4. 安全启动处理**
```
if (!flag) {
	for (int p : prime_num) factor[p] = x;
    vis[x] = 1;
    cout << "Success\n";
}
```

- **无冲突时操作**：
    - **记录质因数**：将 `x` 的所有质因数加入哈希表，标记其“所有权”。
    - **更新状态**：设置 `vis[x] = 1`，标记为已启动。
- **示例**：
    - 启动 `x=6`（质因数 `{2,3}`）后，`factor[2]=6`、`factor[3]=6`。
    - 后续启动 `x=10`（质因数 `2`）时，检测到 `factor[2]` 存在，触发冲突。

---
### **关键设计思想**
#### **1. 质因数的唯一性判定**
- **数学原理**：若两数有公共质因数，则它们不互质（`gcd(a,b) ≥ 该质因数`）。
- **优化意义**：将互质性判断简化为质因数存在性检查，时间复杂度从 `O(k)` 降为 `O(1)`（`k` 为已启动对撞机数量）。
#### **2. 哈希表的动态维护**
- **写入时机**：仅在安全启动时更新哈希表，确保记录最新的有效冲突关系。
- **删除逻辑**：在关闭对撞机时，需遍历其质因数并从 `factor` 中删除对应条目
#### **3. 性能与正确性平衡**
- **时间复杂度**：
    - 质因数分解：`O(√x)`（试除法）。
    - 冲突检测：`O(质因数数量)`，通常远小于 `O(n)`。
- **空间复杂度**：`O(n)` 存储质因数和状态。
## AC Code
```
#include <bits/stdc++.h>
using namespace std;
const int N = 1e5 + 5;
int vis[N] = {0}; // 记录对撞机状态
unordered_map<int, int> factor; // 记录已启动对撞机的质因数
// 预处理每个数的质因数
vector<int> getPrime(int x) {
    vector<int> factors;
    for (int i = 2; i * i <= x; ++i) {
        if (x % i == 0) {
            factors.push_back(i);
            while (x % i == 0) x /= i;
        }
    }
    if (x > 1) factors.push_back(x);
    return factors;
}
int main() {
    ios::sync_with_stdio(0), cin.tie(0),  cout.tie(0);
    int n, m; cin >> n >> m;
    while (m--) {
        char op; int x;
        cin >> op >> x;
        if (op == '+') {
            if (vis[x]) cout << "Already on\n";
            else {
                vector<int> prime_num = getPrime(x);
                bool flag = false;
                for (int p : prime_num) {
                    if (factor.find(p) != factor.end())  {
                        cout << "Conflict with " << factor[p] << "\n";
                        flag = true;
                        break;
                    }
                }
                if (!flag) {
                    for (int p : prime_num) factor[p] = x;
                    vis[x] = 1;
                    cout << "Success\n";
                }
            }
        }
        else {
            if (!vis[x]) cout << "Already off\n";
            else {
                vector<int> prime_num = getPrime(x);
                for (int p : prime_num) factor.erase(p);
                vis[x] = 0;
                cout << "Success\n";
            }
        }
    }
    return 0;
}
```