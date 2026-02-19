### 我们知道 朴素模拟应用快速排序维护数组中第k小的元素 其时间复杂度为$O(n^2logn)$ 在1e5规模的数组中可能TLE
#### 此时我们便需要一种高效的数据结构以$O(1)$ 的查询方式维护,此时注意到priority_queue可以做到 其最终的时间复杂度为$O(nlogn)$ 

#### 其实现需要维护一个大顶堆和一个小顶堆,以达到数据对冲的效果
如图
![[wgonylvh.webp]]
```
priority_queue<int, vector<int>, greater<int>> pq_mi; //小顶堆
priority_queue<int> pq_mx; //大顶堆
```
#### 假设我们以动态维护数组中第k大的元素为例 此时需要查找的元素位于小顶堆中
先向大顶堆中push一个0, 或者是后续进行pq_mx.empty()判断
#### 随后我们判断即将入堆的元素x与两个堆顶的关系 如果x比大顶堆的堆顶大, 把x放进小顶堆, 反之同理
```
if(x >= pq_mx.top()) pq_mi.push(x);
else pq_mx.push(x);
```
#### 这个时候判断pq_mi.size()和k的大小关系 如果pq_mi的大小比k要小,说明元素不够,要从大顶堆里面取一个过来 反之同理
```
if(pq_mi.size() < k) pq_mi.push(pq_mx.top()), pq_mx.pop();
else if(pq_mi.size() > k) pq_mx.push(pq_mi.top()), pq_mi.pop();
```
#### 最后答案即为pq_mi.top();

https://www.luogu.com.cn/problem/P7072
https://www.luogu.com.cn/problem/P3871
https://www.luogu.com.cn/problem/P1801