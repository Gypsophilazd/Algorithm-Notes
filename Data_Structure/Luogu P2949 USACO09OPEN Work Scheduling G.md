## 题目描述

Farmer John has so very many jobs to do! In order to run the farm efficiently, he must make money on the jobs he does, each one of which takes just one time unit.

His work day starts at time 0 and has 1,000,000,000 time units (!).  He currently can choose from any of N (1 <= N <= 100,000) jobs

conveniently numbered 1..N for work to do. It is possible but

extremely unlikely that he has time for all N jobs since he can only work on one job during any time unit and the deadlines tend to fall so that he can not perform all the tasks.

Job i has deadline D\_i (1 <= D\_i <= 1,000,000,000). If he finishes job i by then, he makes a profit of P\_i (1 <= P\_i <= 1,000,000,000).

What is the maximum total profit that FJ can earn from a given list of jobs and deadlines?  The answer might not fit into a 32-bit integer.

## 输入格式

\* Line 1: A single integer: N

\* Lines 2..N+1: Line i+1 contains two space-separated integers: D\_i and P\_i

## 输出格式

\* Line 1: A single number on a line by itself that is the maximum possible profit FJ can earn.

## 输入输出样例 #1

### 输入 #1

```
3 
2 10 
1 5 
1 7
```

### 输出 #1

```
17
```

## 说明/提示

Complete job 3 (1,7) at time 1 and complete job 1 (2,10) at time 2 to maximize the earnings (7 + 10 -> 17).

## Solution
首先注意"The answer might not fit into a 32-bit integer." 不开longlong见祖宗
题意转换:给定总任务数n, 给定每一天的截止时间和利润,要求完成所有任务的最大利润
属于-**满足一定操作次数的最大价值/最小利润问题**,所以我们考虑**反悔贪心**

解决此类问题的一般步骤,建立一个大根堆,即`priority_queue<int, vector<int>, greater<int>> pq;` 
(1)先将任务塞进结构体,按照时间排序(因为我们需要完成最早需要完成的任务)
大根堆的 `pq.size()`即为当前的总天数, **我们把利润塞进堆里** 这样堆顶就是最小的利润.
(2)我们遍历每个按照时间排序的任务,如果当前的截止时间d <= pq.size(), 我们就考虑一下是否需要替换, 判断堆顶的最小利润与当前利润的关系,如果当前利润大于堆顶最小利润,先把ans减掉堆顶(反悔操作), 再把当前堆顶弹出, 把当前利润塞进去, ans重新加上去;
(3)否则呢就直接把当前利润push进去 ans直接加上去即可
```
#include <bits/stdc++.h>
#define int long long
using namespace std;
const int N = 1e5 + 5;
struct Node{
    int d, p;
    bool operator < (const Node &A) const{
        return d < A.d;
    }
}m[N];
priority_queue<int, vector<int>, greater<int>> pq;

signed main(){
    int n; cin >> n;
    int ti = 0, ans = 0;
    for(int i = 1; i <= n; ++i) cin >> m[i].d >> m[i].p;
    sort(m + 1, m + 1 + n);
    for(int i = 1; i <= n; ++i){
        if(m[i].d <= pq.size()){
            if(m[i].p > pq.top()){
                ans -= pq.top();
                pq.pop();
                pq.push(m[i].p);
                ans += m[i].p;
            }
        }
        else{
            pq.push(m[i].p);
            ans += m[i].p;
        }
    }
    cout << ans << endl;
    return 0;
}
```