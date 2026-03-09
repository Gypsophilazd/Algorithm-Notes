# CF1774C Ice and Fire
`rating = 1300`
## 题意简述

对于任意参赛人数 $x$（选手战力值为 $1$ 到 $x$），在给定的一系列“战力大者胜 (1)”或“战力小者胜 (0)”的淘汰赛规则序列下，求理论上存在多少名选手有可能夺冠。

## Solution

- **Hint 1：倒推法边界控制。**

- **Hint 2**：一场淘汰赛的最终赢家，其存活概率**只受最后连续相同的比赛规则限制**，前面的所有规则都可以通过“暗箱操作（让神仙打架、弱者爆冷）”被完美规避。可以尝试模拟。

- **推论**：如果最后连续出现了 $k$ 场“大者胜”（或“小者胜”），那么战力值绝对垫底（或绝对登顶）的 $k$ 个人必定凑不够连胜的垫脚石，绝对无法生还。剩下的人全都有机会。**答案永远是：当前参赛总人数 $x - k$**。注意是当前 $i$ 的后缀，所以从前往后遍历即可。

## Code
```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int t; cin >> t;
    while(t--){
        int n; cin >> n;
        string s; cin >> s;
        int f = 1;
        for(int i = 0; i < n - 1; ++i){
            if(i > 0 && s[i] == s[i - 1]) f++;
            else f = 1;
            cout << (i + 2) - f << " ";
        }
        cout << '\n';
    }
    return 0;
}
```


# CF1768C Elemental Decompress
`rating = 1300`
## 题意简述

给出数组 $a$，要求构造两个排列 $p$ 和 $q$，使得对每个位置都有 $\max(p_i, q_i) = a_i$。
## Solution

- **Hint 1：极大值的鸽巢原理 & 供需关系提取。**

- **Hint 2：**
    
    1. 若 $\max(p_i, q_i) = x$，那么 $p_i$ 和 $q_i$ 中**必定有一个就是 $x$**。
        
    2. 如果 $x$ 在原数组只出现 1 次，那必须 $p_i = q_i = x$。
        
    3. 如果 $x$ 出现 2 次，一个位置必须给 $x$，另一个位置只能给一个**小于 $x$ 的闲置数字（填坑）**！

- **Hint 3**：分离出出现 0 次的数（库存）和出现 2 次的数（需求）。由于从小到大遍历，这两个序列天然有序。为了保证后面的大坑有数可填，**必须用最小的库存去填最小的需求！**（按位比对 $cnt0[i] < cnt2[i]$）。如果满足，直接存进映射表 $O(1)$ 分配。

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 2e5 + 5;
int cnt[N];

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int t; cin >> t;
    while(t--){
        memset(cnt, 0, sizeof(cnt));
        int n; cin >> n;
        bool flag = true;
        vector<int> a(n + 1);
        for(int i = 1; i <= n; ++i){
            cin >> a[i];
            cnt[a[i]]++;
            if(cnt[a[i]] > 2) flag = false;
        }
        if(!cnt[n] || cnt[1] > 1 || !flag){
            cout << "NO" << '\n';
            continue;
        }
        vector<int> cnt0, cnt2;
        for(int i = 1; i <= n; ++i){
            if(cnt[i] == 0) cnt0.push_back(i);
            else if(cnt[i] == 2) cnt2.push_back(i);
        }
        map<int, int> mp;
        for(int i = 0; i < cnt0.size(); ++i){
            if(cnt0[i] > cnt2[i]){
                flag = false;
                break;
            }
            mp[cnt2[i]] = cnt0[i];
        }
        vector<int> p(n + 1), q(n + 1);
        for(int i = 1; i <= n; ++i){
            if(cnt[a[i]] == 1) p[i] = a[i], q[i] = a[i];
            else if(cnt[a[i]] == 2) p[i] = a[i], q[i] = mp[a[i]], cnt[a[i]]++;
            else if(cnt[a[i]] == 3) q[i] = a[i], p[i] = mp[a[i]];
        }
        if(!flag){
            cout << "NO" << '\n';
            continue;
        }
        else{
            cout << "YES" << '\n';
            for(int i = 1; i <= n; ++i) cout << p[i] << " ";
            cout << '\n';
            for(int i = 1; i <= n; ++i) cout << q[i] << " ";
            cout << '\n';
        }
    }
    return 0;
}
```