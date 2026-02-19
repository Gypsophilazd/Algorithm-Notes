
 在一条长度为 $B$ 的线段 $l$ 上，等距截取一些端点，两个相邻端点的距离为 $d\le D$。  
每个端点处，都作了一条长度为 $H$ 的线段，且与线段 $l$ 垂直。在两个相邻竖线之间，悬挂一条全等的抛物线，所有抛物线的弧长总和为 $L$。  

给定 $H,D,B,L$，求一个合法的 $d$（$d\le D$），使得抛物线的最低点到线段 $l$ 的垂直距离 $y$ 尽可能小（$y\ge0$），输出这个 $y$，保留两位小数。

## Solution

1. **分段与杆数**  
   - 最少杆数   $$
       n = \left\lceil \frac{B}{D} \right\rceil
         = \frac{B-1}{D} + 1.
     $$
   - 每段长度  
     $$
       w = \frac{B}{n},
       \quad \text{总电线长度平摊到每段后 }L \gets \frac{L}{n}.
     $$

2. **抛物线模型与弧长公式**  
   - 设抛物线向下开口：  
     $$
       y = a x^2.
     $$
   - 若“抬高”高度为 $h$，则顶点从杆顶高度 $H$ 下降到 $H-h$，对应  
     $$
       a = \frac{4h}{w^2}.
     $$
   - 弧长微元：  
     $$
       ds = \sqrt{1 + \bigl(y'(x)\bigr)^2}\,dx
          = \sqrt{1 + (2ax)^2}\,dx.
     $$
   - 抛物线关于 $x=0$ 对称，计算半段 $[0,\tfrac w2]$ 的弧长，再乘以 2。
2. **自适应 Simpson 计算弧长**  
   - 单步 Simpson 公式：  
$$S(l,r) = \frac{r-l}{6}\Bigl[f(l) + 4f\bigl(\tfrac{l+r}{2}\bigr) + f(r)\Bigr],\quad f(x)=\sqrt{1+4a^2x^2}.
$$
   - 如果一次 Simpson 估值误差超限，则对子区间递归细分。

4. **二分搜索“抬高”高度**  
   - 在区间 $h\in[0,H]$ 上做二分：  
     - 中点 $m=(l+r)/2$，计算此时单段弧长 $\mathrm{res}$。  
     - 若 $\mathrm{res}\le L$，说明可以再抬高，令 $l=m$；否则 $r=m$。  
   - 迭代至区间长度 $<\varepsilon$，答案即 $y = H - l$。
## Code

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
const double eps = 1e-9;
double D, H, B, L, w, a;

// 被积函数：sqrt(1 + (y')^2)
double f(double x) {
    return sqrt(1 + 4 * a * a * x * x);
}

// 单步 Simpson
double simpson(double l, double r) {
    double mid = (l + r) / 2;
    return (f(l) + 4 * f(mid) + f(r)) * (r - l) / 6.0;
}

// 自适应 Simpson
double calc(double l, double r, double eps, double res) {
    double mid = (l + r) / 2;
    double x = simpson(l, mid);
    double y = simpson(mid, r);
    if (fabs(x + y - res) <= 15 * eps) return x + y + (x + y - res) / 15;
    return calc(l, mid, eps / 2, x) + calc(mid, r, eps / 2, y);
}

// 检查给定抬高 h 时，单段弧长是否 <= L
bool check(double h) {
    a = 4 * h / (w * w);
    double half = calc(0, w / 2, eps, simpson(0, w / 2));
    double res = half * 2;
    return res <= L;
}

signed main() {
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int t; cin >> t;
    int T = 0;
    while(t--){
        cin >> D >> H >> B >> L;
        int n = (B - 1) / D + 1;
        w = B / n, L = L / n;
        double l = 0, r = H;
        while(r - l > eps){
            double mid = (l + r) / 2;
            if(check(mid)) l = mid;
            else r = mid;
        }
        printf("Case %d:\n%.2lf\n", ++T, H-l);
        if(t) puts("");
    }
    return 0;
}
