设

$$
S(n)=1\oplus 2\oplus 3\oplus \cdots \oplus n
$$

则有：

$$
S(n)=
\begin{cases}
n, & n\bmod 4=0 \\
1, & n\bmod 4=1 \\
n+1, & n\bmod 4=2 \\
0, & n\bmod 4=3
\end{cases}
$$

常搭配 $\operatorname{xor}$ 前缀和与自反性使用。

---
## CF 424C Magic Formulas

题目给出：

$$
q_i=p_i\oplus(i\bmod 1)\oplus(i\bmod 2)\oplus\cdots\oplus(i\bmod n)
$$

要求计算：

$$
Q=q_1\oplus q_2\oplus\cdots\oplus q_n
$$

直接展开是一个 $n\times n$ 的异或矩阵，复杂度为 $O(n^2)$，不可接受。

---

### Trick：按列重排异或矩阵

先把所有 $q_i$ 展开：

$$
Q=(p_1\oplus p_2\oplus\cdots\oplus p_n)
\oplus
\bigoplus_{i=1}^{n}\bigoplus_{j=1}^{n}(i\bmod j)
$$

由于异或满足交换律和结合律，可以交换求异或的顺序：

$$
\bigoplus_{i=1}^{n}\bigoplus_{j=1}^{n}(i\bmod j)
=
\bigoplus_{j=1}^{n}\bigoplus_{i=1}^{n}(i\bmod j)
$$

也就是固定模数 $j$，分析这一列：

$$
(1\bmod j)\oplus(2\bmod j)\oplus\cdots\oplus(n\bmod j)
$$

原题是按行定义 $q_i$，但优化时要按列统计。

### 固定模数 $j$ 的列贡献

固定一个模数 $j$，考虑：

$$
T(j)=(1\bmod j)\oplus(2\bmod j)\oplus\cdots\oplus(n\bmod j)
$$

序列呈周期性：

$$
1,2,3,\dots,j-1,0,\ 1,2,3,\dots,j-1,0,\dots
$$

令：$n=qj+r$。

其中：$q=\left\lfloor\frac{n}{j}\right\rfloor,\qquad r=n\bmod j$。

完整周期为：$1,2,\dots,j-1,0$。

因为 $0$ 对异或没有影响，所以一个完整周期的异或值是：$S(j-1)$。

完整周期出现 $q$ 次。由于：$X\oplus X=0$，所以完整周期只需要判断 $q$ 的奇偶性。

残尾是：$1,2,\dots,r$，尾巴贡献为：$S(r)$。

因此：

$$
T(j)=
\begin{cases}
S(r), & q\equiv 0\pmod 2\\
S(j-1)\oplus S(r), & q\equiv 1\pmod 2
\end{cases}
$$

其中：

$$
q=\left\lfloor\frac{n}{j}\right\rfloor,\qquad r=n\bmod j
$$

最终答案为：

$$
Q=
\left(p_1\oplus p_2\oplus\cdots\oplus p_n\right)
\oplus
\left(T(1)\oplus T(2)\oplus\cdots\oplus T(n)\right)
$$

其中每个 $T(j)$ 可以 $O(1)$ 求出，因此总复杂度为：$O(n)$。

### 易错点

1. 余数必须是 $n\bmod j$。固定模数 $j$ 时，分析的是：$1\bmod j,\ 2\bmod j,\dots,\ n\bmod j$。

所以：$r=n\bmod j$，不能写成：$r=j\bmod n$。

因为 $j\le n$ 时，$j\bmod n$ 基本等于 $j$，会把尾巴长度算错。

2. 完整周期贡献是 $S(j-1)$，不是 $S(n-1)$。固定模数 $j$ 时，一个完整周期是：$1,2,\dots,j-1,0$。

所以完整周期贡献为：$S(j-1)$，而不是：$S(n-1)$。

3. 尾巴是 $1,2,\dots,r$，不是 $0,1,\dots,r$。因为序列从：$1\bmod j$

开始，而不是从：$0\bmod j$ 开始。当：$n=qj+r$ 时，尾巴是：$1,2,\dots,r$

所以尾巴贡献是：$S(r)$。

## 本质

1. 异或出现偶数次会抵消；
2. 二维异或矩阵可以交换统计顺序；
3. 固定模数后，取模序列具有周期结构，可以用连续异或前缀 $S(x)$ 快速求出。

---
