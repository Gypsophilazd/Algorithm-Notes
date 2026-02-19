#### 注:二维单调队列典题
## 题目描述

为了增添公园的景致，现在需要在公园中修筑一个花坛，同时在花坛四周修建一片绿化带，让花坛被绿化带围起来。

如果把公园看成一个 $M\times N$ 的矩形，那么花坛可以看成一个 $C\times D$ 的矩形，绿化带和花坛一起可以看成一个 $A\times B$ 的矩形。

如果将花园中的每一块土地的“肥沃度”定义为该块土地上每一个小块肥沃度之和，那么，绿化带的肥沃度为 $A\times B$ 块的肥沃度减去 $C\times D$ 块的肥沃度。

为了使得绿化带的生长得旺盛，我们希望绿化带的肥沃度最大。

## 输入格式

第一行有六个正整数 $M,N,A,B,C,D$。

接下来一个 $M\times N$ 的数字矩阵，其中矩阵的第 $i$ 行 $j$ 列元素为一个整数 $x_{i,j}$，表示该花园的第 $i$ 行第 $j$ 列的土地 “肥沃度”。

## 输出格式

一个正整数，表示绿化带的最大肥沃程度。

## 输入输出样例 #1

### 输入 #1

```
4 5 4 4 2 2
20 19 18 17 16
15 14 13 12 11
10 9 8 7 6
5 4 3 2 1
```

### 输出 #1

```
132
```

## 说明/提示

对于 $30\%$ 的数据，$1\leq M,N\leq 50$。

对于 $100\%$ 的数据，$1\leq M,N\leq 1000$，$1\leq A\leq M$，$1\leq B\leq N$，$1\leq C\leq A-2$，$1\leq D\leq B-2$，$1\leq x_{i,j}\leq 100$。

# Solution
二维的单调队列做法：先就某一维上对原始数据进行处理，在对另一维上对前一维的数据进行处理，就得到了所需要的东西。

回到这题，也是同样的做法，下边是我解题时的思路：

1. 预处理出以A\[i,j]、B\[i,j]分别为右下角的花坛(C*D)、大矩形(A*B)的肥沃度和
    
2. 按行，利用单调队列求出P\[i,j]=以A\[i, j-b+1+d→j-1]中的最小值，即以(i, j-b+1+d→j-1)为右下角的花坛肥沃度的最小值。

其中 j - (b - 1) +d 为花坛的左边界, 故q\[hh] 即单调队列的队首需要>j - (b - 1) + d + 1
小于则需要hh++;
    
3. 按列，利用单调队列求出Q\[i,j]=以P\[i-a+1+c→i-1, j]中的最小值，即以(i-a+1+c→i-1, j-b+1+d→j-1) 为右下角的花坛肥沃度的最小值，显然，这些花坛包含在了以(i,j)为右下角的大矩形中；
    
4. 显然B\[i,j]-Q\[i,j]即为以(i,j)为右下角的绿化带的最大肥沃值。
    

时间复杂度 O(nm)。代码实现中n、m与题意是反的。

原来的区间\[j-b+1+d, j-1]是对于j来说的；但此处我们在j的位置时更新j+1，所以范围调整为\[j-b+2+d, j]。下同。

## AC Code
```
#include <bits/stdc++.h>
using namespace std;
const int N = 1005;
int q[N];
int s[N][N],A[N][N],B[N][N],P[N][N],Q[N][N];
int hh = 1, tt = 0;
  
int main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int m, n, a, b, c, d; cin >> n >> m >> a >> b >> c >> d;
    for(int i = 1; i <= n; ++i){
        for(int j = 1; j <= m; ++j) {
            cin >> s[i][j];
            s[i][j] += s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1];
        }
    }

    for(int i = c + 1; i < n; ++i){
        for(int j = d + 1; j < m; ++j) A[i][j] = s[i][j] - s[i - c][j] - s[i][j - d] + s[i - c][j - d];
    }
    for(int i = a; i <= n; ++i){
        for(int j = b; j <= m; ++j) B[i][j] = s[i][j] - s[i - a][j] - s[i][j - b] + s[i - a][j - b];
    }

    for(int i = c + 1; i < n; ++i){
        hh = 0, tt = 1;
        for(int j = d + 1; j < m; ++j){
            while(hh <= tt && q[hh] < j - b + d + 2) hh++;
            while(hh <= tt && A[i][q[tt]] >= A[i][j]) tt--;
            q[++tt] = j;
            if(j >= b - 1) P[i][j + 1] = A[i][q[hh]];
        }
    }

  
    for(int j = b; j <= m; ++j){
        hh = 0; tt = 1;
        for(int i = c + 1; i < n; ++i){
            while(hh <= tt && q[hh] < i - a + c + 2) hh++;
            while(hh <= tt && P[q[tt]][j] >= P[i][j]) tt--;
            q[++tt] = i;
            if(i >= a - 1) Q[i + 1][j] = P[q[hh]][j];
        }
    }
    long long ans = 0;
    for(int i = a; i <= n; ++i){
        for(int j = b; j <= m; ++j){
            ans = max(ans, 1ll * B[i][j] - Q[i][j]);
        }
    }
    cout << ans << endl;
    return 0;
}
```