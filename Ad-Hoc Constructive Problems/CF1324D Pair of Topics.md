`rating = 1400`
## 题目描述

下一节高中课程需要讨论两个主题。第 $i$ 个主题对老师的有趣程度为 $a_i$ 单位，对学生的有趣程度为 $b_i$ 单位。

如果一对主题 $i$ 和 $j$（$i < j$）满足 $a_i + a_j > b_i + b_j$（即对老师来说更有趣），则称这对主题为“好对”。

你的任务是计算有多少对“好对”主题。

## 输入格式

输入的第一行包含一个整数 $n$（$2 \le n \le 2 \cdot 10^5$），表示主题的数量。

第二行包含 $n$ 个整数 $a_1, a_2, \dots, a_n$（$1 \le a_i \le 10^9$），其中 $a_i$ 表示第 $i$ 个主题对老师的有趣程度。

第三行包含 $n$ 个整数 $b_1, b_2, \dots, b_n$（$1 \le b_i \le 10^9$），其中 $b_i$ 表示第 $i$ 个主题对学生的有趣程度。

## 输出格式

输出一个整数，表示“好对”主题的数量。

## 输入输出样例 #1

### 输入 #1

```
5
4 8 2 6 2
4 5 4 1 3
```

### 输出 #1

```
7
```

## 输入输出样例 #2

### 输入 #2

```
4
1 3 2 4
1 3 2 4
```

### 输出 #2

```
0
```

## 说明/提示

由 ChatGPT 4.1 翻译

## 题意重述

给定两个长度为 $n$ 的数组 $a, b$，找出有几组 $(i, j)$ 满足 $i < j 且 a_i + a_j < b_i + b_j$。

## Solution

### （1）Official Solution $O(nlogn)$

分离变量，即 $(a_i - b_i) + (a_j - b_j) > 0$, 令 $c_i = a_i - b_i$， 则转化为找有多少对 $(i, j)$ 满足 $i < j 且 c_i + c_j > 0$。

这时考虑对撞指针，即先构造出 $c_i$，随后排序，根据加法满足交换律的特性，令 $l=1,r=n$
① $c[l] + c[r] > 0$，则有 $r - l$ 个贡献，此时令 $r--$。
② $c[l] + c[r] \le 0$，$c[l]$ 拖后腿，此时令 $l++$。

这里总结了双指针的 Tricks。
[[双指针（2-Pointers）Tricks]]
Code 略
### （2）CDQ - Divide and Conquer $O(nlog^2n)$

自己摸的，偏序计数问题，考虑CDQ分治。转化问题为$a_i - b_i < b_j < a_j$，令不等式左右两边为新的 $a_i,b_j$，偏序约束为$i<j且a_i<b_j$。

CDQ分治的一般处理策略：
	① 后续处理跨区间，即`CDQ(l, mid), CDQ(mid + 1, r);`
	② 跨区间天然满足 $i < j$，只需对 $a[l...mid],b[mid+1...r]$ 进行排序，双指针跨区间统计答案即可。

## Code

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 2e5 + 5;
int a[N], b[N], c[N];
int ans = 0;
void CDQ(int l, int r){
    if(l >= r) return ;
    int mid = (l + r) >> 1;
    CDQ(l, mid); CDQ(mid + 1, r);
    vector<int> A, B;
    for(int i = l; i <= mid; ++i) A.push_back(a[i]);
    for(int i = mid + 1; i <= r; ++i) B.push_back(b[i]);
    sort(A.begin(), A.end()); sort(B.begin(), B.end());
    int cnt = 0;
    int j = 0;
    for(int x : A){
        while(j < B.size() && B[j] < x) j++;
        cnt += j;
    }
    ans += cnt;
}

signed main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int n; cin >> n;
    for(int i = 1; i <= n; ++i) cin >> a[i], c[i] = a[i];
    for(int i = 1; i <= n; ++i) cin >> b[i];
    for(int i = 1; i <= n; ++i) a[i] -= b[i], b[i] -= c[i];
    CDQ(1, n);
    cout << ans << endl;
    return 0;
}
```