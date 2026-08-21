
*注：本场ABCD简单*
## E. Graph Composition


**题意：** 给定两个拥有相同 $n$ 个点的无向图 $F,G$。
只能对 $F$ 添加或删除边。
目标是使对于任意 $u,v$：
$$
u,v\text{ 在 }F\text{ 中连通}
\iff
u,v\text{ 在 }G\text{ 中连通}
$$
求最少操作数。


### Solution

定义：

$$
P_F(u,v)=[u,v\text{ 在 }F\text{ 中连通}]
$$
$$
P_G(u,v)=[u,v\text{ 在 }G\text{ 中连通}]
$$
题目要求：
$$
\forall u,v,\quad P_F(u,v)\iff P_G(u,v)
$$
因此等价于：

$$
\boxed{F\text{ 和 }G\text{ 的连通块划分完全相同}}
$$
一旦把题意写成这个形式，问题直接变成 DSU。

第一想法即删除 F 中非法跨块边
首先用 $G$ 建立 DSU。
对于 $F$ 中边 $(u,v)$，如果：
$$
root_G(u)\neq root_G(v)
$$
说明该边连接了两个本应在 $G$ 中不连通的 component，因此这条边必须删除：`ans++`
否则这条边可以保留，并在 $F$ 的 DSU 中：
```cpp
uniteF(u, v);
```

第二步是补齐 G component 内部的 F 连通性
删除所有非法跨块边后：

> $F$ 的每个连通块一定完全包含在某个 $G$ 的连通块中。

但一个 $G$ component 可能在 $F$ 中仍然被切成多个小 component。
遍历 $G$ 的边：

```cpp
if(rootF(u) != rootF(v)){
    uniteF(u, v);
    ans++;
}
```

每添加一条边，$F$ 的 component 数量减少 1，最终 $F$ 与 $G$ 的 component partition 完全一致。
