## 线段树与离线询问

线段树与离线询问结合的问题在 OI 领域也有出现。

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

假设一个数据结构，允许以 $O(T(n))$ 的时间复杂度添加元素。本文描述一种允许以 $O(T(n)\log n)$ 的时间复杂度进行离线删除的技术。

### 算法

从被添加开始到被删除结束，每个元素都在数据结构中存活一段时间。

在查询上构建一个线段树。元素处于活动状态的线段将拆分变为树的 $O(\log n)$ 节点。查询有关结构的信息时，将每个查询放入相应的叶节点中。

为了处理所有查询，在段树上运行深度优先搜索。进入节点时，添加该节点中的所有元素。然后进一步查找该节点的子节点，如果该节点是叶节点则回答查询。离开节点时撤消添加。

如果更改结构的时间复杂度为 $O(T(n))$，可以通过设置一个栈保留更改，以 $O(T(n))$ 的时间复杂度回滚。回滚不维持均摊复杂度。

### 注意

当某个对象处于活动状态时，在线段上创建段树的想法不仅适用于数据结构问题。

### 实现

此实现用于动态连接问题。它可以添加边、删除边和计算连接的零部件数量。

```cpp
struct dsu_save {
  int v, rnkv, u, rnku;

  dsu_save() {}

  dsu_save(int _v, int _rnkv, int _u, int _rnku)
      : v(_v), rnkv(_rnkv), u(_u), rnku(_rnku) {}
};

struct dsu_with_rollbacks {
  vector<int> p, rnk;
  int comps;
  stack<dsu_save> op;

  dsu_with_rollbacks() {}

  dsu_with_rollbacks(int n) {
    p.resize(n);
    rnk.resize(n);
    for (int i = 0; i < n; i++) {
      p[i] = i;
      rnk[i] = 0;
    }
    comps = n;
  }

  int find_set(int v) { return (v == p[v]) ? v : find_set(p[v]); }

  bool unite(int v, int u) {
    v = find_set(v);
    u = find_set(u);
    if (v == u) return false;
    comps--;
    if (rnk[v] > rnk[u]) swap(v, u);
    op.push(dsu_save(v, rnk[v], u, rnk[u]));
    p[v] = u;
    if (rnk[u] == rnk[v]) rnk[u]++;
    return true;
  }

  void rollback() {
    if (op.empty()) return;
    dsu_save x = op.top();
    op.pop();
    comps++;
    p[x.v] = x.v;
    rnk[x.v] = x.rnkv;
    p[x.u] = x.u;
    rnk[x.u] = x.rnku;
  }
};

struct query {
  int v, u;
  bool united;

  query(int _v, int _u) : v(_v), u(_u) {}
};

struct QueryTree {
  vector<vector<query>> t;
  dsu_with_rollbacks dsu;
  int T;

  QueryTree() {}

  QueryTree(int _T, int n) : T(_T) {
    dsu = dsu_with_rollbacks(n);
    t.resize(4 * T + 4);
  }

  void add_to_tree(int v, int l, int r, int ul, int ur, query& q) {
    if (ul > ur) return;
    if (l == ul && r == ur) {
      t[v].push_back(q);
      return;
    }
    int mid = (l + r) / 2;
    add_to_tree(2 * v, l, mid, ul, min(ur, mid), q);
    add_to_tree(2 * v + 1, mid + 1, r, max(ul, mid + 1), ur, q);
  }

  void add_query(query q, int l, int r) { add_to_tree(1, 0, T - 1, l, r, q); }

  void dfs(int v, int l, int r, vector<int>& ans) {
    for (query& q : t[v]) {
      q.united = dsu.unite(q.v, q.u);
    }
    if (l == r)
      ans[l] = dsu.comps;
    else {
      int mid = (l + r) / 2;
      dfs(2 * v, l, mid, ans);
      dfs(2 * v + 1, mid + 1, r, ans);
    }
    for (query q : t[v]) {
      if (q.united) dsu.rollback();
    }
  }

  vector<int> solve() {
    vector<int> ans(T);
    dfs(1, 0, T - 1, ans);
    return ans;
  }
};
```

**本页面主要译自博文 [Deleting from a data structure](https://cp-algorithms.com/data_structures/deleting_in_log_n.html)，版权协议为 CC-BY-SA 4.0。**

### 习题

[Codeforces - Connect and Disconnect](https://codeforces.com/gym/100551/problem/A)

[Codeforces - Addition on Segments](https://codeforces.com/contest/981/problem/E)

[Codeforces - Extending Set of Points](https://codeforces.com/contest/1140/problem/F)
