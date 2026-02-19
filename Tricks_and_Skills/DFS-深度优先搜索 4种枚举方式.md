### 1. **组合型枚举** (Combination Enumeration)

- **特点**：选定 `m` 个数，顺序不重要，**不重复选择**。
- 关键点：选出 m 个，顺序不重要，不重复选择。
- **代码示例**：
```
#include <bits/stdc++.h>
using namespace std;
int n, m;
const int N = 30;
int ans[N];
void dfs(int st, int pos){
    if(n - st + 1 < m - pos) return ;
    if(pos == m){
        for(int i = 1; i <= m; ++i) cout << ans[i] << " ";
        cout << endl;
        return ;
    }
    for(int i = st; i <= n; ++i){
        ans[pos + 1] = i;
        dfs(i + 1, pos + 1);
    }
}
int main(){
    ios::sync_with_stdio(0), cin.tie(0);
    cin >> n >> m;
    dfs(1, 0);
    return 0;
}
```

### 2. **排列型枚举** (Permutation Enumeration)

- **特点**：选定 `k` 个数，顺序重要，**不重复选择**。
- **关键点**：递归时不限制元素的选择顺序，允许每次选择的元素位置不同。
- **代码示例**：从n个数中选k个进行全排列 每次从头枚举
-  通过vis数组避免重复选择
- 遍历范围始终是全集（1~n），但通过标记过滤已用元素
```
#include <bits/stdc++.h>
using namespace std;
const int N = 15;
int n, k;
bool vis[N] = {false};
int ans[N];
void dfs(int st){
    if(st > k){
        for(int i = 1; i <= k; ++i) cout << ans[i] << " ";
        cout << endl;
        return ;
    }
    for(int i = 1; i <= n; ++i){
        if(!vis[i]){
            vis[i] = true;
            ans[st] = i;
            dfs(st + 1);
            vis[i] = false;
        }
    }
}
int main(){
    ios::sync_with_stdio(0), cin.tie(0);
    cin >> n >> k;
    dfs(1);
    return 0;
}
```

### 3. **指数型枚举** (Subset Enumeration)

- **特点**：从 `n` 个数中选择任意个数，顺序不重要，**可以重复选择**。
- **关键点**：对于每个元素，递归时有两种选择：“选”或“不选”。
- 不需要vis数组
- **代码示例**：

```
#include <bits/stdc++.h>
using namespace std;
const int N = 15;
int n;
char ans[N];
void dfs(int dep){
    if(dep == n + 1){
        for(int i = 1; i <= n; ++i) cout << ans[i];
        cout << endl;
        return ;
    }
    ans[dep] = 'N';   //选
    dfs(dep + 1);
    ans[dep] = 'Y';  //不选
    dfs(dep + 1);
   /*
   类似的 在外部记录一个cnt值
   不选:直接dfs(dep+1)
   选:ans[cnt++] = dep;再dfs(dep + 1),再cnt--回溯
   */
}
int main(){
    ios::sync_with_stdio(0), cin.tie(0);
    cin >> n;
    dfs(1);  
    return 0;
}
```


### 4. **元组型枚举** (Tuple Enumeration)
```
#include <bits/stdc++.h>
using namespace std;
const int N = 10;
int n, k, cnt = 1;
int ans[N];
void dfs(int dep){
    if(dep == n + 1){
        for(int i = 1; i <= n; ++i) cout << ans[i] << " ";
        cout << endl;
        return ;
    }
    for(int i = 1; i <= k; ++i){
        ans[dep] = i;
        dfs(dep + 1);
    }
}
int main(){
    ios::sync_with_stdio(0), cin.tie(0);
    cin >> n >> k;
    dfs(1);
    return 0;
}
```
### 一、四大枚举类型核心对比表

| **维度**    | **排列型枚举**  | **指数型枚举（子集）** | **组合型枚举**                 | **元组型枚举**     |
| --------- | ---------- | ------------- | ------------------------- | ------------- |
| **本质特征**  | 全排列，顺序敏感   | 元素选/不选，顺序不敏感  | 选k个元素，顺序不敏感               | 每个位置独立选值，可重复  |
| **递归参数**  | `u`（已选元素数） | `pos`（当前决策位置） | `start`（起始位置）+ `cnt`（已选数） | `dep`（当前填充位置） |
| **终止条件**  | `u > n`    | `pos > n`     | `cnt == k`                | `dep > n`     |
| **时间复杂度** | O(n!)      | O(2ⁿ)         | O(C(n,k))                 | O(kⁿ)         |
| **剪枝策略**  | 无          | 无             | `剩余元素数 >= 待选数`            | 无（除非添加约束）     |
| **输出顺序**  | 依赖遍历顺序     | 字典序（先不选后选）    | 升序                        | 数值递增序         |
| **典型应用**  | 任务调度、路径规划  | 子集和、特征组合      | 选代表、药品配方                  | 密码生成、参数组合测试   |
