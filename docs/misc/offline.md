## 分治思想

https://codgician.me/zh-hans/posts/2019/07/offline-divide-and-conquer/#%E4%BB%8E%E5%BD%92%E5%B9%B6%E6%8E%92%E5%BA%8F%E8%AE%B2%E8%B5%B7

分治算法，即“分而治之”，每次将当前的问题化成两个或更多相同或相似的子问题，再对每个分出的问题进行同样的操作…… 直到子问题足够地小，让我们能够方便地直接求解。归并排序便是分治算法的经典应用之一。

不过，在过去我们遇到的题目中，可能分治思想更多应用在动态规划这一类的题目中。我们来考虑，在允许离线算法的数据结构问题中，是否可以尝试对操作和询问进行分治？

数据结构问题往往要求我们维护一系列数据，并对一系列操作依次做出响应。其中，操作往往可以分为以下两种：

1. 查询：统计数据信息；
2. 修改：更新数据状态。

有了这样的定义后，我们便可以把常见的数据结构问题分为如下两类：

1. 静态问题：只有查询，或者所有查询都在所有修改之后进行。
2. 动态问题：所有不是静态问题的数据结构问题。

同时，我们也可以把我们用于数据结构的算法分为两类：

1. 在线算法：对于每一个查询及时进行响应的算法；
2. 离线算法：预先知道所有操作，集中进行处理后再批量回答查询。

比如，常见的线段树就是一个在线算法，因为它能够对每一个询问做出及时的响应；而莫队则是一个离线算法，因为我们必须预先知道完整的操作序列才能够计算答案。

顾名思义，本文所要讲的算法都是离线算法。离线所能带来的好处就是通过增加 O(logn) 的复杂度达到原问题进行简化的目的，或许是把动态问题转换成更加容易处理的静态问题，亦或是把带有插入和删除的操作序列转换成只包含插入操作…… 非常的优雅与简洁，这也是我非常喜欢这一思想的主要原因。

## 基于时间的离线分治

### CDQ 分治

**从归并排序讲起**

mergesort(l,r)：对序列下表区间 \[l,r\] 内元素按值进行排序

- mid = (l+r)/2
- mergesort(l,mid)；
- mergesort(mid+1,r)；
- 双指针合并两个左右有序区间：merge(l,mid+1,r+1)

不难发现，归并排序巧妙地避开了对区间进行排序本身，把问题完全转变为了两个有序区间的合并问题。只要能解决后者，那么前者就解决了。这能不能给我们带来一些启发呢？

**偏序问题**

对于两个 k 元组 $t_1: \langle x_1, x_2, \dots x_k \rangle, \ t_2: \langle y_1, y_2, \dots y_k \rangle $，如果 $\forall i \in [1, k]$ ，满足 $x_i < y_i$ ，那么我们称 $\langle t_1, t_2 \rangle$ 构成一组 k 维偏序（有的地方可能会把 < 定义成 $\le$ ）。

我们来考虑这样一个问题：

???+note "三维偏序"
    给定 n 个三元组 $p_i = \langle a_i, b_i, c_i \rangle$ ，对于每一个元素 $p_i$ ，求满足： $a_i < a_j, b_i < b_j, c_i < c_j$ 的 $\langle i, j \rangle$ 对数。为了方便，我们先考虑不存在 i, j 使得 $a_i = a_j$ 或 $b_i = b_j$ 或 $c_i = c_j$ 的情况。

    数据范围：$n \le 10^5, 1 \le a_i, b_i, c_i \le 10^9$ 

首先，如果是二维偏序问题的话（即假设每个元素只有 a,b 两维），我们可以很容易想到一个解法：首先将所有元素按照 a 维从小到大排序，然后将 b 维离散化，按值扔进一个树状数组双指针统计一下就好了（即从小到大遍历 a，然后在树状数组中 b 的位置 +1，每次都统计一下当前树状数组中小于当前 a 的前缀和即可）。理论上三维偏序我们可以用一个二维树状数组解决，然而空间开不下，这可怎么办呢……

我们还是先将所有元素按照 a 维从小到大进行排序，接下来考虑这样一个算法：

$solve(l, r)：\forall k \in [l, r]$，计算第 \[l, k - 1\] 项元素对第 k 项元素的贡献（与此同时对所有三元组按照 b 维排序）：

- $mid = \frac{1}{2}(l + r)$
- solve(l, mid)
- solve(mid + 1, r)

计算 \[l, mid\] 中元素对 \[mid + 1, r\] 中每个元素造成的贡献，同时双指针合并两个左右有序区间：merge(l, mid + 1, r + 1)。

接下来我们简要谈谈为什么这一算法是正确的。对于第 kk 项元素而言（作为偏序对中较大的元素）：

- 若 $k \le mid$ ，则 solve(l, mid) 已经计算它所能与第 \[l, k - 1\] 项元素构成的偏序对数量；
- 若 k > mid，则 solve(mid + 1, r) 已经计算了其所能与第 \[mid + 1, k - 1\] 项元素构成的偏序对数量；最后一步则计算了其可与第 \[l, mid\] 项元素构成的偏序对数量。两者相加后即为其与第 \[l, k - 1\] 项元素可构成的偏序对数量。

那么接下来我们需要仔细探讨一下上述算法中第 44 步应当如何进行。不难发现在这一步时有两个重要特性：

- 左区间中所有元素 a 维一定小于右区间中所有元素 a 维；
- 左右区间都已经按 b 维排好序了。

显然，这意味着我们不用去管 a 维了，因为只要两个元素一个来自于左区间而另一个来自右区间，那么它们在 a 维上肯定是满足偏序对要求的。那么…… b 维已经排好序了，c 维…… 等等，这不就是个二维偏序问题吗？所以我们只需要采用之前对二维偏序的做法对其进行处理就完事了~ 不难发现，采用了分治思想后，我们成功地将原问题的维度降低了一维。

接下来我们简单分析一下复杂度：记当前分治区间长度为 n，且第 4 步处理所需复杂度为 $\mathcal{O}(f(n))$（比如在上例中就是树状数组的复杂度即 $\mathcal{O}(\log{n})$ ，则有：

$T(n) = 2T(\frac{n}{2}) + \mathcal{O}(f(n))$

借助主定理解一下（或者画递归树看一看）不难得到：

$T(n) \le \mathcal{O}(f(n) \log{n})$

通过复杂度计算我们也得到一条重要注意事项：我们必须保证在执行第 4 步的时候复杂度与当前分治区间长度相关而不可以与整个区间长度相关，否则复杂度就不是上面这个样子了。在之前提到的具体问题中，就是清空树状数组的时候不能直接 memset，而必须逐一删除被修改了的部分，这样才能保证复杂度正确。

下面简要上一段代码以供参考：

```cpp
void divideConquer(int headPt, int tailPt) {
    if (headPt == tailPt)
        return;
    int midPt = (headPt + tailPt) >> 1;
    divideConquer(headPt, midPt); divideConquer(midPt + 1, tailPt);
    int j = headPt; // j: 左区间中指针；i: 右区间中指针
    for (int i = midPt + 1; i <= tailPt; i++) {
        for (; j <= midPt && arr[j].b < arr[i].b; j++) 
            add(arr[j].c, 1);
        // 统计总三维偏序对数量（树状数组中 c 维小于当前的数量）
        ans += prefixSum(arr[i].c - 1);
    }
    // 撤销对树状数组的更改
    for (int i = headPt; i < j; i++)
        add(arr[i].c, -1);
    // 按 b 维合并左右有序区间（cmpSnd 为按照 b 维从小到大排序的比较函数）
    inplace_merge(arr + headPt, arr + midPt + 1, arr + tailPt + 1, cmpSnd);
}
```

接下来，我们再来仔细考虑一下之前为了方便引入的限制。如果不满足不同元素间同一维上的值互不相同会引入什么问题？

问题依然是出在第 4 步。在这一步时，我们只能保证左区间 a 维小于等于右区间，而不能保证严格小于。换言之，在对 b 维进行分治时，可能出现对于右区间某元素，左区间中存在某个元素使得两者 a 维相等。读者可以先自行思考一下如何解决这一问题~

这里提供一种解决方法以供参考：先将 a 维离散化，在双指针处理时开一个数组来记录某个 a 维值对应的左区间中的元素的个数，在统计时减掉 a 维相等的情况。

最后，大家可以去尝试一下一道模板题（注意这道题与前面举的例子不完全一样），顺便附上代码以供参考。

???+note "[陌上花开](https://www.luogu.com.cn/problem/P3810)"
    这是一道模板题，可以使用 bitset，CDQ 分治，KD-Tree 等方式解决。

    有 n 个元素，第 i 个元素有 $a_i,b_i,c_i$ 三个属性，设 f(i) 表示满足 $a_j \leq a_i$ 且 $b_j \leq b_i$ 且 $c_j \leq c_i$ 且 $j \ne i$ 的 j 的数量。

    对于 $d \in [0, n)$ ，求 f(i)=d 的数量。

    输入格式: 第一行两个整数 n, k，表示元素数量和最大属性值。

    接下来 n 行，每行三个整数 $a_i ,b_i,c_i$ ，分别表示三个属性值。

    输出格式: n 行，第 d+1 行表示 f(i)=d 的 i 的数量。

???+note "参考代码"

    ```cpp
    // luogu-judger-enable-o2
    #include <bits/stdc++.h>
    using namespace std;

    #define SIZE 100010
    #define VAL_SIZE 200020

    typedef struct _Node {
        int a, b, c;
        int num, ans;

        bool operator == (const struct _Node & snd) const {
            return a == snd.a && b == snd.b && c == snd.c;
        }
    } Node;

    Node arr[SIZE];
    int ans[SIZE], bit[VAL_SIZE], siz;

    bool cmpSnd(const Node & fst, const Node & snd) {
        if (fst.b != snd.b)
            return fst.b < snd.b;
        return fst.c < snd.c;
    }

    bool cmpFst(const Node & fst, const Node & snd) {
        if (fst.a != snd.a)
            return fst.a < snd.a;
        return cmpSnd(fst, snd);
    }

    int lowbit(int n) {
        return n & -n;
    }

    void add(int pos, int val) {
        for (int i = pos; i <= siz; i += lowbit(i))
            bit[i] += val;
    }

    int prefixSum(int pos) {
        int ret = 0;
        for (int i = pos; i > 0; i -= lowbit(i))
            ret += bit[i];
        return ret;
    }

    void divideConquer(int headPt, int tailPt) {
        if (headPt == tailPt)
            return;
        int midPt = (headPt + tailPt) >> 1;
        divideConquer(headPt, midPt);
        divideConquer(midPt + 1, tailPt);

        int j = headPt;
        for (int i = midPt + 1; i <= tailPt; i++) {
            for (; j <= midPt && arr[j].b <= arr[i].b; j++)
                add(arr[j].c, arr[j].num);
            arr[i].ans += prefixSum(arr[i].c);
        }

        for (int i = headPt; i < j; i++)
            add(arr[i].c, -arr[i].num);

        inplace_merge(arr + headPt, arr + midPt + 1, arr + tailPt + 1, cmpSnd);
    }

    int main() {
        ios::sync_with_stdio(false);
        cin.tie(0); cout.tie(0);
        memset(ans, 0, sizeof(ans));
        memset(bit, 0, sizeof(bit));

        int num;
        cin >> num >> siz;
        for (int i = 0; i < num; i++) {
            cin >> arr[i].a >> arr[i].b >> arr[i].c;
            arr[i].num = 1; arr[i].ans = 0;
        }

        sort(arr + 0, arr + num, cmpFst);

        int ovrPt = 0, cntPt = 1;
        while (cntPt < num) {
            if (arr[ovrPt] == arr[cntPt])
                arr[ovrPt].num += arr[cntPt].num;
            else
                arr[++ovrPt] = arr[cntPt];
            cntPt++;
        }
        ovrPt++;

        divideConquer(0, ovrPt - 1);
        for (int i = 0; i < ovrPt; i++) {
            arr[i].ans += arr[i].num - 1;
            ans[arr[i].ans] += arr[i].num;
        }
        for (int i = 0; i < num; i++)
            cout << ans[i] << '\n';
        return 0;
    }
    ```
**四维偏序**

既然三维偏序可以借助分治降成二维偏序，那么四维偏序何尝不可以借助分治降为三维偏序，然后再借助一层分治降成二维偏序呢？（不过 $\mathcal{O}(n\log^3{n})$ ) 这个复杂度也是挺尴尬的……

如果您已经跃跃欲试，不妨去 [COGS 2479:【HZOI 2016】偏序](https://www.cnblogs.com/candy99/p/6442434.html) 当场表演一波

不过当我们在对 c 维进行分治（即三维偏序降为二维偏序）时只能保证任意左区间元素 $p_i$ 和任意右区间元素 $p_j$ 之间满足 $b_i \le b_j$ ，而并不能保证 $a_i \le a_j$ 。因此：

- 对 b 维进行分治时，合并左右区间前应标记此时元素属于哪个区间，且合并后应当备份这一区间；
- 对 c 维进行分治时计算两层分治都在左区间的元素对两层分治都在右区间的元素的贡献，完成后应将区间从备份恢复以继续 b 维上的分治。

具体的实现可以参考一下[我的代码](https://github.com/codgician/Competitive-Programming/blob/master/COGS/2479/divide_and_conquer_2d.cpp)。

**十维偏序？**
在 10^5^ 的这个数据范围下，$\mathcal{O}(n^2)$ ) 不香吗

### 动态数据结构问题

接下来我们讨论这个思想怎么跟动态数据结构问题扯上联系。

我们来考虑一下动态数据结构问题中回答查询的本质：即计算初始数据和本次查询之前，所有修改对该查询造成的影响。那么我们能不能借助分治思想使得修改和查询不要混在一起呢？

$solve(l, r) ：\forall k \in [l, r]$ ，若第 k 项操作是查询，则计算 \[l, k - 1\] 中修改对其造成的影响：

- $mid = \frac{1}{2}(l + r)$
- solve(l, mid)
- solve(mid + 1, r)
- 计算第 \[l, mid\] 项操作中所有修改对 \[mid + 1, r\] 中所有查询造成的影响。

这样一来，我们便将 “计算操作集区间 \[l, r\] 内所有查询” 这一问题转换为了 “计算左区间中所有修改对右区间中所有查询造成影响”。换言之，以 $\mathcal{O}(\log{n})$ 的代价，我们成功地将一个动态问题转换成了静态问题。

那么接下来我们就来看几个例子吧。

???+note "动态二维数点"

    给定二维平面上 n 个点且有 q 次操作。需要支持两种操作：

    - 添加一个新坐标点；
    - 删除一个已有的坐标点；

    每次询问一个矩形区域 $(x_L, y_L), \ (x_R, y_R)$ 内点的数量（左下和右上顶点坐标）。

    数据范围：$n \le 10^5, \ q \le 10^5$ 。

为了方便说明，我们先引入一些标记：

- 记 a(x,y) 代表 (x,y) 位置上点的个数；
- 记 $Q(x_L, y_L, x_R, y_R)$ 代表矩形区域 $(x_L, y_L), \ (x_R, y_R)$ 内点的数量；
- 记 F(x,y) 代表矩形区域 $(1, 1), (x, y)$ 内点的数量（即二维前缀和）。

那么显然，我们可以得到：

$Q(x_L, y_L, x_R, y_R) = \sum\limits_{i = x_L}^{x_R}\sum\limits_{j = y_L}^{y_R} a(i, j)$

借助一点点容斥原理，我们也可以推出：

$\begin{aligned} Q(x_L, y_L, x_R, y_R) = & F(x_R, y_R) + F(x_L − 1, y_L − 1) \\ & − F(x_L − 1, y_R) − F(x_R, y_L − 1) \end{aligned}$

那么现在问题就转换为求解 F(x, y)F(x,y) 了。为什么要做这一转换？因为通过之前的了解我们也发现分治思想能够很好解决两者之间的大小关系，而很难解决三者之间的大小关系。转换为前缀和后我们处理的便是两者间的大小关系了。

另外既然要支持修改，那么意味着操作之间的顺序不能变。这等价于引入了时间维度 t，越早出现的操作 t 越小。这样问题便被转换为，若 $P_i: \langle t_i, x_i, y_i \rangle$ 是查询，则需计算所有满足以下条件修改的影响之和：

- $t_j < t_i$
​- $x_j \le x_i, \ y_j \le y_i$
 
Umm… 这不就成一个三维偏序问题了吗？接下来我们会介绍这一模型的一种经典应用。

**动态区间不同值**

    ???+note "[数颜色](https://www.luogu.com.cn/problemnew/show/P1903)"
    给定一个长度为 n 的序列，共 q 次操作。需要支持两种操作：

    - 把位置 i 上的数修改为 v（从 1 开始编号）；
    - 查询区间 \[L, R\] 内不同的数有多少种。

    数据范围：$n \le 10^5, \ q \le 10^5$ 。

下面给出一点提示：

- 记 pre(i) 代表在位置 i 左边与它最近的数值相同的数的位置；
- 对于每次查询只计算满足 $pre(𝑖) < L$ 的位置个数；
- $\langle l, 0 \rangle \le \langle i, pre(i) \rangle \le \langle r, l - 1 \rangle$ 。

至此我们成功得到了一个二维数点问题。

不过最后还有一点问题，每次修改会影响 pre 值。不过我们发现每次修改至多影响 3 个位置的 pre 值（假设修改位置 i，那么 pre(i)、原先值的后继 和 新值前驱的后继会改变），那么按值开 n 个 std::set 维护即可。

最后附上代码以供参考。

???+note "参考代码"

    ```cpp
    #include <bits/stdc++.h>
    using namespace std;

    #define SIZE 100010
    #define VAL_SIZE 1000010

    typedef struct _Query {
        int type;   // 0: Query+, 1: Query-, 2: Modify
        int x, y, val;

        bool operator < (const _Query & snd) const {
            if (x != snd.x)
                return x < snd.x;
            if (y != snd.y)
                return y < snd.y;
            return type > snd.type;
        }
    } Query;

    Query qArr[SIZE << 3]; 

    set<int> st[VAL_SIZE];

    int arr[SIZE], pre[SIZE], last[VAL_SIZE], bit[SIZE], ans[SIZE];

    int lowbit(int n) { 
        return n & -n;
    }

    void add(int pos, int val) {
        pos += 3;
        for (int i = pos; i < SIZE; i += lowbit(i))
            bit[i] += val;
    }

    int prefixSum(int pos) {
        int ret = 0; pos += 3;
        for (int i = pos; i > 0; i -= lowbit(i))
            ret += bit[i];
        return ret;
    }

    void divideConquer(int headPt, int tailPt) {
        if (headPt == tailPt)
            return;
        int midPt = (headPt + tailPt) >> 1;
        divideConquer(headPt, midPt);
        divideConquer(midPt + 1, tailPt);

        int j = headPt;
        for (int i = midPt + 1; i <= tailPt; i++) {
            if (qArr[i].type == 2)
                continue;
            for (; j <= midPt && qArr[j].x <= qArr[i].x; j++)
                if (qArr[j].type == 2)
                    add(qArr[j].y, qArr[j].val);
            ans[qArr[i].val] += (1 - (qArr[i].type << 1)) * prefixSum(qArr[i].y);
        }

        for (int i = headPt; i < j; i++)
            if (qArr[i].type == 2)
                add(qArr[i].y, -qArr[i].val);

        inplace_merge(qArr + headPt, qArr + midPt + 1, qArr + tailPt + 1);
    }

    int main() {
        ios::sync_with_stdio(false);
        cin.tie(0); cout.tie(0);
        memset(last, -1, sizeof(last));
        memset(bit, 0, sizeof(bit));
        int len, qNum, ansPt = 0, qPt = 0;
        cin >> len >> qNum;
        for (int i = 0; i < len; i++) {
            cin >> arr[i];
            pre[i] = last[arr[i]];
            qArr[qPt++] = {2, i, pre[i], 1};
            last[arr[i]] = i;
            st[arr[i]].insert(i);
        }

        for (int i = 0; i < qNum; i++) {
            char op; int x, y;
            cin >> op >> x >> y;
            x--;
            if (op == 'R') {  // Modify
                // Erase original
                qArr[qPt++] = {2, x, pre[x], -1};

                auto it = st[arr[x]].upper_bound(x);
                if (it != st[arr[x]].end()) {
                    qArr[qPt++] = {2, *it, pre[*it], -1};
                    pre[*it] = pre[x];
                    qArr[qPt++] = {2, *it, pre[*it], 1};
                }

                st[arr[x]].erase(x);

                // Insert new
                it = st[y].upper_bound(x);
                if (it != st[y].end()) {
                    pre[x] = pre[*it];
                    qArr[qPt++] = {2, x, pre[x], 1};
                    qArr[qPt++] = {2, *it, pre[*it], -1};
                    pre[*it] = x;
                    qArr[qPt++] = {2, *it, pre[*it], 1};
                } else {
                    if (st[y].empty()) {
                        pre[x] = -1;
                        qArr[qPt++] = {2, x, pre[x], 1};
                    } else {
                        it = prev(it);
                        pre[x] = *it;
                        qArr[qPt++] = {2, x, pre[x], 1}; 
                    }
                }

                st[y].insert(x);
                arr[x] = y;

            } else {    // Query
                y--;
                qArr[qPt++] = {0, y, x - 1, ansPt};
                qArr[qPt++] = {1, x - 1, x - 1, ansPt};
                ansPt++;
            }
        }

        divideConquer(0, qPt - 1);

        for (int i = 0; i < ansPt; i++)
            cout << ans[i] << '\n';
        return 0;
    }
    ```

**动态曼哈顿最近点对**

???+note "[天使玩偶](https://www.luogu.com.cn/problem/P4169)"
    二维平面上，有 q 种操作。需要支持三种操作：

    - 添加一个新坐标点 (x, y)
    -删除一个已有的坐标点 (x, y)
    - 查询距离 $(x_A, y_A)$ 曼哈顿距离最接近的点。

    数据范围：均为 $10^5$ 数量级。

A,B 两点之间的曼哈顿距离：

$dist(A, B) = |x_A - x_B| + |y_A - y_B|$

首先看着这个绝对值符号就很不爽，我们需要考虑将绝对值符号去除掉。我们不妨对所有情况进行讨论试一试：

$\begin{cases} (x_A + y_A) - (x_B + y_B) & x_A \ge x_B, \ y_A \ge y_B \\ (x_A - y_A) - (x_B - y_B) & x_A \ge x_B, \ y_A < y_B \\ (-x_A + y_A) - (-x_B + y_B) & x_A < x_B, \ y_A \ge y_B \\ (-x_A - y_A) - (-x_B - y_B) & x_A < x_B, \ y_A < y_B \end{cases}$

不难发现对于每一次询问 $(x_A, y_A)$ 而言，上面四个式子前半部分都是定值。既然要最小化结果，那么我们只要让后半部分最小就好了。换言之，我们只需要对这四种情况都分别跑一次 CDQ 分治并且取最小答案就好了。按照 x 坐标排序，按照 y 坐标将 x+y 插入树状数组，并求出每个询问点左下角最大的 x+y 的值

不过，在实现的时候还有一个小技巧。我们还可以对上面的四种情况进行进一步整理：

$\begin{cases} (x_A + y_A) - (x_B + y_B) & x_A \ge x_B, \ y_A \ge y_B \\ (x_A + (\infty - y_A)) - (x_B + (\infty - y_B)) & x_A \ge x_B, \ (\infty - y_A) > (\infty - y_B) \\ ((\infty - x_A) + y_A) - ((\infty - x_B) + y_B) & (\infty - x_A) > (t - x_B), \ y_A \ge y_B \\ ((\infty - x_A) + (\infty - y_A)) - ((\infty - x_B) + (\infty - y_B)) & (\infty - x_A) > (\infty - x_B), \ (\infty - y_A) > (\infty - y_B) \end{cases}$

需要注意的是，由于本题是取最大值，因此把上面式子中的 >> 换成 \ge≥ 是不影响答案的；而如果是要计数的话则需要留意这一细节。

不难发现，我们成功地把四个式子都转换成了如下的形式：

$x_A' + y_A' - (x_B' + y_B') \quad x_A' \ge x_B', \ y_A' \ge y_B'$

这样一来，我们只需要考虑这样一种情况就好了，我们只需要改变一下 $x_A', y_A', x_B', y_B'$ 的值跑四遍一种情况即可，这样极大方便了代码编写。

也就是说，现在问题转化成了需要支持下面两种操作：

- 插入新点 (x, y)(x,y)；
- 给定点 $(x_A, y_A)$，询问满足以下条件的 $(x_B, y_B)$：
    - $x_A \ge x_B, \ y_A \ge y_B$
    - $x_B + y_B$ 取值最大。

可以看出，这依然是个三维偏序问题的变种。至于维护最大值，我们其实需要维护的只是前缀最大值，魔改一下树状数组就好了。

???+note "参考代码"

    ```cpp
    #include "bits/stdc++.h"
    #define hhh printf("hhh\n")
    #define see(x) (cerr<<(#x)<<'='<<(x)<<endl)
    using namespace std;
    typedef long long ll;
    typedef pair<int,int> pr;
    inline int read() {int x=0;char c=getchar();while(c<'0'||c>'9')c=getchar();while(c>='0'&&c<='9')x=x*10+c-'0',c=getchar();return x;}

    const int maxn = 6e5+10;
    const int mod = 2e9+7;
    const double eps = 1e-9;
    const int maxx = 2e6+10;

    struct P{
        int f, x, y, id;
        P() {}
        P(int f, int x, int y, int id): f(f), x(x), y(y), id(id) {}
        bool operator < (const P &rhs) const {
            return x<rhs.x;
        }
    }a[maxn], a1[maxn];

    int n, m, mx, my, mm, nx;
    int b[maxx], ans[maxn];

    inline void max(int &x, int y) { if(x<y) x=y; }
    inline void min(int &x, int y) { if(x>y) x=y; }
    inline void clear(int x) { while(x<=mm) if(b[x]) b[x]=0, x+=x&-x; else return; }
    inline void update(int x, int d) { while(x<=mm) max(b[x],d), x+=x&-x; }
    inline int query(int x) { int ans=0; while(x) max(ans,b[x]), x-=x&-x; return ans;}

    void pre() {
        int mx=0, my=0; nx=0;
        for(int i=1; i<=n; ++i)
            if(a[i].f) max(mx,a[i].x), max(my,a[i].y);
        for(int i=1; i<=n; ++i)
            if(a[i].x<=mx&&a[i].y<=my) a[++nx]=a[i];
    }

    void solve(int l, int r) {
        if(l==r) return;
        int m=(l+r)/2;
        solve(l,m), solve(m+1,r);
        sort(a+l,a+m+1), sort(a+m+1,a+r+1);
        int j=l;
        for(int i=m+1; i<=r; ++i) {
            if(!a[i].f) continue;
            while(j<=m&&a[j].x<=a[i].x) {
                if(!a[j].f) update(a[j].y,a[j].x+a[j].y);
                ++j;
            }
            int tmp=query(a[i].y);
            if(tmp) min(ans[a[i].id],a[i].x+a[i].y-tmp);
        }
        while(--j>=l) if(!a[j].f) clear(a[j].y);
    }

    int main() {
        //ios::sync_with_stdio(false); cin.tie(0);
        n=read(), m=read();
        for(int i=1; i<=n; ++i) {
            int x=read()+1, y=read()+1;
            max(mx,x), max(my,y);
            a1[i]=P(0,x,y,i);
        }
        while(m--) {
            int f=read()-1, x=read()+1, y=read()+1;
            max(mx,x), max(my,y);
            ++n, a1[n]=P(f,x,y,n);
        }
        ++mx, ++my, mm=mx+my;

        for(int i=1; i<=n; ++i) a[i]=a1[i], ans[i]=1e9;
        pre(); solve(1,nx);
        for(int i=1; i<=n; ++i) a[i]=a1[i], a[i].x=mx-a[i].x;
        pre(); solve(1,nx);
        for(int i=1; i<=n; ++i) a[i]=a1[i], a[i].y=my-a[i].y;
        pre(); solve(1,nx);
        for(int i=1; i<=n; ++i) a[i]=a1[i], a[i].x=mx-a[i].x, a[i].y=my-a[i].y;
        pre(); solve(1,nx);

        for(int i=1; i<=n; ++i) if(a1[i].f) printf("%d\n", ans[i]);
    }
    ```

### 优化动态规划

**三维 LIS**

???+note "[Pinball Game 3D](http://acm.hdu.edu.cn/showproblem.php?pid=4742)"
    给定长为 n 的三元组序列 $p_i = \langle a_i, b_i, c_i \rangle$，求其最长的上升子序列。

    数据范围：$n \le 10^5, a_i, b_i, c_i \le 2^{30}$

借助分治对 dp(i) 进行更新就行了~

首先，LIS 的 DP 方程式很好写出：

我们记 dp(i) 代表前 i 个元素所能构成的 LIS 长度，那么对于能够使得 $\langle i, j \rangle$ 构成偏序对的 j，我们可以用如下转移方程来更新答案：

$dp(i) = \max\limits_{j < i}\{dp(j)\} + 1$

但是在 DP 时我们需要注意顺序问题，比如采用上面方程的话我们需要从左往右 DP，换言之在更新 dp(i) 时，$\forall j < i$ ，其 dp(j) 应当都已被计算完成。所以在进行分治的时候我们要 “中序” 进行，即先左区间递归，然后计算左右之间贡献，最后再右区间递归。由于双指针合并要求左右区间有序，因此左区间递归后还需对右区间进行排序，同时递归右区间前还需还原右区间的顺序。

???+note "参考代码“

    ```cpp
    #include <bits/stdc++.h>
    using namespace std;

    #define SIZE 100010

    const int mod = 1 << 30;

    typedef struct _Node {
        int x, y, z;
    } Node;

    Node arr[SIZE], tmp[SIZE];

    pair<int, int> dp[SIZE], bit[SIZE];
    int dsc[SIZE], len;

    void upd(pair<int, int> & ret, const pair<int, int> & val) {
        if (ret.first < val.first)
            ret = val;
        else if (ret.first == val.first)
            ret.second += val.second, ret.second -= (ret.second >= mod ? mod : 0);
    }

    int lowbit(int n) {
        return n & -n;
    }

    void addMax(int pos, const pair<int, int> & val) {
        pos++;
        for (int i = pos; i <= len; i += lowbit(i))
            upd(bit[i], val);
    }

    void clear(int pos) {
        pos++;
        for (int i = pos; i <= len; i += lowbit(i))
            bit[i] = make_pair(0, 0);
    }

    pair<int, int> prefixMax(int pos) {
        pair<int, int> ret = make_pair(0, 0); pos++;
        for (int i = pos; i > 0; i -= lowbit(i))
            upd(ret, bit[i]);
        return ret;
    }

    bool cmpSnd(const Node & fst, const Node & snd) {
        if (fst.y != snd.y)
            return fst.y < snd.y;
        return fst.z < snd.z;
    }

    bool cmpFst(const Node & fst, const Node & snd) {
        if (fst.x != snd.x)
            return fst.x < snd.x;
        return cmpSnd(fst, snd);
    }

    void divideConquer(int headPt, int tailPt) {
        if (headPt == tailPt)
            return;

        int midPt = (headPt + tailPt) >> 1;
        divideConquer(headPt, midPt);
        
        copy(arr + midPt + 1, arr + tailPt + 1, tmp + midPt + 1);
        sort(arr + midPt + 1, arr + tailPt + 1, cmpSnd);

        int j = headPt;
        for (int i = midPt + 1; i <= tailPt; i++) {
            for (; j <= midPt && arr[j].y <= arr[i].y; j++)
                addMax(arr[j].z, dp[arr[j].x]);
            
            pair<int, int> ret = prefixMax(arr[i].z); ret.first++;
            upd(dp[arr[i].x], ret);
        }

        for (int i = headPt; i <= j; i++)
            clear(arr[i].z);

        copy(tmp + midPt + 1, tmp + tailPt + 1, arr + midPt + 1);
        divideConquer(midPt + 1, tailPt);
        inplace_merge(arr + headPt, arr + midPt + 1, arr + tailPt + 1, cmpSnd);
    }

    int main() {
        ios::sync_with_stdio(false);
        cin.tie(0); cout.tie(0);

        int caseNum; cin >> caseNum;
        while (caseNum--) {
            cin >> len;
            fill(bit + 0, bit + len, make_pair(0, 0));
            for (int i = 0; i < len; i++) {
                cin >> arr[i].x >> arr[i].y >> arr[i].z;
                dsc[i] = arr[i].z;
            }

            sort(arr + 0, arr + len, cmpFst);
            sort(dsc + 0, dsc + len);
            int dscLen = unique(dsc + 0, dsc + len) - dsc;

            for (int i = 0; i < len; i++) {
                arr[i].z = lower_bound(dsc + 0, dsc + dscLen, arr[i].z) - dsc;
                arr[i].x = i; dp[i] = make_pair(1, 1);
            }

            divideConquer(0, len - 1);

            pair<int, int> ans = make_pair(0, 0);
            for (int i = 0; i < len; i++)
                upd(ans, dp[i]);
            cout << ans.first << " " << ans.second << endl; 
        }

        return 0;
    }
    ```

### 小结

通过上面的例题，我们大致领略了 CDQ 分治这一思想的几个神奇作用，即 以 $\mathcal{O}(\log{n})$ 复杂度的代价：

- 将区间内的问题转化为两个有序区间之间的问题；
- 将高维的问题维度降低一维；
- 将动态数据结构问题转化成静态数据结构问题。

## 线段树分治

前面也提到了动态数据结构问题带有修改操作，而修改操作也可以被分为插入和删除两种。如果插入和删除都比较好实现的话，那么运用上面的 CDQ 分治就可以容易求解问题了。但是再有的题目里面，插入很好实现，但是删除却难以实现。比如并查集维护图的连通性时，添加边很容易，但是如果要删除边就会很复杂（因为并查集里面只维护了连通性而没有维护原图本身）。在这种时候，我们便可以考虑用线段树分治以 $\mathcal{O}(\log{n})$ 的代价把带有插入和删除的问题转化为只带插入的问题。

为了便于理解，我们直接上一道经典例题。

**动态二分图判定**

???+note "[二分图](https://darkbzoj.tk/problem/4025)"
    给定一张 n 个点的图，在 T 时间内一些边会在 $s_i$ 时刻出现，并在 $t_i$ 时刻消失。询问每一个时刻该图是否是二分图。

    数据范围：均在 $10^5$ 级别。

首先，判断是否构成二分图的本质即判断是否存在奇环。一张无向图不存在奇环是该图为二分图的充要条件。那么如何维护奇环信息？我们可以用带权的并查集解决，即在并查集中记录一个深度，如果在连边时两点深度加起来在加 1 是奇数的话，就说明存在奇环了（大家可以想想为什么）。

现在问题来了，并查集中加边容易删边难，这可怎么办 QAQ。

首先，对于元素又有插入又有删除这一特性，说明每一个元素都有一个存活的时间区间（在这道题里面是直接把存活区间给出了）。对于更一般的情况，我们可以对于每一个元素预处理出时间区间，然后对时间区间进行分治。

假设一共有 q 次询问（即 q 个事件），考虑对完整时间区间 \[1, q\] 建一棵线段树，那么对于时间点的询问即询问这棵线段树的叶子节点上的状态；而对于边的修改则是对线段树上对应区间上的修改。这样问题就转换为对于一个时间区间加边和查询叶子节点，也就不存在删边这种操作了。而整个分治过程本质上就是对这一棵线段树做先序遍历。

solve(l,r,Q)：对于时刻 \[l, r\]，操作集为 Q，满足 $\forall q \in Q$ ，其存活时间区间与当前分治的时间区间交集非空；而这个函数的作用为对于 \[l, r\] 内每个时刻，判断此时图是否是二分图。操作集 Q 中元素 q 带有 4 个属性：$\langle u, v, l, r \rangle$ 。其中，u,v 代表边的两个端点，而 l,r 代表出现和消失时刻。我们可以考虑如下算法：

- $mid = \frac{1}{2} (l + r)$，并新建两个空操作集 $Q_L, Q_R$
- 遍历 $q \in Q$
    - 若 $q.l = l \land q.r = r$（说明这一操作对其子树中所有叶子节点都有影响，那么直接加边）：
        - 并查集中合并 q.u 和 q.v，并判断是否出现奇环；
        - 存在奇环？说明时间区间 \[l, r]\ 都凉了，均标记答案为否！撤销对并查集的操作并回溯（所以我们需要可撤销并查集，其实也很简单，就搞一个栈记录一下操作之前值的情况然后撤销的时候弹栈就好了）；
    - 若 $q.l \le mid$ ：令 $q.r = \min⁡(q.r, mid)$，并把 q 放入 $Q_L$
    - 若 $q.r > mid$ ：$q.l = \max⁡(q.l, mid + 1)$ ，并把 q 放入 $Q_R$
- 若 l = r，标记答案为是，撤销对并查集的操作并回溯；
- $solve(l, mid, Q_L)$
- $solve(mid + 1, r, Q_R)$
- 撤销对并查集的操作并回溯。

这样一来，我们就成功避免了删除操作，从而更容易地得以解决问题。

最后附上[我的代码](https://github.com/codgician/Competitive-Programming/blob/master/BZOJ/4025/divide_and_conquer_disjoint_set_heuristic.cpp) 以供参考。我的写法实际上是没有建出线段树的，当然也有另外建出线段树的写法，大家也可以去了解一下。

**虚假的强制在线**

这一启发来自于 Codeforces 上一道有趣的题目。

???+note "[Forced Online Queries Problem](https://codeforces.com/contest/1217/problem/F)"
    给定一个初始无边的含有 n 个点的无向图（标号从 1 开始），共 q 次操作。操作有两种形式（其中 last 代表上一次 2 类操作的结果）：

    - $1 \ x \ y \ (1 \le x, y \le n, \ x \neq y)$ ：在点 $(x + last - 1) \bmod (n + 1)$ 和点 $(y + last - 1) \bmod (n + 1)$ 间添加一条无向边；
    - $2 \ x \ y \ (1 \le x, y \le n, \ x \neq y)$ ：查询点 $(x + last - 1) \bmod (n + 1)$ 和点 $(y + last - 1) \bmod (n + 1)$ 是否连通。

    数据范围：$2 \le n, q \le 10^5$
 
虽说这道题看起来强制在线了，但是 $last \in [0, 1]$ ，其实可能的操作种数顶多就 2q 个。因此我们不妨考虑把所有操作都加入线段树分治，然后处理的时候只选取有效的那个处理。

但是怎么判这个有效呢？我们是不是只有在执行完其之前最接近的 2 类操作才能知道当前操作哪一个有效。换言之，只要在分治过程中能保证在处理每一条边前在其前面的 2 类操作已经被处理过就行了。由于查询都在叶子上，所以我们单独开一个数组记录第 i 时刻的查询是什么（即直接记录在线段树的叶子上）。

我们可以施加一点小技巧。原本边的出现时间是区间 (s, t)，我们现在把它改成 (s + 1, t)。这一修改导致的结果非常微妙：首先，在 s 时刻的操作其实是加边，不会同时进行查询，因此这一修改不会影响答案；其次，这样的修改保证了我们在把这一条边加入并查集是，s 时刻对应的叶子节点一定是被遍历过的，即我们已经知道了 last 确切的值，因此我们就可以判断出哪些操作是有效的了。

详细的实现大家可以参考一下[我的代码](https://github.com/codgician/Competitive-Programming/blob/master/Codeforces/1217F/divide_and_conquer_disjoint_set_heuristic.cpp)。

## 基于数值的离线分治

### 整体二分

大家对于二分算法大概是非常熟悉的，但是我们接触到的传统的二分往往是针对单个询问进行二分。而整体二分则更进一步，对多个同类操作（包括修改和查询）一起二分答案，同时依据当前二分的判定值对当前操作集进行分类，再应用分治思想分而治之。而这一算法的经典应用即为区间第 k 小。我们不妨以其作为例子来介绍整体二分。

**静态区间第 k 小** 

???+note "[Kth Number](http://bailian.openjudge.cn/practice/2104/)"
    对于一个长度为 n 的数列，现有 q 次询问。每次询问给出 l, r, k，试求数列中下标在 \[l, r\] 中第 k 小的数的数值。

    数据范围：$1 \le n \le 10^5, \ 1 \le q \le 5 \times 10^3$

如果是对于单次查询，我们显然是可以先对区间进行预处理然后进行二分的。一种显而易见的思路就是将数值离散化然后开一个数组来记录每一个值在区间内出现的次数然后对前缀和进行二分。但是，当我们面对多个询问的时候，如果对每一个询问都进行预处理然后再二分，这样的复杂度显然是无法接受的。

不难发现对于每个询问其实预处理的过程都是大致相同的。那么我们是否可以对多个询问同时进行二分操作呢？

假设题目给定了由 n 个询问组成的一个询问集合 Q，并且我们知道所有出现过的数值的区间为 \[l, r\]。二分过程中，我们记 $mid = \frac{1}{2}(l+r)$ 。在每一层递归中，我们首先对于 Q 内每一个询问进行处理，求得该询问区间中数值在 \[l, mid\] 范围内的值的个数 cnt。接下来，我们以此将 Q 中的询问分为两类：$cnt \ge k$ 的以及 $cnt < k$ 的。我们用 $Q_1$ 和 $Q_2$ 来表示这两类询问。

对于 $Q_1$​ 中的询问，我们显然可以在 \[l, mid\] 这一范围中求得答案。换句话说，\[l, r\] 的第 k 小即 \[l, mid\] 中的第 k 小；而对于 $Q_2$ 中的询问，我们便可以得知答案坐落于 (mid, r] 这一范围中。显然，求 \[l, r\] 中的第 k 小等价于求 (mid, r] 中的第 k−cnt 小（我们需要剔除 [l, mid) 中的数）。通过这种方式，我们可以把问题 Q 拆分成 $Q_1$ 和 $Q_2$ 两个子问题然后对它们进行分治并递归求解。递归结束于 l = r，记此时的询问集为 $Q'$ ，则 $Q'$ 中的所有询问的答案即为 r，便可以求解问题了。

我们最后需要解决的问题便是如何求得 cnt 了。我们可以考虑将原始数列中的数值离散化，然后用一个树状数组来维护这一信息。树状数组的第 i 位表示数列中第 i 位上的值是否坐落于 \[l, mid\] 范围内，因此我们只需要在树状数组上求\[L, R\] 区间的区间和便可以得知下标 \[L, R\] 间有多少个数坐落于 \[l, mid\] 范围内，即 cnt。

具体实现时，我们可以在记录数列中每个数的原始下标后对数列按数值从小到大排序。我们可以通过二分搜索快速查找到 l 的位置，接下来我们便可以轻松地对于 \[l, mid\] 中的每个值更新树状数组中的信息。再求得 cnt 后我们再逐一撤销我们方才对树状数组进行的更改以为接下来递归中求 cnt 做准备。需要注意的是，撤销更改时不可以直接用 memset，若是如此每一次预处理就与原数组长度有关而并非与当前二分区间有关了，而这会导致复杂度的变化。显然，这一过程复杂度是 $\mathcal{O}(m\log{n})$ 的，其中 m 是当前二分区间的长度。总的复杂度是 $\mathcal{O}(n\log^2{n})$ 级别的。

最后附上[我的代码](https://github.com/codgician/Competitive-Programming/blob/master/POJ/2104/overall_binary_search_binary_indexed_tree.cpp) 以供参考。

???+note "参考代码"

    ```cpp
    #include <bits/stdc++.h>
    using namespace std;

    #define SIZE 100100

    typedef struct _Node {
        int val, id;
        bool operator < (const struct _Node & snd) const {
            return val < snd.val;
        }
    } Node;

    Node arr[SIZE];

    typedef struct _Query {
        int qLeftPt, qRightPt, k;
        int id, cnt;
    } Query;

    Query qArr[SIZE], bkt[SIZE];
    int ans[SIZE], bit[SIZE], len, qNum;

    int lowbit(int n) {
        return n & -n;
    }

    void add(int pos, int val) {
        for (int i = pos; i <= len; i += lowbit(i))
            bit[i] += val;
    }

    int getPrefixSum(int pos) {
        int ret = 0;
        for (int i = pos; i > 0; i -= lowbit(i))
            ret += bit[i];
        return ret;
    }

    void divideConquer(int headPt, int tailPt, int leftPt, int rightPt) {
        if (leftPt == rightPt) {
            for (int i = headPt; i <= tailPt; i++)
                ans[qArr[i].id] = rightPt;
            return;
        }
        int midPt = (leftPt + rightPt) >> 1;
        // First element that is not smaller than leftPt
        int pos = lower_bound(arr + 0, arr + len, Node{leftPt, -1}) - arr;
        // Update BIT
        for (int i = pos; i < len && arr[i].val <= midPt; i++)
            add(arr[i].id + 1, 1);
        // Calculate cnt
        for (int i = headPt; i <= tailPt; i++)
            qArr[i].cnt = getPrefixSum(qArr[i].qRightPt + 1) - getPrefixSum(qArr[i].qLeftPt);
        // Restore BIT
        for (int i = pos; i < len && arr[i].val <= midPt; i++)
            add(arr[i].id + 1, -1);
        // Divide And Conquer
        int bktHeadPt = headPt, bktTailPt = tailPt;
        for (int i = headPt; i <= tailPt; i++) {
            if (qArr[i].cnt >= qArr[i].k) {
                bkt[bktHeadPt++] = qArr[i];
            } else {
                qArr[i].k -= qArr[i].cnt;
                bkt[bktTailPt--] = qArr[i];
            }
        }
        for (int i = headPt; i <= tailPt; i++)
            qArr[i] = bkt[i];
        if (bktHeadPt != headPt)
            divideConquer(headPt, bktHeadPt - 1, leftPt, midPt);
        if (bktTailPt != tailPt)
            divideConquer(bktTailPt + 1, tailPt, midPt + 1, rightPt);
    }

    int main() {
        ios::sync_with_stdio(false);
        cin.tie(0); cout.tie(0);
        while (cin >> len >> qNum) {
            memset(bit, 0, sizeof(bit));
            int minVal = INT_MAX, maxVal = INT_MIN;
            for (int i = 0; i < len; i++) {
                cin >> arr[i].val;
                minVal = min(minVal, arr[i].val);
                maxVal = max(maxVal, arr[i].val);
                arr[i].id = i;
            }
            sort(arr + 0, arr + len);
            for (int i = 0; i < qNum; i++) {
                cin >> qArr[i].qLeftPt >> qArr[i].qRightPt >> qArr[i].k;
                qArr[i].qLeftPt--; qArr[i].qRightPt--;
                qArr[i].id = i; qArr[i].cnt = 0;
            }
            divideConquer(0, qNum - 1, minVal, maxVal);
            for (int i = 0; i < qNum; i++)
                cout << ans[i] << '\n';
        }
        return 0;
    }
    ```

**动态区间第 k 小**

???+note "[Dynamic Rankings](https://zoj.pintia.cn/problem-sets/91827364500/problems/91827365611)"
    对于一个长度为 n 的数列，现有两类共 q 次操作：

    - 将第 i 个数的值修改为 v；
    - 询问第 l 个数和第 r 个数之间第 k 小的数值。

    数据范围：$1 \le n \le 5 \times 10^4, \ 1 \le q \le 10^4$

本题与上一例唯一的区别便在于新加入了修改操作，这样一来上例中的询问集 Q 在本题中就变成可以含有询问及修改两种操作的操作集了。我们不妨把每个修改操作拆成两步：先删除该位置上原值，再在该位置上插入新值。同时，序列的初始化也可看作 n 次在位置 i 插入新值。

我们注意到每一次修改操作其实对答案的贡献是独立的。具体地说，假设二分过程中当前某区间值域为 \[L, R\] ，当前二分区间为 \[l, r\] 。我们若是将该区间内某位置上的值由 $a_i$ 改变为 $a_i' \ (l \le a_i' \le r)$ ，并不会改变 \[l, r\] 以外的区间（即 [L, l) 和 (r, R] 这两个区间对答案的贡献。 由此，每一次预处理的复杂度依然仅仅是与当前二分区间有关的，可以使用整体二分。

故我们依然采用与上例类似的思路对所有操作进行二分。不同之处在于，由于修改这一操作的引入，我们不能随意改变操作的顺序。因此，区别仅仅在于求 cnt 这一过程的实现。我们不能再像之前一样先对值排序然后二分，因为那样会改变操作的顺序。对于 Q 内的操作，我们必须按照原顺序进行遍历，查询和修改同时进行（若操作为修改，只应用值在 \[l,mid\] 区间之内的）。另外在将 Q 分类为 $Q_1$ 和 $Q_2$ 时也要单独开两个单独的空间来进行分类以防止改变操作之间的顺序。

最后附上[我的代码](https://github.com/codgician/Competitive-Programming/blob/master/ZOJ/2112/overall_binary_search_edit_binary_indexed_tree.cpp)以供参考。

???+note "参考代码"

    ```cpp
    #include <bits/stdc++.h>
    using namespace std;

    #define SIZE 50010
    #define QUE_SIZE 10010
    #define OPR_SIZE 70010

    // If k == 0: modify;
    // qLeftPt = pos, qRightPt = val, cnt = type (-1: delete | 1: add)
    typedef struct _Opr {
        int qLeftPt, qRightPt, k;   // pos, val, 0
        int qId, cnt;            // -1, type
    } Opr;

    Opr oprArr[OPR_SIZE], fstBkt[OPR_SIZE], sndBkt[OPR_SIZE];
    int arr[SIZE], ans[QUE_SIZE], bit[SIZE], len;

    int lowbit(int n) {
        return n & -n;
    }

    void add(int pos, int val) {
        for (int i = pos; i <= len; i += lowbit(i))
            bit[i] += val;
    }

    int prefixSum(int pos) {
        int ret = 0;
        for (int i = pos; i > 0; i -= lowbit(i))
            ret += bit[i];
        return ret;
    }

    void divideConquer(int headPt, int tailPt, int leftPt, int rightPt) {
        if (leftPt == rightPt) {
            for (int i = headPt; i <= tailPt; i++) {
                if (oprArr[i].k == 0)   // Skip modifications
                    continue;
                ans[oprArr[i].qId] = rightPt;
            }
            return;
        }

        int midPt = (leftPt + rightPt) >> 1;

        // Update BIT && Calculate cnt
        for (int i = headPt; i <= tailPt; i++) {
            if (oprArr[i].k > 0)    // query
                oprArr[i].cnt = prefixSum(oprArr[i].qRightPt + 1) - prefixSum(oprArr[i].qLeftPt);
            else if (oprArr[i].qRightPt <= midPt)   // modify
                add(oprArr[i].qLeftPt + 1, oprArr[i].cnt);
        }

        // Restore BIT
        for (int i = headPt; i <= tailPt; i++)
            if (oprArr[i].k == 0 && oprArr[i].qRightPt <= midPt)
                add(oprArr[i].qLeftPt + 1, -oprArr[i].cnt);

        // Divide And Conquer
        int fstPt = 0, sndPt = 0;
        for (int i = headPt; i <= tailPt; i++) {
            if (oprArr[i].k > 0) {
                // query
                if (oprArr[i].k <= oprArr[i].cnt) {
                    fstBkt[fstPt++] = oprArr[i];
                } else {
                    oprArr[i].k -= oprArr[i].cnt;
                    sndBkt[sndPt++] = oprArr[i];
                }
            } else {
                // modify
                if (oprArr[i].qRightPt <= midPt)
                    fstBkt[fstPt++] = oprArr[i];
                else
                    sndBkt[sndPt++] = oprArr[i];
            }
        }
        for (int i = 0; i < fstPt; i++)
            oprArr[headPt + i] = fstBkt[i];
        for (int i = 0; i < sndPt; i++)
            oprArr[headPt + fstPt + i] = sndBkt[i];
        if (fstPt > 0)
            divideConquer(headPt, headPt + fstPt - 1, leftPt, midPt);
        if (sndPt > 0)
            divideConquer(headPt + fstPt, tailPt, midPt + 1, rightPt);
    }

    int main() {
        ios::sync_with_stdio(false);
        cin.tie(0); cout.tie(0);
        int caseNum;
        cin >> caseNum;
        while (caseNum--) {
            memset(bit, 0, sizeof(bit));
            int qNum, oprPt = 0, qPt = 0, minVal = INT_MAX, maxVal = 0;
            cin >> len >> qNum;
            for (int i = 0; i < len; i++) {
                // treat original array as insertions
                cin >> arr[i];
                minVal = min(minVal, arr[i]);
                maxVal = max(maxVal, arr[i]);
                oprArr[oprPt++] = {i, arr[i], 0, -1, 1};
            }
            for (int i = 0; i < qNum; i++) {
                char opr; cin >> opr;
                if (opr == 'Q') {
                    int qLeftPt, qRightPt, k;
                    cin >> qLeftPt >> qRightPt >> k;
                    qLeftPt--, qRightPt--;
                    oprArr[oprPt++] = {qLeftPt, qRightPt, k, qPt++, 0};
                } else if (opr == 'C') {
                    int pos, val;
                    cin >> pos >> val;
                    pos--;
                    // delete
                    oprArr[oprPt++] = {pos, arr[pos], 0, -1, -1};
                    // insert
                    oprArr[oprPt++] = {pos, val, 0, -1, 1};
                    arr[pos] = val;
                    minVal = min(minVal, val);
                    maxVal = max(maxVal, val);
                }
            }
            divideConquer(0, oprPt - 1, minVal, maxVal);
            for (int i = 0; i < qPt; i++)
                cout << ans[i] << endl;
        }
        return 0;
    }
    ```