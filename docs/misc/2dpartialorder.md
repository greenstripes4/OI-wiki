二维偏序是这样一类问题：已知点对的序列 $(a_1,b_1),(a_2,b_2),(a_3,b_3),\cdots$ 并在其上定义某种偏序关系 $\prec$ ，现有点 $(a_i,b_i)$ ，求满足 $(a_j,b_j)\prec(a_i,b_i)$ 的 $(a_j,b_j)$ 的数量。

什么叫偏序呢？数学中讲偏序是满足自反性、反对称性和传递性的序关系。但这听上去太抽象了，事实上，在二维的情形，我们分别对两个属性定义序关系，一定能得到一种偏序关系：

$(a_j,b_j)\prec(a_i,b_i) \;\xlongequal{def}\; a_j\lesseqgtr a_i\;\text{and}\; b_j\lesseqgtr b_i$

好像更抽象了。我们还是来举例子吧：

???+note "[Stars](http://poj.org/problem?id=2352)"
    题意是，给你一张星图，定义星星的等级是既不在它上面也不在它右边的星星的数量，求每个等级的星星的数量。这其实就是定义了这样的偏序关系： 

    $(a_j,b_j)\prec(a_i,b_i) \;\xlongequal{def}\; a_j\le a_i\;\text{and}\; b_j\le b_i$

二维偏序问题一般用树状数组解决。

这道题的数据已经以y坐标为第一关键词、x坐标为第二关键词排好序了，但一般是需要我们手动排一下序的。排序的目的是让所有可能小于当前点的点都在当前点前面被处理。然后我们把这些点的x坐标一个一个地压入树状数组。

因为已经确保了所有可能小于当前点的点都在当前点前面出现，我们只需要使用树状数组不断动态求前缀和即可。树状数组的下标需要从1开始，所以在处理的时候还要稍微注意一下。

```cpp
#include <cstdio>
#define MAXN 15010
#define MAXM 32010
#define lowbit(x) ((x) & (-x))
// 树状数组
int tree[MAXM], ans[MAXN];
void update(int i, int x)
{
    for (; i <= MAXM; i += lowbit(i))
        tree[i] += x;
}
int query(int n)
{
    int ans = 0;
    for (; n; n -= lowbit(n))
        ans += tree[n];
    return ans;
}

int main()
{
    int n, x;
    scanf("%d", &n);
    for (int i = 0; i < n; ++i)
    {
        scanf("%d%*d", &x); // 已经排好序了，y坐标可以不要了
        ans[query(x + 1)]++; // 统计
        update(x + 1, 1); // 更新，注意这两行都要+1
    }
    for (int i = 0; i < n; ++i)
    {
        printf("%d\n", ans[i]);
    }
    return 0;
}
```

实际上，这道题是最简单的二维偏序。其他种类的偏序方式往往需要转化成这种关系进行处理。一个很常见的例子是逆序对。如这道模板题：

???+note "[Ultra-QuickSort](http://poj.org/problem?id=2299)"
    存在某种特定的排序算法，通过交换两个相邻的序列元素来处理 n 个不同整数的序列，直到该序列按升序排序。例如 9 1 0 5 4 会输出为 0 1 4 5 9 ，题目中管这个方法叫“Ultra-QuickSort ”，确定 Ultra-QuickSort 需要执行多少交换操作才能对给定的输入序列进行排序。

其实逆序对本质上就是一种二维偏序：

$(j,a_j)\prec(i,a_i) \;\xlongequal{def}\; j\lt i\;\text{and}\; a_j\gt a_i$

我们可以先用上面的方法求出非严格顺序对，然后用总对数去减。但也有另一种方法，注意到数据很大，我们需要离散化。所以只要在离散化时，将数据按从大到小的顺序排序，就可以转化为之前的那种偏序关系了。部分代码如下（省略树状数组和快读）：

```cpp
const int MAXN = 500005;
int A[MAXN], C[MAXN], L[MAXN], tree[MAXN];
int main()
{
    int n;
    while (n = read())
    {
        ll sum = 0;
        for (int i = 0; i < n; ++i)
            A[i] = read();
        memset(tree, 0, sizeof(tree));
        memcpy(C, A, sizeof(int) * n);
        sort(C, C + n, greater<int>());
        int l = unique(C, C + n) - C;
        for (int i = 0; i < n; ++i)
            L[i] = lower_bound(C, C + l, A[i], greater<int>()) - C + 1;
        for (int i = 0; i < n; ++i)
        {
            update(L[i], 1);
            sum += query(L[i] - 1);
        }
        cout << sum << endl;
    }
    return 0;
}
```

再来看一个不那么明显的二维偏序题：

???+note "[Moving Points](https://codeforces.com/problemset/problem/1311/f)"
    题意是，给出直线上若干个不同的动点的初始位置和速度，求每两个点之间能达到的最短距离之和。

注意到，对于两个点 i,j ，不妨设 $x_i<x_j$ 。那么如果 $v_i\lt v_j$ ，则它们的距离会越来越大（初中追及问题），所以它们的最短距离就是此时的距离；如果 $v_i=v_j$ ，那么它们的距离会保持不变，最大距离仍然是此时的距离；否则，它们迟早会相遇，所以最短距离为0。因此这道题只需要处理 $x_i<x_j$ 且 $v_i\le v_j$ 的点对即可，是一道二维偏序题。

但是这里不是求点对的数量而是求距离之和，所以需要使用两个树状数组，一个维护目前为止所有点到原点的距离之和 L ，一个维护已经进入树状数组的点的数量 n ，则设当前点到原点的距离为 l ，可求得当前的点对的距离和为 nl-L 。部分代码：

```cpp
void update(ll x, ll d, ll tree[])
{
    for (ll i = x; i < MAXN; i += lowbit(i))
        tree[i] += d;
}
ll query(ll x, ll tree[])
{
    ll sum = 0;
    for (ll i = x; i > 0; i -= lowbit(i))
        sum += tree[i];
    return sum;
}
pair<ll, ll> P[MAXN];
ll C[MAXN], L[MAXN], len[MAXN], tot[MAXN];
int main()
{
    ll n = read(), sum = 0;
    for (int i = 0; i < n; ++i)
        P[i].first = read();
    for (int i = 0; i < n; ++i)
        P[i].second = read();
    for (int i = 0; i < n; ++i)
        C[i] = P[i].second;
    sort(P, P + n); // 排序
    sort(C, C + n);
    unique(C, C + n);
    for (int i = 0; i < n; ++i)
        L[i] = lower_bound(C, C + n, P[i].second) - C + 1; // 离散化
    for (int i = 0; i < n; ++i)
    {
        update(L[i], P[i].first, len);
        update(L[i], 1, tot);
        sum += query(L[i], tot) * P[i].first - query(L[i], len);
    }
    cout << sum << endl;
    return 0;
}
```

总而言之，最简单的二维偏序 $(a_j,b_j)\prec(a_i,b_i) \;\xlongequal{def}\; a_j\le a_i\;\text{and}\; b_j\le b_i$ 的处理方法是选 a 为第一关键词， b 为第二关键词进行排序；如果必要，将 b 离散化；然后按顺序把 b 一个一个推入树状数组，动态求前缀和。

而其他二维偏序关系，可以作不同的处理转化为最简单的二维偏序。例如：

- $(a_j,b_j)\prec(a_i,b_i) \;\xlongequal{def}\; a_j\lt a_i\;\text{and}\; ?$ ：把第一关键词的小于等于改成小于，需要在对第二关键词排序时进行逆序排序。
- $(a_j,b_j)\prec(a_i,b_i) \;\xlongequal{def}\; a_j\ge a_i\;\text{and}\; ?$ ：把第一关键词的小于等于改成大于等于，需要在对第一关键词排序时进行逆序排序。
- $(a_j,b_j)\prec(a_i,b_i) \;\xlongequal{def}\; a_j\gt a_i\;\text{and}\; ?$ ：把第一关键词的小于等于改成大于，需要在对两个关键词排序时都进行逆序排序。
- $(a_j,b_j)\prec(a_i,b_i) \;\xlongequal{def}\; ?\;\text{and}\; b_j\lt b_i$ ：把第二关键词的小于等于改成小于，查询时使用query(x-1)而不是query(x) 。
- $(a_j,b_j)\prec(a_i,b_i) \;\xlongequal{def}\; ?\;\text{and}\; b_j\ge b_i$ ：把第二关键词的小于等于改成大于等于，对第二关键词离散化时进行逆序排序。
- $(a_j,b_j)\prec(a_i,b_i) \;\xlongequal{def}\; ?\;\text{and}\; b_j\ge b_i$ ：把第二关键词的小于等于改成大于等于，对第二关键词离散化时进行逆序排序，并且查询时使用query(x-1)而不是query(x)。

前面两个例题没有对第二关键词逆序排序的原因，是它们都保证了第一关键词不重复，这样是小于还是小于等于就无所谓了，但一般的情形，还是需要自己写结构体实现的。