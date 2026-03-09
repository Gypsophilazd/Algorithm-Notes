## 题目描述

人们在英文字典中查找某个单词的时候可能不知道该单词的完整拼法，而只知道该单词的一个错误的近似拼法，这时人们可能陷入困境，为了查找一个单词而浪费大量的时间。带有模糊查询功能的电子字典能够从一定程度上解决这一问题：用户只要输入一个字符串，电子字典就返回与该单词编辑距离最小的几个单词供用户选择。

字符串 $a$ 与字符串 $b$ 的编辑距离是指：允许对 $a$ 或 $b$ 串进行下列“编辑”操作，将 $a$ 变为 $b$ 或 $b$ 变为 $a$，最少“编辑”次数即为距离。

1. 删除串中某个位置的字母；
2. 添加一个字母到串中某个位置；
3. 替换串中某一位置的一个字母为另一个字母。

JSOI 团队正在开发一款电子字典，你需要帮助团队实现一个用于模糊查询功能的计数部件：对于一个待查询字符串，如果它是单词，则返回 $-1$；如果它不是单词，则返回字典中有多少个单词与它的编辑距离为 $1$。

## 输入格式

第一行包含两个正整数 $N$ 和 $M$。

接下来的 $N$ 行，每行一个字符串，第 $i+1$ 行为单词 $W_i$，单词长度在 $1$ 至 $20$ 之间。

再接下来 $M$ 行，每行一个字符串，第 $i+N+1$ 表示一个待查字符串 $Q_i$。待查字符串长度在 $1$ 至 $20$ 之间。$W_i$ 和 $Q_i$ 均由小写字母构成，文件中不包含多余空格。

## 输出格式

输出应包括 $M$ 行，第 $i$ 行为一个整数 $X_i$：

- $X_i = -1$ 表示 $Q_i$ 为字典中的单词；

- 否则 $X_i$ 表示与 $Q_i$ 编辑距离为 $1$ 的单词的个数。

## 输入输出样例 #1

### 输入 #1

```
4 3
abcd
abcde
aabc
abced
abcd
abc
abcdd
```

### 输出 #1

```
-1
2
3
```

## 说明/提示

### 样例解释

- `abcd` 在单词表中出现过；
- `abc` 与单词 `abcd`、`aabc` 的编辑距离都是 $1$；
- `abcdd` 与单词 `abcd`、`abcde`、`abced` 的编辑距离都是 $1$。

### 数据范围与约定

- 所有单词互不相同，但是查询字符串可能有重复；
- 对 $50\%$ 的数据范围，$N,M\le 10^3$；
- 对 $100\%$ 的数据范围，$N,M\le 10^4$。


## Solution

Trie + DFS，极其优雅的状态机结构。
一般的查询函数不足以支持本题比较庞大的平行状态构造，所以考虑 DFS 在 Trie 上进行搜索。
本题存在多次查询的限制，故考虑增加一个技巧：时间戳 `tim`。每个字符串每次只能被同一个平行的DFS访问，玩过就记录 `ans++`。不需要任何清空操作，只靠核对时间戳，就完成了时空上的绝对隔离。 这样 DFS 只需要四个参数，`void dfs(int p, int len, bool flag, int tim)`。

接下来考虑删除，修改，增加的操作
- 删除：直接跳过当前的字母
- 修改：直接遍历从 `a-z` $26$ 个字母，然后从 `ch[p][i]` 开启不同分支（`len` 不变）。
- 增加：类似修改，从 `ch[p][i]` 开启分支，但是 `len + 1`。
考虑剪枝，如果 `!ch[p][j]` 直接 `continue` 可以省下大量时间。
细节很多，参考 Code。

## Code

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 1e4 + 5;
int ch[20 * N][26], cnt[N * 20], idx, ans, LEN;
char s[25];
int vis[N * 20], sig = 0;
void trie_insert(char *s){
    int p = 0;
    for(int i = 0; s[i]; ++i){
        int j = s[i] - 'a';
        if(!ch[p][j]) ch[p][j] = ++idx;
        p = ch[p][j]; 
    }
    cnt[p]++;
}

void dfs(int p, int len, bool flag, int tim){
    if(len == LEN && cnt[p]){
        if(!flag) sig = true;
        else{
            if(vis[p] != tim){
                ans++;
                vis[p] = tim;
            }
        }
    }
    if(sig) return ;
    if(len < LEN){
        int j = s[len] - 'a';
        if(ch[p][j]) dfs(ch[p][j], len + 1, flag, tim);
    }
    if(!flag){
        if(len < LEN) dfs(p, len + 1, 1, tim);
        for(int i = 0; i < 26; ++i){
            if(!ch[p][i]) continue;
            dfs(ch[p][i], len, 1, tim);
            if(i != (s[len] - 'a') && len < LEN) dfs(ch[p][i], len + 1, 1, tim);
        }
    }

}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n, m; cin >> n >> m;
    for(int i = 1; i <= n; ++i){
        cin >> s;
        trie_insert(s);
    }
    for(int i = 1; i <= m; ++i){
        cin >> s;
        ans = 0, LEN = strlen(s), sig = 0;
        dfs(0, 0, 0, i);
        if(sig) cout << -1 << '\n';
        else cout << ans << '\n';
    }
    return 0;
}

```