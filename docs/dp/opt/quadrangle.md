author: Marcythm, zyf0726, hsfzLZH1, MingqiHuang, Ir1d, greyqz, billchenchina, Chrogeek, StudyingFather, NFLSCode

## (罗勇军版本)

### 应用场合

有一些常见的DP问题，通常是区间DP问题，它的状态转移方程是：

`dp[i][j]=min(dp[i][k]+dp[k+1][j]+w[i][j])`

其中 i <= k < j，初始值dp\[i\]\[i\]已知。min()也可以是max()。

方程的含义是：

1. dp\[i\]\[j\]表示从i状态到j状态的最小花费。题目一般是求dp\[1\]\[n\]，即从起始点1到终点n的最小花费。
2. `dp[i][k]+dp[k+1][j]`体现了递推关系。k在i和j之间滑动，k有一个最优值，使得dp\[i\]\[j\]最小。
3. w\[i\]\[j\]的性质非常重要。w\[i\]\[j\]是和题目有关的费用，如果它满足四边形不等式和单调性，那么用DP计算dp的时候，就能进行四边形不等式优化。

这类问题的经典的例子是“石子合并”，它的转移矩阵就是上面的dp\[i\]\[j\]，w\[i\]\[j\]是从第i堆石子到第j堆石子的总数量。

???+note "[石子合并](https://www.luogu.com.cn/problem/P1880)"
    **题目描述：** 有n堆石子排成一排，每堆石子有一定的数量。将n堆石子并成为一堆。每次只能合并相邻的两堆石子，合并的花费为这两堆石子的总数。经过n-1次合并后成为一堆，求总的最小花费。
    
    **输入：** 测试数据第一行是整数n，表示有n堆石子。接下来的一行有n个数，分别表示这n堆石子的数目。
    
    **输出：** 总的最小花费。
    
    输入样例：
    
    3
    
    2 4 5
    
    输出样例：
    
    17
    
    提示：样例的计算过程是：第一次合并2+4=6；第二次合并6+5=11；总花费6+11=17。

在阅读后面的讲解时，读者可以对照“石子合并”这个例子来理解。注意，石子合并有多种情况和解法，详情见本文的例题“洛谷P1880石子合并”。

dp\[i\]\[j\]是一个转移矩阵，如何编码填写这个矩阵？复杂度是多少？如果直接写i、j、k的3层循环，复杂度O(n^3^)。

注意3层循环的写法。dp\[i\]\[j\]是大区间，它从小区间dp\[i\]\[k\]和dp\[k+1\]\[j\]转移而来，所以应该先计算小区间，再逐步扩展到大区间。

```cpp
for(int i=1; i<=n; i++)
    dp[i][i] = 0;                       //初始值
for(int len = 2; len <= n; len++)       //len：从小区间扩展到大区间
	for(int i = 1; i <= n-len+1; i++){  // 区间起点i
   		int j = i + len - 1;            // 区间终点j
   		for(int k = i; k < j; k++) //大区间[i,j]从小区间[i,k]和[k+1,j]转移而来
   			dp[i][j] = min(dp[i][j], dp[i][k] + dp[k + 1][j] + w[i][j]);
	}
```

### 四边形不等式优化

只需一个简单的优化操作，就能把上面代码的复杂度变为O(n^2^)。这个操作就是把循环i ≤ k < j改为：

`s[i][j−1]≤k≤s[i+1][j]`

其中s\[i\]\[j\]记录从i到j的最优分割点。在计算dp\[i\]\[j\]的最小值时得到区间\[i,j\]的分割点k，记录在s\[i\]\[j\]中，用于下一次循环。

这个优化被称为四边形不等式优化。下面给出优化后的代码，优化见注释的几行代码。

```cpp
for(i = 1;i <= n;i++){
    dp[i][i] = 0;                        
    s[i][i] = i;                        //s[][]的初始值
}
for(int len = 2; len <= n; len++)       
	for(int i = 1; i <= n-len+1; i++){   
   		int j = i + len - 1;            
   		for(k = s[i][j - 1]; k <= s[i + 1][j]; k++){   //缩小循环范围
            if(dp[i][j] > dp[i][k] + dp[k + 1][j] + w[i][j]){  //是否更优
   			      			      dp[i][j] = dp[i][k] + dp[k + 1][j] + w[i][j];
               s[i][j] = k;                 //更新最佳分割点
            }
   		}
	}
```

???+warning "最大值"
    这一题除了求最小值，还求最大值。虽然最大值也用DP求解，但是它不满足反四边形不等式的单调性要求，不能优化。而且也没有必要优化，可以用简单的推理得到：区间\[i,j\]的最大值，等于区间\[i,j−1\]和\[i+1,j\]中的最大值加上w(i,j)。

代码的复杂度是多少？

代码中i和k这2个循环，优化前是O(n^2^)的。优化后，每个i内部的k的循环次数是`s[i+1][j]−s[i][j−1]`，其中j=i+len−1。那么：

i=1时，k循环`s[2][len]−s[1][len−1]`次。

i=2时，k循环`s[3][len+1]−s[2][len]`次。

…

i=n−len+1时，k循环`s[n−len+2][n]−s[n−len+1][n+1]`次。

上述次数相加，总次数：
 `s[2][len]−s[1,len−1]+s[3][len+1]−s[2,len]+…+s[n+1,n]−s[n][n]`

=`s[n−len+2][n]−s[1][len−1]` < n

i和k循环的时间复杂度优化到了O(n)。总复杂度从O(n3)优化到了O(n^2^)。

下图给出了四边形不等式优化的效果，s1是区间\[i,j−1\]的最优分割点，s2是区间\[i+1,j\]的最优分割点。

![](../images/dp-27.png)

四边形不等式优化效果

读者对代码可能有2个疑问：

（1）为什么能够把i <= k < j缩小到 `s[i][j−1]≤k≤s[i+1][j]`？

（2）`s[i][j−1]≤s[i+1][j]`成立吗？

下面几节给出四边形不等式优化的正确性和复杂度的严谨证明，解答了这2个问题。

### 四边形不等式定义和单调性定义

在四边形不等式DP优化中，对于w，有2个关键内容：四边形不等式定义、单调性。

（1）四边形不等式定义1：设w是定义在整数集合上的二元函数，对于任意整数i ≤ i′ ≤ j ≤ j′，如果有 w(i,j)+w(i′,j′) ≤ w(i,j′)+w(i′,j)，则称w满足四边形不等式。

四边形不等式可以概况为：两个交错区间的w和，小于等于小区间与大区间的w和。

为什么被称为“四边形”？把它变成一个几何图，画成平行四边形，见下面图中的四边形i′ijj′。图中对角线长度和ij+i′j′大于平行线长度和ij′+i′j，这与四边形的性质是相反的，所以可以理解成“反四边形不等式”。请读者注意，这个“四边形”只是一个帮助理解的示意图，并没有严谨的意义。也有其他的四边形画法，下面这种四边形是储枫论文中的画法。当中间两个点i′=j时，四边形变成了一个三角形。

![](../images/dp-28.png)

四边形不等式 w(i, j) + w(i', j') ≤ w(i, j') + w(i', j)

定义1的特例是定义2。

（2）四边形不等式定义2：对于整数i < i+1 ≤ j < j+1，如果有 w(i,j)+w(i+1,j+1) ≤ w(i,j+1)+w(i+1,j)，称w满足四边形不等式。

定义1和定义2实际上是等价的，它们可以互相推导。

（3）单调性：设w是定义在整数集合上的二元函数，如果对任意整数i ≤ i′ ≤ j ≤ j′，有w(i,j′) ≥ w(i′,j)，称w具有单调性。

单调性可以形象地理解为，如果大区间包含于小区间，那么大区间的w值超过小区间的w值。

![](../images/dp-29.png)

w的单调性w(i, j') ≥ w(i', j)

在石子合并问题中，令w\[i\]\[j\]等于从第i堆石子加到第j堆石子的石子总数。它满足四边形不等式的定义、单调性：

`w[i][j′]≥w[i′][j]`，满足单调性；

`w[i][j]+w[i′][j′]=w[i][j′]+w[i′][j]`，满足四边形不等式定义。

利用w的四边形不等式、单调性的性质，可以推导出四边形不等式定理，用于DP优化。

### 四边形不等式定理（Knuth-Yao DP Speedup Theorem）

在储枫的论文中，提出并证明了四边形不等式定理。

四边形不等式定理：如果w(i,j)满足四边形不等式和单调性，则用DP计算dp\[\]\[\]的时间复杂度是O(n^2^)的。

这个定理是通过下面2个更详细的引理来证明的。

引理1：状态转移方程 `dp[i][j]=min(dp[i][k]+dp[k+1][j]+w[i][j])`，如果w\[i\]\[j\]满足四边形不等式和单调性，那么dp\[i\]\[j\]也满足四边形不等式。

引理2：记s\[i\]\[j\]=k是dp\[i\]\[j\]取得最优值时的k，如果dp满足四边形不等式，那么有`s[i][j−1]≤s[i][j]≤s[i+1][j]`，即`s[i][j−1]≤k≤s[i+1][j]`。

定理2直接用于DP优化，复杂度O(n^2^)。

### 例题

最优二叉搜索树是Knuth（高纳德）解决的经典问题，是四边形不等式优化的起源。
???+note "[Optimal Binary Search Tree](https://vjudge.net/problem/UVA-10304)"
    **题目描述：** 给定n个不同元素的集合S=(e1,e2,...,en)，有e1 < e2 < ... < en，把S的元素建一棵二叉搜索树，希望查询频率越高的元素离根越近。
    
    访问树中元素ei的成本cost(ei)等于从根到该元素结点的路径边数。给定元素的查询频率f(e1)，f(e2)，...，f(en)，定义一棵树的总成本是：
    
    f(e1)∗cost(e1)+f(e2)∗cost(e2)+...+f(en)∗cost(en)
    
    总成本最低的树就是最优二叉搜索树。
    
    **输入：** 输入包含多个实例，每行一个。每行以1 ≤ n ≤ 250开头，表示S的大小。在n之后，在同一行中，有n个非负整数，它们表示元素的查询频率，0 ≤ f(ei) ≤ 100。
    
    **输出：** 对于输入的每个实例，输出一行，打印最优二叉搜索树的总成本。
    
    样例输入：
    
    1 5
    
    3 10 10 10
    
    3 5 10 20
    
    样例输出：
    
    0
    
    20
    
    20

二叉搜索树（BST）的特点是每个结点的值，比它的左子树上所有结点的值大，比右子树上所有值小。二叉搜索树的中序遍历，是从小到大的排列。第3个样例的最优二叉搜索树的形状见下图，它的总成本是5∗2+10∗1=20。

![](../images/dp-30.png)

二叉搜索树

题目给的元素已经按照从小到大排列，可以方便地组成一棵BST。

设dp\[i\]\[j\]是区间\[i,j\]的元素组成的BST的最小值。把区间\[i,j\]分成两部分\[i,k−1\]和\[k+1,j\]，k在i和j之间滑动。用区间\[i,j\]建立的二叉树，k是根结点。这是典型的区间DP，状态转移方程：

`dp[i][j]=min{dp[i][k−1]+dp[k+1][j]+w(i,j)−e[k]}`

w(i,j)是区间和， $w(i,j)=f_i + f_{i+1} + ... + f_j$ 。当把两棵左右子树连在根结点上时，本身的深度增加1，所以每个元素都多计算一次，这样就解决了cost(ei)的计算。最后，因为根节点k的层数是0，所以减去根节点的值e\[k\]。
w(i, j)符合四边形不等式优化的条件，所以dp[i][j]可以用四边形不等式优化。

### 习题

- [Tree Construction](http://acm.hdu.edu.cn/showproblem.php?pid=3516)
- [Lawrence](http://acm.hdu.edu.cn/showproblem.php?pid=2829)
- [Monkey Party](http://acm.hdu.edu.cn/showproblem.php?pid=3506)
- [诗人小G](https://www.luogu.com.cn/problem/P1912)
- [邮局](https://www.luogu.com.cn/problem/P4767)
- [Division](http://acm.hdu.edu.cn/showproblem.php?pid=3480)

## (OIWiki版本) 区间类（2D1D）动态规划中的应用

在区间类动态规划（如石子合并问题）中，我们经常遇到以下形式的 2D1D 状态转移方程：

$$
f_{l,r} = \min_{k=l}^{r-1}\{f_{l,k}+f_{k+1,r}\} + w(l,r)\qquad\left(1 \leq l < r \leq n\right)
$$

直接简单实现状态转移，总时间复杂度将会达到 $O(n^3)$，但当函数 $w(l,r)$ 满足一些特殊的性质时，我们可以利用决策的单调性进行优化。

-   **区间包含单调性**：如果对于任意 $l \leq l' \leq r' \leq r$，均有 $w(l',r') \leq w(l,r)$ 成立，则称函数 $w$ 对于区间包含关系具有单调性。
-   **四边形不等式**：如果对于任意 $l_1\leq l_2 \leq r_1 \leq r_2$，均有 $w(l_1,r_1)+w(l_2,r_2) \leq w(l_1,r_2) + w(l_2,r_1)$ 成立，则称函数 $w$ 满足四边形不等式（简记为“交叉小于包含”）。若等号永远成立，则称函数 $w$ 满足 **四边形恒等式**。

**引理 1**：若 $w(l, r)$ 满足区间包含单调性和四边形不等式，则状态 $f_{l,r}$ 满足四边形不等式。

定义 $g_{k,l,r}=f_{l,k}+f_{k+1,r}+w(l,r)$ 表示当决策为 $k$ 时的状态值，任取 $l_1\leq l_2\leq r_1\leq r_2$，记 $u=\mathop{\arg\min}\limits_{l_1\leq k < r_2}g_{k,l_1,r_2},v=\mathop{\arg\min}\limits_{l_2\leq k < r_1}g_{k,l_2,r_1}$ 分别表示状态 $f_{l_1,r_2}$ 和 $f_{l_2,r_1}$ 的最小最优决策点。

> 注意到，仅当 $l_2 < r_1$ 时 $v$ 才有意义。因此我们有必要单独考虑 $l_2 = r_1$ 的情形。

首先注意到若 $l_1 = l_2$ 或 $r_1 = r_2$ 时，显然成立。

考虑对区间长度 $r_2 - l_1$ 使用数学归纳法（$r_2 - r_1 = 0$ 时，显然成立）。

-   $l_1 < l_2 = r_1 < r_2$（证明过程需要 $w$ 满足区间包含单调性）

    1.  若 $u < r_1$ 则 $f_{l_1, r_1} \leq f_{l_1, u} + f_{u + 1, r_1} + w(l_1, r_1)$，由归纳法 $f_{u + 1, r_1} + f_{l_2, r_2} \leq f_{u + 1, r_2} + f_{l_2, r_1}$，两式相加再消去相同部分得到（下面最后一个不等式用到了 $w(l_1, r_1) < w(l_1, r_2)$）：

        $$
        f_{l_1, r_1} + f_{l_2, r_2} \leq f_{l_1, u} + f_{u + 1, r_2} + f_{l_2, r_1} + w(l_1, r_1) \leq f_{l_1, r_2} + f_{l_2, r_1}
        $$

    2.  若 $u \geq r_1$ 则 $f_{l_2, r_2} \leq f_{l_2, u} + f_{u + 1, r_2} + w(l_2, r_2)$ 由归纳法 $f_{l_1, r_1} + f_{l_2, u} \leq f_{l_1, u} + f_{l_2, r_1}$，两式相加再消去相同部分得到（下面最后一个不等式用到了 $w(l_2, r_2) < w(l_1, r_2)$）：

        $$
        f_{l_1, r_1} + f_{l_2, r_2} \leq f_{l_1, u} + f_{l_2, r_1} + f_{u + 1, r_2} + w(l_2, r_2) \leq f_{l_1, r_2} + f_{l_2, r_1}
        $$

-   $l_1 < l_2 < r_1 < r_2$（仅需要 $w$ 满足四边形不等式）

    1.  若 $u\leq v$，则 $l_1\leq u< r_1,\ l_2\leq v< r_2$，因此

        $$
        \begin{aligned}
            f_{l_1,r_1} \leq g_{u,l_1,r_1} &= f_{l_1,u} + f_{u+1,r_1} + w(l_1,r_1) \\
            f_{l_2,r_2} \leq g_{v,l_2,r_2} &= f_{l_2,v} + f_{v+1,r_2} + w(l_2,r_2)
        \end{aligned}
        $$

        再由 $u+1 \leq v+1 \leq r_1 \leq r_2$ 和归纳假设知

        $$
        f_{u+1,r_1} + f_{v+1,r_2} \leq f_{u+1,r_2} + f_{v+1,r_1}
        $$

        将前两个不等式累加，并将第三个不等式代入，可得

        $$
        \begin{aligned}
            f_{l_1,r_1} + f_{l_2,r_2} & \leq f_{l_1,u} + f_{l_2,v} + f_{u+1,r_1} + f_{v+1,r_2} + w(l_1,r_1) + w(l_2,r_2) \\
            & \leq g_{u,l_1,r_2} + g_{v,l_2,r_1} = f_{l_1,r_2} + f_{l_2,r_1}
        \end{aligned}
        $$

    2.  若 $v< u$，则 $l_1\leq v<r_1,l_2\leq u<r_2$，因此

        $$
        \begin{aligned}
        f_{l_1,r_1} \leq g_{v,l_1,r_1} &= f_{l_1,v} + f_{v+1,r_1} + w(l_1,r_1) \\
        f_{l_2,r_2} \leq g_{u,l_2,r_2} &= f_{l_2,u} + f_{u+1,r_2} + w(l_2,r_2)
        \end{aligned}
        $$

        再由 $l_1 \leq l_2 \leq v \leq u$ 和归纳假设知

        $$
        f_{l_1,v} + f_{l_2,u} \leq f_{l_1,u} + f_{l_2,v}
        $$

        将前两个不等式累加，并将第三个不等式代入，可得

        $$
        \begin{aligned}
        f_{l_1,r_1} + f_{l_2,r_2} & \leq f_{l_1,u} + f_{l_2,v} + f_{v+1,r_1} + f_{u+1,r_2} + w(l_1,r_2) + w(l_2,r_1) \\
        & \leq g_{u,l_1,r_2} + g_{v,l_2,r_1} = f_{l_1,r_2} + f_{l_2,r_1}
        \end{aligned}
        $$

综上所述，两种情形均有 $f_{l_1,r_1} + f_{l_2,r_2} \leq f_{l_1,r_2} + f_{l_2,r_1}$，即四边形不等式成立。

**定理 1**：若状态 $f$ 满足四边形不等式，记 $m_{l,r}=\min\{k:f_{l,r} = g_{k,l,r}\}$ 表示最优决策点，则有

$$
m_{l,r-1} \leq m_{l,r} \leq m_{l+1,r} \qquad (l + 1 < r)
$$

记 $u = m_{l,r},\ k_1=m_{l,r-1},\ k_2=m_{l+1,r}$，分情况讨论：

1.  若 $k_1>u$，则 $u+1 \leq k_1+1 \leq r-1 \leq r$，因此根据四边形不等式有

    $$
    f_{u+1,r-1} + f_{k_1+1,r} \leq f_{u+1,r} + f_{k_1+1,r-1}
    $$

    再根据 $u$ 是状态 $f_{l,r}$ 的最优决策点可知

    $$
    f_{l,u} + f_{u+1,r} \leq f_{l,k_1} + f_{k_1+1, r}
    $$

    将以上两个不等式相加，得

    $$
    f_{l,u} + f_{u+1,r-1} \leq f_{l,k_1}+f_{k_1+1,r-1}
    $$

    即 $g_{u,l,r-1} \leq g_{k_1,l,r-1}$，但这与 $k_1$ 是最小的最优决策点矛盾，因此 $k_1\leq u$。

2.  若 $u>k_2$，则 $l\leq l+1 \leq k_2\leq u$，根据四边形不等式可得

    $$
    f_{l,k_2} + f_{l+1,u} \leq f_{l,u} + f_{l+1, k_2}
    $$

    再根据 $k_2$ 是状态 $f_{l+1, r}$ 的最优决策点可知

    $$
    f_{l+1,k_2} + f_{k_2+1, r} \leq f_{l+1,u} + f_{u+1,r}
    $$

    将以上两个不等式相加，得

    $$
    f_{l,k_2}+f_{k_2+1,r} \leq f_{l,u} + f_{u+1,r}
    $$

    即 $g_{k_2,l,r} \leq g_{u,l,r}$，但这与 $u$ 是最小的最优决策点矛盾，因此 $u \leq k_2$。

因此，如果在计算状态 $f_{l,r}$ 的同时将其最优决策点 $m_{l,r}$ 记录下来，那么我们对决策点 $k$ 的总枚举量将降为

$$
\sum_{1\leq l<r\leq n} m_{l+1,r} - m_{l,r-1} = \sum_{i=1}^n m_{i,n} - m_{1,i}\leq n^2
$$

???+ note "核心代码"
    === "C++"
    
        ```cpp
        for (int len = 2; len <= n; ++len)  // 枚举区间长度
        for (int l = 1, r = len; r <= n; ++l, ++r) {
            // 枚举长度为len的所有区间
            f[l][r] = INF;
            for (int k = m[l][r - 1]; k <= m[l + 1][r]; ++k)
            if (f[l][r] > f[l][k] + f[k + 1][r] + w(l, r)) {
                f[l][r] = f[l][k] + f[k + 1][r] + w(l, r);  // 更新状态值
                m[l][r] = k;  // 更新（最小）最优决策点
            }
        }
        ```
    
    === "Python"
    
        ```python
        for len in range(2, n + 1): # 枚举区间长度
            r = len
            l = 1
            while(r <= n):
                # 枚举长度为len的所有区间
                r += 1
                l += 1
                f[l][r] = INF
                k = m[l][r - 1]
                while k <= m[l + 1][r]:
                    if f[l][r] > f[l][k] + f[k + 1][r] + w(l, r):
                        f[l][r] = f[l][k] + f[k + 1][r] + w(l, r) # 更新状态值
                        m[l][r] = k # 更新（最小）最优决策点
                    k += 1
        ```

### 基于分治的决策单调性优化

某些 dp 问题形式如下：

$$
f_{i,j} = \min_{k \leq j}\{f_{i-1,k}\} + w(k,j)\qquad\left(1 \leq i \leq n,1 \leq j \leq m\right)
$$

总共有 $n \times m$ 个状态，每个状态有 $O(m)$ 个决策，上述 dp 问题的时间复杂度就是 $O(n m^2)$。

> 实际上此形式也有同样的结论，可以在 $O((n+m)m)$ 复杂度解决（且 **只需要满足四边形不等式即可**），读者可以模仿 2D1D 类似的给出证明。

令 $opt(i, j)$ 为使上述表达式最小化的 $k$ 的值。如果对于所有的 $i, j$ 都有 $opt(i, j) \leq opt(i, j + 1)$，那么我们就可以用分治法来优化 dp 问题。

我们可以更有效地解出所有状态。假设我们对于固定的 $i$ 和 $j$ 计算 $opt(i, j)$。那么我们就可以确定对于任何 $j' < j$ 都有 $opt(i, j') \leq opt(i, j)$，这意味着在计算 $opt(i, j')$ 时，我们不必考虑那么多其他的点。

为了最小化运行时间，我们便需要应用分治法背后的思想。首先计算 $opt(i, \dfrac{m}{2})$ 然后计算 $opt(i, \dfrac{m}{4})$。通过递归地得到 $opt$ 的上下界，就可以达到 $O(n m \log m)$ 的时间复杂度。每一个 $opt(i, j)$ 的值只可能出现在 $\log m$ 个不同的节点中。

??? note "参考代码"
    ```cpp
    int n;
    long long C(int i, int j);
    vector<long long> dp_before(n), dp_cur(n);
    
    // compute dp_cur[l], ... dp_cur[r] (inclusive)
    void compute(int l, int r, int optl, int optr) {
      if (l > r) return;
      int mid = (l + r) >> 1;
      pair<long long, int> best = {INF, -1};
      for (int k = optl; k <= min(mid, optr); k++) {
        best = min(best, {dp_before[k] + C(k, mid), k});
      }
      dp_cur[mid] = best.first;
      int opt = best.second;
      compute(l, mid - 1, optl, opt);
      compute(mid + 1, r, opt, optr);
    }
    ```

## 1D1D 动态规划中的应用

除了经典的石子合并问题外，四边形不等式的性质在一类 1D1D 动态规划中也能得出决策单调性，从而优化状态转移的复杂度。考虑以下状态转移方程：

$$
f_{r} = \min_{l=1}^{r-1}\{f_{l}+w(l,r)\}\qquad\left(1 < r \leq n\right)
$$

**定理 2**：若函数 $w(l,r)$ 满足四边形不等式，记 $h_{l,r}=f_l+w(l,r)$ 表示从 $l$ 转移过来的状态 $r$,$k_{r}=\min\{l|f_{r}=h_{l,r}\}$ 表示最小最优决策点，则有

$$
\forall r_1 \leq r_2:k_{r_1} \leq k_{r_2}
$$

记 $l_1=k_{r_1},\ l_2=k_{r_2}$，若 $l_1>l_2$，则 $l_2<l_1<r_1\leq r_2$，根据四边形不等式有

$$
w(l_2,r_1) + w(l_1,r_2) \leq w(l_1,r_1) + w(l_2,r_2)
$$

又由于 $l_2$ 是最优决策点，因此 $h_{l_2,r_2} \leq h_{l_1,r_2}$，即

$$
f_{l_2}+w(l_2,r_2) \leq f_{l_1}+w(l_1,r_2)
$$

将以上两个不等式相加，可得

$$
f_{l_2}+w(l_2,r_1) \leq f_{l_1}+w(l_1,r_1)
$$

即 $h_{l_2,r_1} \leq h_{l_1,r_1}$，但这与 $l_1$ 是最小最优决策点矛盾，因此必有 $l_1 \leq l_2$。

但与 2D1D 动态规划中的情形不同，在这里我们根据决策单调性只能得出每次枚举 $l$ 时的下界，而无法确定其上界。因此，简单实现该状态转移方程仍然无法优化最坏时间复杂度。

先考虑一种简单的情况，转移函数的值在动态规划前就已完全确定。即如下所示状态转移方程：

$$
f_{r} = \min_{l=1}^{r-1}w(l,r) \qquad\left(1 \leq r \leq n\right)
$$

在这种情况下，我们定义过程 $\textsf{DP}(l, r, k_l, k_r)$ 表示求解 $f_{l}\sim f_{r}$ 的状态值，并且已知这些状态的最优决策点必定位于 $[k_l, k_r]$ 中，然后使用分治算法如下：

???+ note "代码实现"
    === "C++"
    
        ```cpp
        void DP(int l, int r, int k_l, int k_r) {
          int mid = (l + r) / 2, k = k_l;
          // 求状态f[mid]的最优决策点
          for (int i = k_l; i <= min(k_r, mid - 1); ++i)
            if (w(i, mid) < w(k, mid)) k = i;
          f[mid] = w(k, mid);
          // 根据决策单调性得出左右两部分的决策区间，递归处理
          if (l < mid) DP(l, mid - 1, k_l, k);
          if (r > mid) DP(mid + 1, r, k, k_r);
        }
        ```
    
    === "Python"
    
        ```python
        def DP(l, r, k_l, k_r):
            mid = int((l + r) / 2)
            k = k_l
            # 求状态f[mid]的最优决策点
            for i in range(k_l, min(k_r, mid - 1)):
                if w(i, mid) < w(k, mid):
                    k = i
            f[mid] = w(k, mid)
            # 根据决策单调性得出左右两部分的决策区间，递归处理
            if l < mid:
                DP(l, mid - 1, k_l, k)
            if r > mid:
                DP(mid + 1, r, k, k_r)
        ```

使用递归树的方法，容易分析出该分治算法的复杂度为 $O(n\log n)$，因为递归树每一层的决策区间总长度不超过 $2n$，而递归层数显然为 $O(\log n)$ 级别。

### [「POI2011」Lightning Conductor](https://loj.ac/problem/2157)

???+ note "题目大意"
    给定一个长度为 $n$（$n\leq 5\times 10^5$）的序列 $a_1, a_2, \cdots, a_n$，要求对于每一个 $1 \leq r \leq n$，找到最小的非负整数 $f_r$ 满足
    
    $$
    \forall l\in\left[1,n\right]:a_l \leq a_r + f_{r} - \sqrt{|r-l|}
    $$

显然，经过不等式变形，我们可以得到待求整数 $f_r = \max\limits_{l=1}^{n}\{a_l+\sqrt{r-l}-a_r\}$。不妨先考虑 $l < r$ 的情况（另外一种情况类似），此时我们可以得到状态转移方程：

$$
f_r = \min_{l=1}^{r-1}\{\ -a_l-\sqrt{r-l}+a_r\}
$$

根据 $-\sqrt{x}$ 的凸性，我们很容易得出（后文将详细描述）函数 $w(l, r) = -a_l - \sqrt{r-l} + a_r$ 满足四边形不等式，因此套用上述的分治算法便可在 $O(n\log n)$ 的时间内解决此题了。

现在处理一般情况，即转移函数的值是在动态规划的过程中按照一定的拓扑序逐步确定的。此时我们需要改变思维方式，由“确定一个状态的最优决策”转化为“确定一个决策是哪些状态的最优决策“。具体可见上文的「单调栈优化 DP」。

## 满足四边形不等式的函数类

为了更方便地证明一个函数满足四边形不等式，我们有以下几条性质：

**性质 1**：若函数 $w_1(l,r),w_2(l,r)$ 均满足四边形不等式（或区间包含单调性），则对于任意 $c_1,c_2\geq 0$，函数 $c_1w_1+c_2w_2$ 也满足四边形不等式（或区间包含单调性）。

**性质 2**：若存在函数 $f(x),g(x)$ 使得 $w(l,r) = f(r)-g(l)$，则函数 $w$ 满足四边形恒等式。当函数 $f,g$ 单调增加时，函数 $w$ 还满足区间包含单调性。

**性质 3**：设 $h(x)$ 是一个单调增加的凸函数，若函数 $w(l,r)$ 满足四边形不等式并且对区间包含关系具有单调性，则复合函数 $h(w(l,r))$ 也满足四边形不等式和区间包含单调性。

**性质 4**：设 $h(x)$ 是一个凸函数，若函数 $w(l,r)$ 满足四边形恒等式并且对区间包含关系具有单调性，则复合函数 $h(w(l,r))$ 也满足四边形不等式。

首先需要澄清一点，凸函数（Convex Function）的定义在国内教材中有分歧，此处的凸函数指的是（可微的）下凸函数，即一阶导数单调增加的函数。

前两条性质根据定义很容易证明，下面证明第三条性质，性质四的证明过程类似。

任取 $l \leq l' \leq r' \leq r$，根据函数 $w$ 对区间包含关系的单调性有 $w(l',r')\leq w(l,r)$ 成立。又因为 $h(x)$ 单调增加，故 $h(w(l',r')) \leq h(w(l,r))$，即复合函数 $h\circ w$ 满足区间包含单调性。

任取 $l_1\leq l_2\leq r_1\leq r_2$，根据函数 $w$ 满足四边形不等式，有

$$
w(l_1,r_1) + w(l_2,r_2) \leq w(l_1,r_2) + w(l_2,r_1)
$$

移项，并根据 $w$ 对区间包含满足单调性，可得

$$
0 \leq w(l_1,r_1) - w(l_2,r_1) \leq w(l_1,r_2) - w(l_2,r_2)
$$

记 $t = w(l_1,r_2) - w(l_2,r_2)\geq 0$，则 $w(l_1,r_1)\leq w(l_2,r_1)+t,\ w(l_1,r_2)= w(l_2,r_2)+t$，故根据函数 $h$ 的单调性可知（如果是证明性质四则第一个不等式变为等式，无需用到单调性）

$$
\begin{aligned}
    h(w(l_1,r_1)) - h(w(l_2,r_1)) & \leq  h(w(l_2,r_1)+t) - h(w(l_2,r_1)) \\
    h(w(l_1,r_2)) - h(w(l_2,r_2)) & = h(w(l_2,r_2)+t) - h(w(l_2,r_2))
\end{aligned}
$$

设 $\Delta h(x) = h(x+t)-h(x)$，则 $\Delta h'(x) = h'(x+t)-h'(x)$。由于 $h(x)$ 是一个凸函数，故导函数 $h'(x)$ 单调增加，因此函数 $\Delta h$ 也单调增加，此时有

$$
\begin{aligned}
h(w(l_1,r_1)) - h(w(l_2,r_1)) & \leq \Delta h(w(l_2,r_1)) \\
    & \leq \Delta h(w(l_2,r_2)) = h(w(l_1,r_2)) - h(w(l_2,r_2))
\end{aligned}
$$

即 $h(w(l_1,r_1)) + h(w(l_2,r_2)) \leq h(w(l_1,r_2)) + h(w(l_2,r_1))$，说明 $h\circ w$ 也满足四边形不等式。

回顾例题 1 中的 $w(l, r) = -a_l - \sqrt{r-l} + a_r$，由性质 2 可知 $-a_l+a_r$ 满足四边形不等式，而 $r-l$ 满足四边形恒等式和区间包含单调性。再根据 $-\sqrt{x}$ 的凸性以及性质 4 可知 $-\sqrt{r-l}$ 也满足四边形不等式，最终利用性质 1，即可得出 $w(l, r)$ 满足四边形不等式性质了。

### [「HNOI2008」玩具装箱 toy](https://loj.ac/problem/10188)

???+ note "题目大意"
    有 $n$ 个玩具需要装箱，要求每个箱子中的玩具编号必须是连续的。每个玩具有一个长度 $C_i$，如果一个箱子中有多个玩具，那么每两个玩具之间要加入一个单位长度的分隔物。形式化地说，如果将编号在 $[l,r]$ 间的玩具装在一个箱子里，那么这个箱子的长度为 $r-l+\sum_{k=l}^r C_k$。现在需要制定一个装箱方案，使得所有容器的长度与 $K$ 差值的平方之和最小。

设 $f_{r}$ 表示将前 $r$ 个玩具装箱的最小代价，则枚举第 $r$ 个玩具与哪些玩具放在一个箱子中，可以得到状态转移方程为

$$
f_{r} = \min_{l=1}^{r-1}\{f_{l} + \left(r-l-1-K+\sum_{k=l+1}^r C_k\right)^2\}
$$

记 $s(r) = r+\sum_{k=1}^r C_k$，则有 $w(l, r) = (s(r) - s(l) - 1 - K)^2$。显然 $s(r)$ 单调增加，因此根据性质 1 和性质 2 可知 $s(r) - s(l) - 1 - K$ 满足区间包含单调性和四边形不等式。再根据 $x^2$ 的单调性和凸性以及性质 3 可知，$w(l, r)$ 也满足四边形不等式，此时使用单调栈优化即可。

## 习题

-   [「IOI2000」邮局](https://www.luogu.com.cn/problem/P4767)
-   [Codeforces - Ciel and Gondolas](https://codeforces.com/contest/321/problem/E)(Be careful with I/O!)
-   [SPOJ - LARMY](https://www.spoj.com/problems/LARMY/)
-   [Codechef - CHEFAOR](https://www.codechef.com/problems/CHEFAOR)
-   [Hackerrank - Guardians of the Lunatics](https://www.hackerrank.com/contests/ioi-2014-practice-contest-2/challenges/guardians-lunatics-ioi14)
-   [ACM ICPC World Finals 2017 - Money](https://open.kattis.com/problems/money)

## 参考资料

-   [noiau 的 CSDN 博客](https://blog.csdn.net/noiau/article/details/72514812)
-   [Quora Answer by Michael Levin](https://www.quora.com/What-is-divide-and-conquer-optimization-in-dynamic-programming)
-   [Video Tutorial by "Sothe" the Algorithm Wolf](https://www.youtube.com/watch?v=wLXEWuDWnzI)

***

**本页面主要译自英文版博文 [Divide and Conquer DP](https://cp-algorithms.com/dynamic_programming/divide-and-conquer-dp.html)。版权协议为 CC-BY-SA 4.0。**
