# 一、字典树（Trie）

## 1.定义

Trie 是一种能够快速插入和查询字符串的**多叉树**结构。节点的编号各不相同，根节点编号为 $0$，其他节点用来标识路径，还可以标记单词插入的次数。边表示字符。

Trie 维护字符串的集合，支持两种操作：
	1.向集合中**插入**一个字符串，`void insert(char *s)`
	2.在集合中**查询**一个字符串，`int query(char *s)`
	
[Open: Pasted image 20260304130229.png](Note/ACM%E8%AE%AD%E7%BB%83/Tricks_and_Skills/images/e862d3e4482fab001baad55a14a5c80b_MD5.jpg)
![](Note/ACM%E8%AE%AD%E7%BB%83/Tricks_and_Skills/images/e862d3e4482fab001baad55a14a5c80b_MD5.jpg)
## 2.建字典树与查询

用一个儿子数组 `ch[p][j]` 存储从**节点 $p$ 沿着 $j$ 这条边走到的子节点序号**。
	边为 $26$ 个小写字母(a-z) 对应的映射值 $0-25$，每个节点最多可以有 $26$ 个分叉。
	例如：`ch[0][2] = 1, ch[1][0] = 2, ch[2][19] = 3`
计数数组 `cnt[p]` 存储**以节点 $p$ 为结尾的单词的插入次数**。
节点编号 `idx`。

### 建树流程

1. 空 Trie 只有一个根节点，编号 $0$。
2. 从根开始插，枚举字符串的每个字符，如果有儿子，则 $p$ 指针走到儿子，如果没儿子，先创建儿子再把 $p$ 移动到儿子。
3. 在单词结束点记录插入次数。

```cpp
char s[N]; // N represents the length of the longest word;
int ch[N][26], cnt[N], idx;
void trie_input(char *s){
	int p = 0;
	for(int i = 0; s[i]; ++i){
		int j = s[i] - 'a';
		if(!ch[p][j]) ch[p][j] = ++idx;
		p = ch[p][j];
	}
	cnt[p]++;
}
```

### 查询流程

1. 从根开始查，扫描字符串。
2. 有字母 `s[i]`，则走下来，能走到词尾，就返回插入次数。
3. 如果没有字母 `s[i]`，返回 $0$。

```cpp
int trie_output(char *s){
	int p = 0;
	for(int i = 0 ; s[i]; ++i){
		int j = s[i] - 'a';
		if(!ch[p][j]) return 0;
		p = ch[p][j];
	}
	return cnt[p];
}
```

## 3.例题

Trie + DFS [Luogu P4407 JSOI2009 电子字典](Luogu%20P4407%20JSOI2009%20电子字典.md)
Trie + LCA [Luogu P6088 JSOI2015 字符串树](Luogu%20P6088%20JSOI2015%20字符串树.md)
