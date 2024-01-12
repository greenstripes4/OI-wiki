author: xiezheyuan

线段树与离线询问结合的问题在 OI 领域也有出现。这种技巧又被称为线段树分治。

假设一个数据结构，允许以 $O(T(n))$ 的时间复杂度添加元素。本文描述一种允许以 $O(T(n)\log n)$ 的时间复杂度进行离线删除的技术。

假如你需要维护一些信息，这些信息会在某一个时间段内出现，要求在离线的前提下回答某一个时刻的信息并，则可以考虑使用线段树分治的技巧。

实际上线段树分治常用于不带删的数据结构转成可以带删的数据结构，抑或是对于某一个属性的信息分别计算。

### 例题

???+note "[HH的项链](https://www.luogu.com.cn/problem/P1972)"
    给定一个静态数列，进行多个区间查询，求各个区间上不同的数的个数

这个题用树状数组，线段树等等都可以做，不过用树状数组写起来更方便。

此题首先应考虑到这样一个结论：

对于若干个询问的区间\[l,r\]，如果他们的r都相等的话，那么数列中出现的同一个数字，一定是只关心出现在最右边的那一个的，例如：

数列是：1 3 4 5 1

那么，对于r=5的所有的询问来说，第一个位置上的1完全没有意义，因为r已经在第五个1的右边，对于任何查询的\[L,5\]区间来说，如果第一个1被算了，那么他完全可以用第五个1来替代。

因此，我们可以对所有查询的区间按照r来排序，然后再来维护一个树状数组，这个树状数组是用来干什么的呢？看下面的例子：

1 2 1 3

对于第一个1，insert(1,1)；表示第一个位置出现了一个不一样的数字，此时树状数组所表示的每个位置上的数字（不是它本身的值而是它对应的每个位置上的数字）是：1 0 0 0

对于第二个2，insert(2,1)；此时树状数组表示的每个数字是1 1 0 0

对于第三个1，因为之前出现过1了，因此首先把那个1所在的位置删掉insert(1,-1),然后在把它加进来insert(3,1)。此时每个数字是0 1 1 0

如果此时有一个询问\[2,3\]，那么直接求sum(3)-sum(2-1)=2就是答案。

???+note "参考代码"

    ```cpp
    #include<bits/stdc++.h>
    using namespace std;
    int a[1000010];
    struct Node{
        int L,R,post,date;
    }dian[1000010];
    bool cmp(Node p,Node q){
        return p.R<q.R;
    }
    bool cm(Node p,Node q){
        return p.post<q.post;
    }
    int vis[1000010];
    int n,m;
    int b[1000010],c[1000010];
    int lowbit(int x){
        return x&(-x);
    }
    void updata(int i,int k){//在i位置加上k
        while(i <= n){
            c[i] += k;
            i += lowbit(i);
        }
    }
    int getsum(int i){//求A[1 - i]的和
        int res = 0;
        while(i > 0){
            res += c[i];
            i -= lowbit(i);
        }
        return res;
    }
    int main(){
        scanf("%d",&n);
        for(int i=1;i<=n;i++){
            scanf("%d",&a[i]);
        }
        scanf("%d",&m);
        for(int i=1;i<=m;i++){
            scanf("%d%d",&dian[i].L,&dian[i].R);
            dian[i].post=i;
        }
        sort(dian+1,dian+1+m,cmp);
        int now=1;
        for(int i=1;i<=n;i++){
            if(vis[a[i]]){
                updata(vis[a[i]],-1);
            }
            vis[a[i]]=i;
            updata(i,1);
            while(dian[now].R==i){
                dian[now].date=getsum(dian[now].R)-getsum(dian[now].L-1);
                now++;
            }
        }
        sort(dian+1,dian+1+m,cm);
        for(int i=1;i<=m;i++) printf("%d\n",dian[i].date);
        return 0;
    } 
    ```

### 习题
- [Turing Tree](https://vjudge.net/problem/HDU-3333)
- [DQuery](https://vjudge.net/problem/SPOJ-DQUERY)

* * *

下面为OIWiki内容

* * *

## 过程

首先我们建立一个线段树来维护时刻，每一个节点维护一个 `vector` 来存储位于这一段时刻的信息。

插入一个信息到线段树中和普通线段树的区间修改是类似的。

然后我们考虑如何处理每一个时间段的信息并。考虑从根节点开始分治，维护当前的信息并，然后每到一个节点的时候将这个节点的所有信息进行合并。回溯时撤销这一部分的贡献。最后到达叶子节点时的信息并就是对应的答案。

如果更改信息的时间复杂度为 $O(T(n))$，可以通过设置一个栈保留更改，以 $O(T(n))$ 的时间复杂度撤销。撤销不维持均摊复杂度。

整个分治流程的总时间复杂度是 $O(n\log n(T(n) + M(n)))$ 的，其中 $O(M(n))$ 为合并信息的时间复杂度，空间复杂度为 $O(n\log n)$。

??? note "实现"
    ```cpp
    #define ls (i << 1)
    #define rs (i << 1 | 1)
    #define mid ((l + r) >> 1)
    
    vector<Object> tree[N << 2];  // 线段树
    
    void update(int ql, int qr, Object obj, int i, int l, int r) {  // 插入
      if (ql <= l && r <= qr) {
        tree[i].push_back(obj);
        return;
      }
      if (ql <= mid) update(ql, qr, obj, ls, l, mid);
      if (qr > mid) update(ql, qr, obj, rs, mid + 1, r);
    }
    
    stack<Object> sta;  // 用于撤销的栈
    Object now;         // 当前的信息并
    Object ans[N];      // 答案
    
    void solve(int i, int l, int r) {
      auto lvl = sta.size();  // 记录一下应当撤销到底几个
      for (Object x : tree[i]) sta.push(now), now = Merge(now, x);  // 合并信息
      if (l == r)
        ans[i] = now;  // 记录一下答案
      else
        solve(ls, l, mid), solve(rs, mid + 1, r);  // 分治
      while (sta.size() != lvl) {                  // 撤销信息
        now = sta.top();
        sta.pop();
      }
    }
    ```

## 例题

???+ note "[luogu P5787 二分图/【模板】线段树分治](https://www.luogu.com.cn/problem/P5787)"
    你需要维护一个 $n$ 个点 $m$ 条边的无向图。第 $i$ 条边为 $(x_i,y_i)$，出现的时刻为 $[l_i,r_i)$，其余时刻消失。

    对于每一个时刻，若此时该图为二分图，输出 `Yes`，否则输出 `No`。

    ??? "解题思路"
          使用种类并查集来维护一个图是否是二分图，然后就可以套用线段树分治了。

          注意可撤销的并查集不能路径压缩，只能按秩合并。
      
    ??? "参考代码"
        ```cpp
        --8<-- "docs/topic/code/segment-tree-offline/segment-tree-offline_1.cpp"
        ```

???+ note "颜色限制 restriction"
    一个 $n$ 点 $m$ 边的无向图，有 $k$ 种颜色编号为 $0\sim k-1$，每条边有一种颜色。

    对于每种颜色，请判断假如删去所有这种颜色的边，得到的图是否连通？是否是一棵树？

    输出满足删去后图连通的颜色数和删去后图是树的颜色数。

    ??? "解题思路"
        对于每一种颜色，建立一个时间，在这个时间内没有这个颜色的边，其他边都有。用一个并查集维护一下即可。

    ??? "参考代码"
        ```cpp
        --8<-- "docs/topic/code/segment-tree-offline/segment-tree-offline_2.cpp"
        ```

???+ note "[luogu P4219 \[BJOI2014\] 大融合](https://www.luogu.com.cn/problem/P4219)"
    需要维护一个 $n$ 个点的森林，初始时是散点。

    有 $q$ 个操作，支持：

    - `A x y` 连边 $(x,y)$。
    - `Q x y` 输出经过边 $(x,y)$ 的路径数。

    允许离线。

    ??? "解题思路"
        考虑允许离线，因此可以想到线段树分治。

        然后考虑如何支持 Q 操作。如果不存在 $(x,y)$ 这条边，答案就是 $x$ 所在连通块大小乘上 $y$ 所在连通块大小。可以用并查集维护。

        因此你可以将 Q 拆成三个时间 $k-1,k,k+1$。其中 $k-1$ 是这条边的终止时刻，$k+1$ 是这条边的起始时刻。这样 $k$ 就没有这条边，正好回答询问。

    ??? "参考代码"
        ```cpp
        --8<-- "docs/topic/code/segment-tree-offline/segment-tree-offline_3.cpp"
        ```

???+ note "[luogu P2056 \[ZJOI2007\] 捉迷藏](https://www.luogu.com.cn/problem/P2056)"
    出一个 $n$ 个点的树，每个点有黑白两种颜色。初始时每个点都是黑色的。$q$ 次操作，支持：

    - `C x` 将第 $x$ 个点的颜色反转。
    - `G` 询问树上两个黑色点的最远距离。特别地，若不存在黑色点，输出 $-1$。

    允许离线。

    ??? "解题思路"
        首先考虑如何维护树上点集的直径，有下面的推论：

        > 对于一个集合 $S$ 和只有一个点的集合 $\{P\}$。若集合 $S$ 的直径为 $(U,V)$。则点集 $S\cap\{P\}$ 的直径只可能为 $(U,V),(U,P)$ 或 $(V,P)$。

        然后考虑解决原问题。我们可以考虑维护黑色点集，维护每一个点在黑色点集中的若干个时间段（具体你开一个桶记录一下上一次进入黑色点集的时刻即可）。

        然后就自然地想到离线，将所有时间刻插入到线段树中。然后在线段树上分治，每次线段树上的点会记录当前时间段点集新增的点，新增点可以使用上面的推论，找到新点集直径的两个端点。

        撤销是平凡的，开一个栈记录一下直径端点的变化即可。

    ??? "参考代码"
        ```cpp
        --8<-- "docs/topic/code/segment-tree-offline/segment-tree-offline_4.cpp"
        ```

## 习题

-   [CF601E A Museum Robbery](https://codeforces.com/problemset/problem/601/E) 线段树分治 + 背包 dp。
-   [CF19E Fairy](https://codeforces.com/problemset/problem/19/E) 线段树分治 + 种类并查集。
-   [luogu P5227 \[AHOI2013\] 连通图](https://www.luogu.com.cn/problem/P5227) 线段树分治 + 并查集。
-   [luogu P4319 变化的道路](https://www.luogu.com.cn/problem/P4319) 线段树分治 + Link Cut Tree 维护最小生成树。
-   [luogu P3733 \[HAOI2017\] 八纵八横](https://www.luogu.com.cn/problem/P3733) 线段树分治 + 线性基。

**本页面部分内容参考自博文 [Deleting from a data structure](https://cp-algorithms.com/data_structures/deleting_in_log_n.html)，版权协议为 CC-BY-SA 4.0。**
