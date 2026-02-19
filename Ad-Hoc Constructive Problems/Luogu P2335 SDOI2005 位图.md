## 题目描述

现在我们给出一个 $n\ \times m$ 的单色位图，且该图中至少含有一个白色的像素。我们用 $(i,j)$ 来代表第 $i$ 行第 $j$ 列的像素，并且定义两点 $p_1=(i_1,j_1)$ 和 $p_2=(i_2,j_2)$ 之间的距离为：

$$d(p_1,p_2)=|i_1-i_2|+|j_1-j_2|$$

### 任务

请写一个程序，读入该位图，并对于每个像素，计算出离该像素最近的白色像素与它的距离。把结果输出。

## 输入格式

第一行包括两个用空格分开的整数 $n$ 和 $m$，$1 \le n \le 150$，$1 \le m \le 150$。

以下的 $n$ 行每行包括一个长度为 $m$ 的整数为 $0$ 或 $1$，在第 $i+1$ 行的第 $j$ 个字符如果为 $1$，那么表示像素 $(i,j)$ 为白的，否则为黑的。

## 输出格式

输出一个 $n\ \times m$ 的数表，其中的第 $i$ 行的第 $j$ 个数字为 $f(i,j)$ 表示像素 $(i,j)$ 到最近的白色像素的距离。

## 输入输出样例 #1

### 输入 #1

```
3 4
0 0 0 1
0 0 1 1
0 1 1 0
```

### 输出 #1

```
3 2 1 0
2 1 0 0
1 0 0 1
```


### Solution
使用广度优先搜索(BFS) 遍历最短路(类似dijkstra)
1. **输入处理**：读取矩阵并初始化所有白色像素点的距离为0，其他点初始化为-1。
2. **队列初始化**：将白色像素点加入队列作为BFS的起点。
3. **BFS扩散**：从队列中取出点，检查四个方向。若相邻点未访问过，则更新其距离并加入队列。
4. **结果输出**：遍历距离矩阵，输出每个像素点的最短距离。
时间复杂度为O(nm)

### AC Code
```
#include <bits/stdc++.h>
using namespace std;
const int N = 155;
int dist[N][N];
int dx[] = {-1, 1, 0, 0};
int dy[] = {0, 0, -1, 1};
queue<pair<int, int>> q;
int main() {
    int n, m;
    cin >> n >> m;
    memset(dist, -1, sizeof(dist));
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < m; ++j) {
            int x; cin >> x;
            if (x == 1) {
                dist[i][j] = 0;
                q.push({i,  j});
            }
        }
    }
    while (!q.empty())  {
        auto [x, y] = q.front(); q.pop();
        for (int d = 0; d < 4; ++d) {
            int nx = x + dx[d], ny = y + dy[d];
            if (nx >= 0 && nx < n && ny >= 0 && ny < m && dist[nx][ny] == -1) {
                dist[nx][ny] = dist[x][y] + 1;
                q.push({nx,  ny});
            }
        }
    }
    for (int i = 0; i < n; ++i){
        for (int j = 0; j < m; ++j) cout << dist[i][j] << " ";
        cout << endl;
    }
    return 0;
}
```