author: FFjet, ChungZH, frank-xjh, hsfzLZH1, Xarfa, AndrewWayne

本页面将简要介绍两种双向搜索算法：「双向同时搜索」和「Meet in the middle」。

## 双向同时搜索

### 定义

双向同时搜索的基本思路是从状态图上的起点和终点同时开始进行 [广搜](./bfs.md) 或 [深搜](./dfs.md)。

如果发现搜索的两端相遇了，那么可以认为是获得了可行解。

双向广搜的应用场合：有确定的起点和终点，并且能把从起点到终点的单个搜索，变换为分别从起点出发和从终点出发的“相遇”问题，可以用双向广搜。

从起点s（正向搜索）和终点t（逆向搜索）同时开始搜索，当两个搜索产生相同的一个子状态v时就结束。得到的s-v-t是一条最佳路径，当然，最佳路径可能不止这一条。

注意，和普通BFS一样，双向广搜在搜索时并没有“方向感”，所谓“正向搜索”和“逆向搜索”其实是盲目的，它们从s和t逐层扩散出去，直到相遇为止。

与只做一次BFS相比，双向BFS能在多大程度上改善算法复杂度？下面以网格图和树形结构为例，推出一般性结论。

（1）网格图。

用BFS求下面图中两个黑点s和t间的最短路。左图是一个BFS；右图是双向BFS，在中间的五角星位置相遇。

![](./images/bidirectional-1.png)

（1）单个BFS         （2）双向BFS

图1 网格图搜索

设两点的距离是k。左边的BFS，从起点s扩展到t，一共访问了2k(k+1)≈2k2个结点；右边的双向BFS，相遇时一共访问了约k2个结点。两者差2倍，改善并不明显。

在这个网格图中，BFS扩展的第k和第k+1层，结点数量相差(k+1)/k倍，即结点数量是线性增长的。

（2）树形结构。

![](./images/bidirectional-2.png)

（1）单个BFS         （2）双向BFS

图2 二叉树搜索

以二叉树为例，求根结点s到最后一行的黑点t的最短路。

左图做一次BFS，从第1层到第k-1层，共访问1+2+...+2k−1≈2k个结点。右图是双向BFS，分别从上往下和从下往上BFS，在五角星位置相遇，共访问约2×2k/2个结点。双向广搜比做一次BFS改善了2k/2倍，优势巨大。

在二叉树的例子中，BFS扩展的第k和第k+1层，结点数量相差2倍，即结点数量是指数增长的。
从上面2个例子可以得到一般性结论：

1. 做BFS扩展的时候，下一层结点（一个结点表示一个状态）数量增加越快，双向广搜越有效率。
2. 是否用双向广搜替代普通BFS，除了（1）以外，还应根据总状态数量的规模来决定。双向BFS的优势，从根本上说，是它能减少需要搜索的状态数量。有时虽然下一层数量是指数增长的，但是由于去重或者限制条件，总状态数并不多，也就没有必要使用双向BFS。例如后面的例题“hdu 1195 open the lock”，密码范围1111～9999，共约9000种，用BFS搜索时，最多有9000个状态进入队列，就没有必要使用双向BFS。而例题HDU 1401 Solitaire，可能的棋局状态有1500万种，走8步棋会扩展出168种状态，大于1500万，相当于扩展到所有可能的棋局，此时应该使用双向BFS。

很多教材和网文讲解双向广搜时，常用八数码问题做例子。下图演示了从状态A移动到状态F的搜索过程。

八数码共有9! = 362880种状态，不太多，用普通BFS也行。不过，用双向广搜更好，因为八数码每次扩展，下一层的状态数量是上一层的2～4倍，比二叉树的增长还快，效率的提升也就更高。

![](./images/bidirectional-3.png)

图3 八数码问题的搜索树

### 过程

双向广搜的队列，有两种实现方法：

1. 合用一个队列。正向BFS和逆向BFS用同一个队列，适合正反2个BFS平衡的情况。正向搜索和逆向搜索交替进行，两个方向的搜索交替扩展子状态，先后入队。直到两个方向的搜索产生相同的子状态，即相遇了，结束。这种方法适合正反方向扩展的新结点数量差不多的情况，例如上面的八数码问题。
2. 分成两个队列。正向BFS和逆向BFS的队列分开，适合正反2个BFS不平衡的情况。让子状态少的BFS先扩展下一层，另一个子状态多的BFS后扩展，可以减少搜索的总状态数，尽快相遇。例题见后面的“洛谷p1032 字串变换”。

和普通BFS一样，双向广搜在扩展队列时也需要处理去重问题。把状态入队列的时候，先判断这个状态是否曾经入队，如果重复了，就丢弃。

双向广搜的步骤：

```text
将开始结点和目标结点加入队列 q
标记开始结点为 1
标记目标结点为 2
while (队列 q 不为空)
{
  从 q.front() 扩展出新的 s 个结点
  
  如果 新扩展出的结点已经被其他数字标记过
    那么 表示搜索的两端碰撞
    那么 循环结束
  
  如果 新的 s 个结点是从开始结点扩展来的
    那么 将这个 s 个结点标记为 1 并且入队 q 
    
  如果 新的 s 个结点是从目标结点扩展来的
    那么 将这个 s 个结点标记为 2 并且入队 q
}
```

### 例题

???+note "[open the lock](http://acm.hdu.edu.cn/showproblem.php?pid=1195)"
    **题目描述：** 打开密码锁。密码由四位数字组成，数字从1到9。可以在任何数字上加上1或减去1，当'9'加1时，数字变为'1'，而'1'减1时，数字变为'9'。相邻的数字可以交换。每个动作是一步。任务是使用最少的步骤来打开锁。注意：最左边的数字不是最右边的数字的邻居。
    
    **输入：** 输入文件以整数T开头，表示测试用例的数量。
    
    每个测试用例均以四位数N开头，指示密码锁定的初始状态。然后紧跟另一行带有四个下标M的行，指示可以打开锁的密码。每个测试用例后都有一个空白行。
    
    **输出：** 对于每个测试用例，一行中打印最少的步骤。
    
    样例输入：
    
    2
    
    1234
    
    2144
    
    1111
    
    9999
    
    样例输出：
    
    2
    
    4

    ??? tip
        题目中的4位数字，走一步能扩展出11种情况；如果需要走10步，就可能有11^10种情况，数量非常多，看起来用双向广搜能大大提高搜索效率。不过，这一题用普通BFS也行，因为并没有11^10种情况，密码范围1111～9999，只有约9000种。用BFS搜索时，最多有9000个状态进入队列，没有必要使用双向广搜。
        
        密码进入队列时，应去重，去掉重复的密码。去重用hash最方便。

???+note "[Solitaire](http://acm.hdu.edu.cn/showproblem.php?pid=1401)"
    8×8的方格，放4颗棋子在初始位置，给定4个最终位置，问在8步内是否能从初始位置走到最终位置。规则：每个棋子能上下左右移动，若4个方向已经有一棋子则可以跳到下一个空白位置。例如，图中(4,4)位置的棋子有4种移动方法。

    ![](./images/bidirectional-4.png)

    ??? tip

        在8×8的方格上放4颗棋子，有64×63×62×61≈1500万种布局。走一步棋，4颗棋子共有16种走法，连续走8步棋，会扩展出168种棋局，168大于1500万，走8步可能会遍历到1500万棋局。
        
        此题应该使用双向BFS。从起点棋局走4步，从终点棋局走4步，如果能相遇就有一个解，共扩展出2*164=131072种棋局，远远小于1500万。
        
        本题也需要处理去重问题，扩展下一个新棋局时，看它是否在队列中处理过。用hash的方法，定义char vis[8][8][8][8][8][8][8][8]表示棋局，其中包含4个棋子的坐标。当vis=1时表示正向搜过这个棋局，vis=2表示逆向搜过。例如4个棋子的坐标是(a.x, a.y)、(b.x, b.y)、(c.x, c.y)、(d.x, d.y)，那么：
        
        vis[a.x][a.y][b.x][b.y][c.x][c.y][d.x][d.y] = 1
        
        表示这个棋局被正向搜过。
        
        4个棋子需要先排序，然后再用vis记录。如果不排序，一个棋局就会有很多种表示，不方便判重。
        
        char vis[8][8][8][8][8][8][8][8] 用了8^8 = 16M空间。如果定义为int,占用64M空间，超过题目的限制。

???+note "[Eleven puzzle](http://acm.hdu.edu.cn/showproblem.php?pid=3095)"
    如图是13个格子的拼图，数字格可以移动到黑色格子。左图是开始局面，右图是终点局面。一次移动一个数字格，问最少移动几次可以完成。

    ![](./images/bidirectional-5.png)

    ??? tip
        1. 可能的局面有13!，极大。
        2. 用一个BFS，复杂度过高。每次移动1个黑格，移动方法最少1种，最多8种。如果移动10次，那么最多有810 ≈ 10亿种。
        3. 用双向广搜，能减少到2×8^5 = 65536种局面。
        4. 判重：可以用hash，或者用STL的map。

???+note "[字串变换](https://www.luogu.com.cn/problem/P1032)"
    **题目描述：** 已知有两个字串A,B及一组字串变换的规则（至多6个规则）:
    
    A1->B1
    
    A2->B2
    
    规则的含义为：在A中的子串 A1可以变换为B1，A2可以变换为 B2…。
    
    例如：A=abcd，B=xyz，
    
    变换规则为：
    
    abc→xu，ud→y，y→yz
    
    则此时，A可以经过一系列的变换变为B，其变换的过程为：
    
    abcd→xud→xy→xyz。
    
    共进行了3次变换，使得A变换为B。
    
    **输入输出：** 给定字串A、B和变换规则，问能否在10步内将A变换为B，输出最少的变换步数。字符串长度的上限为20。


    ??? tip
        1. 若用一个BFS，每层扩展6个规则，经过10步，共610 ≈ 6千万次变换。
        2. 用双向BFS，可以用2*65 = 15552次变换搜完10步。
        3. 用两个队列分别处理正向BFS和逆向BFS。由于起点和终点的串不同，它们扩展的下一层数量也不同，也就是进入2个队列的串的数量不同，先处理较小的队列，可以加快搜索速度。2个队列见下面的代码示例[完整代码](https://blog.csdn.net/qq_45772483/article/details/104504951)。

        ```cpp
        void bfs(string A, string B){        //起点是A，终点是B
            queue <string> qa, qb;           //定义2个队列
            qa.push(A);                      //正向队列
            qb.push(B);                      //逆向队列 
            while(qa.size() && qb.size()){   
                if (qa.size() < qb.size())   //如果正向BFS队列小，先扩展它
                    extend(qa, ...);         //扩展时，判断是否相遇
                else                         //否则扩展逆向BFS
                    extend(qb, ...);         //扩展时，判断是否相遇
            }
        }
        ```

## Meet in the middle

???+ warning
    本节要介绍的不是 [**二分搜索**](../basic/binary.md)（二分搜索的另外一个译名为「折半搜索」）。

### 引入

Meet in the middle 算法没有正式译名，常见的翻译为「折半搜索」、「双向搜索」或「中途相遇」。

它适用于输入数据较小，但还没小到能直接使用暴力搜索的情况。

### 过程

Meet in the middle 算法的主要思想是将整个搜索过程分成两半，分别搜索，最后将两半的结果合并。

### 性质

暴力搜索的复杂度往往是指数级的，而改用 meet in the middle 算法后复杂度的指数可以减半，即让复杂度从 $O(a^b)$ 降到 $O(a^{b/2})$。

### 例题

???+note "[4 Sum](http://poj.org/problem?id=2785)"
    ![](./images/middle-1.png)

    ??? tip
        ![](./images/middle-2.png)

???+note "超大背包问题"
    ![](./images/middle-3.png)

    ??? tip
        ![](./images/middle-4.png)

???+note "例题 [「USACO09NOV」灯 Lights](https://www.luogu.com.cn/problem/P2962)"
    有 $n$ 盏灯，每盏灯与若干盏灯相连，每盏灯上都有一个开关，如果按下一盏灯上的开关，这盏灯以及与之相连的所有灯的开关状态都会改变。一开始所有灯都是关着的，你需要将所有灯打开，求最小的按开关次数。
    
    $1\le n\le 35$。

    ??? tip
        如果这道题暴力 DFS 找开关灯的状态，时间复杂度就是 $O(2^{n})$, 显然超时。不过，如果我们用 meet in middle 的话，时间复杂度可以优化至 $O(n2^{n/2})$。meet in middle 就是让我们先找一半的状态，也就是找出只使用编号为 $1$ 到 $\mathrm{mid}$ 的开关能够到达的状态，再找出只使用另一半开关能到达的状态。如果前半段和后半段开启的灯互补，将这两段合并起来就得到了一种将所有灯打开的方案。具体实现时，可以把前半段的状态以及达到每种状态的最少按开关次数存储在 map 里面，搜索后半段时，每搜出一种方案，就把它与互补的第一段方案合并来更新答案。

    ??? note "参考代码"
        ```cpp
        --8<-- "docs/search/code/bidirectional/bidirectional_1.cpp"
        ```

## 习题

??? note "[Subset](http://poj.org/problem?id=3977)"
    给你一个集合N（N<=35）,问集合的子集，除了空集，使得自己中所有元素和的绝对值最小，若存在多个值，那么选择子集中元素最少的那个。

    ??? tip
        1.n 最大到 35，每个数有选、不选两种可能，最多有 $2^{35}$ 个子集,因此暴力枚举的话，一定会Time Limit Exceed，采用折半枚举的思想，分成两个集合，这样每边最多 18 个元素，分别进行枚举，复杂度降到 $2^{18}$ 
        
        2.利用二进制将和以及元素个数存在两个数组中，先预判是否满足题意，再将其中一个元素和取相反数后排序，因为总元素和越接近零越好，再二分查找即可，用lower_bound时考虑查找到的下标和他前一个下标，比较元素和以及元素个数，不断更新即可。
        
        3.由于是求大数的绝对值，此时需要开函数，（不知道为什么，我调用labs函数，结果是错误的，所以自己写了一个）
        
        4.然后枚举其中一个子集，排序后暂存后，再枚举另一个子集，通过二分查找第一个集合中与该值的相反数最接近的元素，要注意的是如果有多个元素与相反值最接近，取数的个数最小的那一个。
        
        5.与寻找合适的子集并与第一个集合的子集相加，从而找到绝对值最小的子集.

    ??? note "参考代码"

        ```cpp
        #include<stdio.h>
        #include<map>
        #include<string.h>
        #include<math.h>
        #include<algorithm>
        #include<iostream>
        #define inf 0x3f3f3f3f3f3f3f3f
        using namespace std;
        typedef long long ll;
        const int M=50;
        int n,cnt=0;
        map<ll,ll> vis;
        ll Labs(ll x){
            if(x>0) return x;
            return -x;
        }
        ll a[M],dp[1<<20],ans;//vis当前所选的最少个数,dp数组存储前一半的值，ans是最终结果
        ll solve(ll num)
        {
            if(num<=dp[1])
                return dp[1];
            else if(num>=dp[cnt])
                return dp[cnt];
            int mid=upper_bound(dp+1,dp+cnt+1,num)-dp;
            if(Labs(dp[mid]-num)==Labs(dp[mid-1]-num)){
                if(vis[dp[mid-1]]<vis[dp[mid]])
                    return dp[mid-1];
                return dp[mid];
            }
            else if(Labs(dp[mid]-num)>Labs(dp[mid-1]-num))
                return dp[mid-1];
            return dp[mid];
        }//查找与x最接近的元素，要注意的是如果有多个元素与x最接近，取数的个数最小的那一个
        int main()
        {
            while(~scanf("%d",&n)&&n)
            {
                cnt=0,ans=inf;
                vis.clear();
                memset(a,0,sizeof(a));
                memset(dp,0,sizeof(dp));
                for(int i=1; i<=n; i++)
                    scanf("%lld",&a[i]);
                if(n==1){
                    printf("%lld 1\n",Labs(a[1]));
                    continue;
                }
                int x=n/2,y=n-x;
                int mi=(1<<x)-1;/*mi为所选元素的所有可能性，除去空集*/
                ll id=inf;
                for(int i=1; i<=mi; i++){
                    ll now=0,tot=0;
                    for(int j=1; j<=x; j++){
                        if(i&(1<<(j-1)))/**枚举i为多少种方式，j为多少个数，用j来控制位数，i控制方式*/
                            now+=a[j],tot++;
                    }
                    if(vis[now])
                        vis[now]=min(vis[now],tot);//如果当前值已经出现过，元素个数取较小的
                    else
                        vis[now]=tot,dp[++cnt]=now;//没有出现过，建立映射关系
                    if(Labs(now)<ans){
                        ans=Labs(now);
                        id=tot;
                    }//如果答案更优，更新答案
                    else if(Labs(now)==ans)
                        id=min(id,tot);//如果答案相同，元素个数取较小的
                }
                sort(dp+1,dp+cnt+1);
                for(int i=1; i<=(1<<y)-1; i++){
                    ll now=0,tot=0;
                    for(int j=1; j<=y; j++)
                        if(i&(1<<(j-1)))
                            now+=a[j+n/2],tot++;
                    if(Labs(now)<ans){
                        ans=Labs(now);
                        id=tot;
                    }
                    else if(Labs(now)==ans)
                        id=min(id,tot);
                    ll num=solve(-now);//二分找到与相反数最接近的数
                    if(ans>Labs(num+now)){
                        ans=Labs(num+now);
                        id=tot+vis[num];
                    }
                    else if(ans==Labs(num+now))
                        id=min(id,tot+vis[num]);
                }
                printf("%lld %d\n",ans,id);
            }
            return 0;
        }
        ```

??? note "[Sumsets](http://poj.org/problem?id=2549)"
    给出一个数集，从中选择四个元素，使得a+b+c=d，最小化d。

    ??? tip
        我们对a+b建立Hash_table,之后枚举c和d，寻找c-d且不由c和d构成的hash值是否存在。如果存在，那么d就可以用来更新答案。也可以使用二分查找。

    ??? note "参考代码"

        ```cpp
        #include<cstdio>
        #include<vector>
        #include<cmath>
        #include<algorithm>
        using namespace std;
        typedef long long ll;
        const int maxn = 2e6 + 5;
        const ll inf = -1e18;
        ll arr[maxn];
        struct node{
            ll x, y, dis;
            node(){}
            node(ll a, ll b, ll c){
                x = a;
                y = b;
                dis = c;
            }
        }temp[maxn];
        bool operator <(const node& a, const node& b){
            return a.dis < b.dis;
        }
        ll max(ll a, ll b){
            return a > b ? a : b;
        }
        int BinarySearch(node *A, int n, ll value){
            int l = 0, r = n - 1;
            while (r - l > 0){
                int m = l + (r  - l) / 2;
                if (A[m].dis == value)	return m;
                else if (A[m].dis < value) l = m + 1;
                else 	r = m;
            }
            return -1;
        }
        void solve(){
            int n;
            while (scanf("%d", &n), n){
                for (int i = 0; i < n; ++i){
                    scanf("%lld", &arr[i]);
                }
                sort(arr, arr + n);
                int cnt = 0;
                for (int i = 0; i < n; ++i){
                    for (int j = i + 1; j < n; ++j){
                        temp[cnt++] = node(arr[i], arr[j], arr[i] + arr[j]);
                    }
                }
                sort(temp, temp + cnt);
                ll ans = inf;
                for (int i = n - 1; i >= 0; --i){
                    for (int j = n - 1; j >= 0; --j){
                        if (i == j)	continue;
                        int pos = BinarySearch(temp, cnt, arr[i] - arr[j]);
                        if (pos == -1) continue;
                        if (temp[pos].x != arr[i] && temp[pos].x != arr[j] && temp[pos].y != arr[i] && temp[pos].y !=arr[j]){
                            ans = arr[i];
                            break;
                        }				
                    }
                    if (ans != inf) break;
                }
                if (ans == inf)		puts("no solution");
                else	printf("%lld\n", ans);
            }
        }
        int main(){
            solve();
            return 0;
        }
        ```

??? note "[Cubic Eight-Puzzle](http://poj.org/problem?id=3131)"
    这是一个立体的8数码问题，不过有点区别，有8个立体的正方体摆在3*3的区域内，留一个空格方便移动。移动的规则是：空格旁边的小正方体可以滚动到空格位置，小正方体原来的位置变成空格。每个小正方体6个面有3中颜色，对面颜色相同，分别是white，blue，red。初始状态每个小正方体的摆放方式都一样，从正面看上面是white，前面是red，右面是blue，空格位置给定。现在给定一个初始的空格位置以及一个终态的上表面的颜色分布，求是否能在30步内从初态到终态，能，输出最少步数，不能，输出-1，注意给定的终态只是上表面的颜色分布，其他面上的颜色不做要求。

    ??? tip
        bfs。这题状态数非常之多，时间空间都要求很高，如果单向bfs，时空复杂度都太高，因此选择双向bfs。首先要解决的是判重问题，这题不像二维平面的8数码，每个位置只有一个状态，因此可以用康托判重，这题每个位置的小方块有6个状态，加上空格，每个位置有7个状态，那么总的状态数就是7^9，显然还是太大了。再考虑到9个位置只能有一个位置是空格，可以把空格单独拿出来判重，这样空间复杂度就降到了6^8*9，勉强可以接受。
        
        这就需要用六进制数表示状态，然后判重也好判了，最后的问题就是模拟方块的滚动了，4个方向滚动，状态会发生怎样的变化都要在草稿纸上画清楚。
        
        ps：这题还要说明一点，因为初状态只有一种，末状态却有2^8种。所以从前往后搜和从后往前搜搜的深度得不一样才行，从前往后多搜点，才能保证双搜沿棱形展开。

    ??? note "参考代码"

        === "STL"

            ```cpp
            include<cstdio>
            #include<cstring>
            #include<queue>
            #include<iostream>
            #include<algorithm>
            using namespace std;            // 0WRB, 1WBR, 2RWB, 3RBW, 4BRW, 5BWR
            
            int base[]={1,6,36,216,1296,7776,46656,279936,1679616};
            int state[6][4]={{2,2,4,4},{5,5,3,3},{0,0,5,5},{4,4,1,1},{3,3,0,0},{1,1,2,2}};
            char hash1[1680000][9];
            char hash2[1680000][9];
            char b[10];
            struct node
            {
                int s[9];
                int dis;
                int space;
                int value;
            }st;
            queue<node> q;
            int dir[4][2]={{1,0},{-1,0},{0,1},{0,-1}};      //下，上，右，左，和前面相反
            
            inline bool isok(int &x,int &y)
            {
                return x>=0&&x<3&&y>=0&&y<3;
            }
            inline int cal(node &t)     //计算初始的hash值
            {
                int value=0;
                int top=0;
                for(int i=0;i<9;i++)
                {
                    if(i==t.space) continue;
                    value+=t.s[i]*base[top++];
                }
                return value;
            }
            int cal(node &t,node &f,int d)     //因为每次移动只会改变几个hash值，所以可以特判
            {
                if(d==2)
                {
                    t.value-=f.s[t.space]*base[f.space];
                    t.value+=t.s[f.space]*base[f.space];
                    return t.value;
                }
                else if(d==3)
                {
                    t.value-=f.s[t.space]*base[t.space];
                    t.value+=t.s[f.space]*base[t.space];
                    return t.value;
                }
                else if(d==0)
                {
                    for(int i=f.space;i<=f.space+2;i++)
                    {
                        t.value-=f.s[i+1]*base[i];
                        t.value+=t.s[i]*base[i];
                    }
                    return t.value;
                }
                else if(d==1)
                {
                    for(int i=t.space;i<=t.space+2;i++)
                    {
                        t.value-=f.s[i]*base[i];
                        t.value+=t.s[i+1]*base[i];
                    }
                    return t.value;
                }
            }
            bool bfs(node st)
            {
                node t1,t2,f;
                queue<node> Q;
                st.dis=0;
                Q.push(st);
                t2.dis=0;
                int k;
                int k1=1,kk1=0,k2=256,kk2=0;
                while(!Q.empty()&&!q.empty())
                {
                    while(k1--){
                    st=Q.front();Q.pop();
                    if(st.dis+1+t2.dis>30) {printf("-1\n");return false;}
                    for(int d=0;d<4;d++)
                    {
                        t1=st;
                        int sx=t1.space/3;
                        int sy=t1.space%3;
                        int nx=sx+dir[d][0];
                        int ny=sy+dir[d][1];
                        if(isok(nx,ny))
                        {
                            t1.space=nx*3+ny;
                            t1.s[sx*3+sy]=state[t1.s[nx*3+ny]][d];
                            t1.s[nx*3+ny]=6;
                            t1.dis=st.dis+1;
                            t1.value=cal(t1,st,d);
                            if(hash1[t1.value][t1.space]) continue;
                            if(hash2[t1.value][t1.space]) {printf("%d\n",t1.dis+t2.dis);return true;}
            
                            hash1[t1.value][t1.space]=t1.dis;
                            Q.push(t1);
                            kk1++;
                        }
                    }
                    }
                    k1=kk1;
                    kk1=0;
                    while(k2--){
                    f=q.front();
                    if(f.dis==9) break;
                    q.pop();
                    for(int d=0;d<4;d++)
                    {
                        t2=f;
                        int sx=t2.space/3;
                        int sy=t2.space%3;
                        int nx=sx+dir[d][0];
                        int ny=sy+dir[d][1];
                        t2.dis=f.dis+1;
                        if(isok(nx,ny))
                        {
                            t2.space=nx*3+ny;
                            t2.s[sx*3+sy]=state[t2.s[nx*3+ny]][d];
                            t2.s[nx*3+ny]=6;
                            t2.value=cal(t2,f,d);
                            if(hash2[t2.value][t2.space]) continue;
                            if(hash1[t2.value][t2.space])
                            {
                                printf("%d\n",t2.dis+st.dis+1);
                                return true;
                            }
                            hash2[t2.value][t2.space]=t2.dis;
                            q.push(t2);
                            kk2++;
                        }
                    }
                    }
                    t1.dis=f.dis+1;
                    k2=kk2;
                    kk2=0;
                }
                printf("-1\n");
            }
            void dfs(node &end,int k)
            {
                if(k==9)
                {
                    end.dis=0;
                    end.value=cal(end);
                    q.push(end);
                    hash2[end.value][end.space]=1;
                    return;
                }
                if(b[k]=='W')
                {
                    end.s[k]=0;
                    dfs(end,k+1);
                    end.s[k]=1;
                    dfs(end,k+1);
                }
                else if(b[k]=='R')
                {
                    end.s[k]=2;
                    dfs(end,k+1);
                    end.s[k]=3;
                    dfs(end,k+1);
                }
                else if(b[k]=='B')
                {
                    end.s[k]=4;
                    dfs(end,k+1);
                    end.s[k]=5;
                    dfs(end,k+1);
                }
                else
                {
                    end.s[k]=6;
                    dfs(end,k+1);
                }
            }
            int main()
            {
                int x,y;
                char a;
                node end;
                while(scanf("%d%d",&y,&x)!=EOF)
                {
                    if(!x&&!y) break;
                    while(!q.empty()) q.pop();
                    memset(hash1,0,sizeof(hash1));
                    memset(hash2,0,sizeof(hash2));
            
                    x--;y--;
                    for(int i=0;i<9;i++)
                    {
                        if(x==i/3&&y==i%3) {st.s[i]=6;st.space=i;}
                        else st.s[i]=0;
                    }
                    for(int i=0;i<9;i++)
                    {
                        scanf(" %c",&a);
                        if(a=='E') end.space=i;
                        b[i]=a;
                    }
                    dfs(end,0);     //得到所有的终点状态，全部加入队列。
            
                    int k;          //一开始就是答案
                    st.value=cal(st);
                    hash1[st.value][st.space]=-1;
                    if(hash2[st.value][st.space]) {printf("0\n");continue;}
                    bfs(st);
                }
                return 0;
            }
            ```

        === "静态数组"

            ```cpp
            #include<cstdio>
            #include<cstring>
            #include<iostream>
            #include<algorithm>
            using namespace std;            // 0WRB, 1WBR, 2RWB, 3RBW, 4BRW, 5BWR
            
            int base[]={1,6,36,216,1296,7776,46656,279936,1679616};
            int state[6][4]={{2,2,4,4},{5,5,3,3},{0,0,5,5},{4,4,1,1},{3,3,0,0},{1,1,2,2}};
            char hash1[1680000][9];
            char hash2[1680000][9];
            char b[10];
            struct node
            {
                int s[9];
                int dis;
                int space;
                int value;
            }q1[300000],q2[100005],st;
            int r2;
            int dir[4][2]={{1,0},{-1,0},{0,1},{0,-1}};      //下，上，右，左，和前面相反
            
            inline bool isok(int &x,int &y)
            {
                return x>=0&&x<3&&y>=0&&y<3;
            }
            inline int cal(node &t)     //计算初始的hash值
            {
                int value=0;
                int top=0;
                for(int i=0;i<9;i++)
                {
                    if(i==t.space) continue;
                    value+=t.s[i]*base[top++];
                }
                return value;
            }
            int cal(node &t,node &f,int d)     //因为每次移动只会改变几个hash值，所以可以特判
            {
                if(d==2)
                {
                    t.value-=f.s[t.space]*base[f.space];
                    t.value+=t.s[f.space]*base[f.space];
                    return t.value;
                }
                else if(d==3)
                {
                    t.value-=f.s[t.space]*base[t.space];
                    t.value+=t.s[f.space]*base[t.space];
                    return t.value;
                }
                else if(d==0)
                {
                    for(int i=f.space;i<=f.space+2;i++)
                    {
                        t.value-=f.s[i+1]*base[i];
                        t.value+=t.s[i]*base[i];
                    }
                    return t.value;
                }
                else if(d==1)
                {
                    for(int i=t.space;i<=t.space+2;i++)
                    {
                        t.value-=f.s[i]*base[i];
                        t.value+=t.s[i+1]*base[i];
                    }
                    return t.value;
                }
            }
            bool bfs(node st)
            {
                node t1,t2,f;
                int r1=0,f1=0,f2=0;
                st.dis=0;
                q1[r1++]=st;
                t2.dis=0;
                int k;
                int k1=1,kk1=0,k2=256,kk2=0;
                while(f1<r1&&f2<r2)
                {
                    while(k1--){
                    st=q1[f1];
                    if(st.dis+1+t2.dis>30) {printf("-1\n");return false;}
                    for(int d=0;d<4;d++)
                    {
                        t1=st;
                        int sx=t1.space/3;
                        int sy=t1.space%3;
                        int nx=sx+dir[d][0];
                        int ny=sy+dir[d][1];
                        if(isok(nx,ny))
                        {
                            t1.space=nx*3+ny;
                            t1.s[sx*3+sy]=state[t1.s[nx*3+ny]][d];
                            t1.s[nx*3+ny]=6;
                            t1.dis=st.dis+1;
                            t1.value=cal(t1,st,d);
                            if(hash1[t1.value][t1.space]) continue;
                            if(hash2[t1.value][t1.space]) {printf("%d\n",t1.dis+t2.dis);return true;}
            
                            hash1[t1.value][t1.space]=t1.dis;
                            q1[r1++]=t1;
                            kk1++;
                        }
                    }
                    f1++;
                    }
                    k1=kk1;
                    kk1=0;
                    while(k2--){
                    f=q2[f2];
                    if(f.dis==8) break;         //超过这个，反向就不搜了，
                    for(int d=0;d<4;d++)
                    {
                        t2=f;
                        int sx=t2.space/3;
                        int sy=t2.space%3;
                        int nx=sx+dir[d][0];
                        int ny=sy+dir[d][1];
                        t2.dis=f.dis+1;
                        if(isok(nx,ny))
                        {
                            t2.space=nx*3+ny;
                            t2.s[sx*3+sy]=state[t2.s[nx*3+ny]][d];
                            t2.s[nx*3+ny]=6;
                            t2.value=cal(t2,f,d);
                            if(hash2[t2.value][t2.space]) continue;
                            if(hash1[t2.value][t2.space])
                            {
                                printf("%d\n",t2.dis+st.dis+1);
                                return true;
                            }
                            hash2[t2.value][t2.space]=t2.dis;
                            q2[r2++]=t2;
                            kk2++;
                        }
                    }
                        f2++;
                    }
            
                    t1.dis=f.dis+1;
                    k2=kk2;
                    kk2=0;
                }
                printf("-1\n");
            }
            void dfs(node &end,int k)
            {
                if(k==9)
                {
                    end.dis=0;
                    end.value=cal(end);
                    q2[r2++]=end;
                    hash2[end.value][end.space]=1;
                    return;
                }
                if(b[k]=='W')
                {
                    end.s[k]=0;
                    dfs(end,k+1);
                    end.s[k]=1;
                    dfs(end,k+1);
                }
                else if(b[k]=='R')
                {
                    end.s[k]=2;
                    dfs(end,k+1);
                    end.s[k]=3;
                    dfs(end,k+1);
                }
                else if(b[k]=='B')
                {
                    end.s[k]=4;
                    dfs(end,k+1);
                    end.s[k]=5;
                    dfs(end,k+1);
                }
                else
                {
                    end.s[k]=6;
                    dfs(end,k+1);
                }
            }
            int main()
            {
                int x,y;
                char a;
                node end;
                while(scanf("%d%d",&y,&x)!=EOF)
                {
                    if(!x&&!y) break;
                    memset(hash1,0,sizeof(hash1));
                    memset(hash2,0,sizeof(hash2));
                    r2=0;
                    x--;y--;
                    for(int i=0;i<9;i++)
                    {
                        if(x==i/3&&y==i%3) {st.s[i]=6;st.space=i;}
                        else st.s[i]=0;
                    }
                    for(int i=0;i<9;i++)
                    {
                        scanf(" %c",&a);
                        if(a=='E') end.space=i;
                        b[i]=a;
                    }
                    dfs(end,0);     //得到所有的终点状态，全部加入队列。
            
                    int k;          //一开始就是答案
                    st.value=cal(st);
                    hash1[st.value][st.space]=-1;
                    if(hash2[st.value][st.space]) {printf("0\n");continue;}
                    bfs(st);
                }
                return 0;
            }
            ```

??? note "[万圣节](https://vjudge.net/problem/UVA-1601)"
    在这个问题中，你被要求编写一个程序，在给定的房屋平面图上，找出移动所有鬼魂的最小步骤。将所有鬼魂移到他们应该在的位置上的最小步骤。
        
    一个楼层由一个正方形单元的矩阵组成。一个单元要么是鬼魂不能进入的墙单元，要么是鬼魂不能进入的走廊单元。或走廊单元，它们可以进入。
        
    在每一步，你可以同时移动任何数量的幽灵。每一个鬼魂都可以留在当前的单元，或者移动到其4个邻近的走廊单元中的一个（即紧挨着的左、右、上或下），如果鬼魂是这样的话。上或下），如果这些鬼魂满足以下条件。

    1. 在步骤结束时，没有一个以上的鬼魂占据一个位置。
    2. 在该步骤中没有一对鬼魂互相交换位置。

    ??? tip
        基本思路是BFS：

        1. 题目中已经说了，每相连的2X2格子中必有一个‘#’,也就是，每个点周围最多也就三个方向可以走。因此，可以把所有空格都提出来，形成一个图，直接遍历每条边，而不是每次判断4个方向是否可以走
        2. 关于结点判重，最初的想法是想用一个六维数组，后来参考了其它，发现其实可以用一个三维数字代替，每个点可以用数字代替，因为题目中整个图最多为16X16，所以数字最大为16*16 = 256，这样做的另一个好处是，移动字母也方便了，可以用数字代替点，比如从（0， 0）移动到（0，1）可以考做是从 0 移动到 1
        3. N的数量不一，移动时要不要判断是N是多少？这是我刚开始的想法，个人采用递归的方法，先放一个点，再放下一个点，观察这是不是最后一个要移动的点，相对而言，比对每个N值进行单独处理要简单点。

    ??? note "参考代码"

        ```cpp
        #include <bits/stdc++.h>
 
        using namespace std;
        
        const int MAXN = 16 + 5;
        char plan[MAXN][MAXN];
        int W, H, N;
        vector<int> link[MAXN*MAXN];
        bool vis[MAXN*MAXN][MAXN*MAXN][MAXN*MAXN];
        int dir[4][2] = {0,1, 0,-1, 1,0, -1,0};
        
        struct State{
            int ghostPos[5], step;
        };
        
        State finalState, firstState;
        
        // more than one ghost occupy a same position
        // p is the prev state, whether two ghots in s exchange with their position
        bool isCollisionOrExChange(State& s, State& p) {
            for(int i=0; i<N; ++i) {
                for(int j=0; j<N; ++j) {
                    if(i!=j && s.ghostPos[i] == s.ghostPos[j]) {
                        return true;
                    }
                    if(i!=j && s.ghostPos[i] == p.ghostPos[j] && s.ghostPos[j] == p.ghostPos[i]) {
                        return true;
                    }
                }
            }
            return false;
        }
        
        void Read() {
            for(int i=0; i<H; ++i) {
                cin.get();
                for(int j=0; j<W; ++j) {
                    plan[i][j] = cin.get();
                }
            }
        }
        
        int GetVisValue(int x, int y) {
            return x * W + y;
        }
        
        void SetVis(State& t, bool flag) {
            int i, a[3]; // the positon of each ghost
            for(i=0; i<N; ++i) {
                a[i] = t.ghostPos[i];
            }
            // the number of ghosts may be less than 3
            for(; i<3; ++i) {
                a[i] = 0;
            }
            vis[a[0]][a[1]][a[2]] = flag;
        }
        
        bool IsVis(State& t) {
            int i, a[3]; // the positon of each ghost
            for(i=0; i<N; ++i) {
                a[i] = t.ghostPos[i];
            }
            // the number of ghosts may be less than 3
            for(; i<3; ++i) {
                a[i] = 0;
            }
            return vis[a[0]][a[1]][a[2]];
        }
        
        bool IsFinal(State& t) {
            for(int i=0; i<N; ++i) {
                if(finalState.ghostPos[i] != t.ghostPos[i]) {
                    return false;
                }
            }
            return true;
        }
        
        // the ghost numbered g move
        void NextState(State& s, int g, queue<State>& q, State& former) {
            for(size_t i=0; i<link[former.ghostPos[g]].size(); ++ i) {
                s.ghostPos[g] = link[former.ghostPos[g]][i];
                if(g == N-1 && !IsVis(s) && !isCollisionOrExChange(s, former)) {
                    SetVis(s, true);
                    q.push(s);
                } else if (g < N-1) {
                    NextState(s, g+1, q, former);
                }
            }
        }
        
        // output the point of all ghosts
        void Test(State& t) {
            for(int i=0; i<N; ++i) {
                cout << t.ghostPos[i] / W << " " << t.ghostPos[i] % W << endl;
            }
            cout << t.step << endl;
            cout << endl;
        }
        
        int Bfs() {
            memset(vis, false, sizeof(vis));
            queue<State> q;
            firstState.step = 0;
            q.push(firstState);
            SetVis(firstState, true);
            while(!q.empty()) {
                State t = q.front();
                q.pop();
                // Test(t);
                if(IsFinal(t)) {
                    return t.step;
                }
                State newS;
                newS.step = t.step + 1;
                NextState(newS, 0, q, t);
            }
            return -1;
        }
        
        // Make a Graph
        void MakeLink() {
            for(int i=0; i<H; ++ i) {
                for(int j=0; j<W; ++ j) {
                    if(plan[i][j] != '#') {
                        // the position means unmoving
                        link[GetVisValue(i, j)].clear();
                        link[GetVisValue(i, j)].push_back(GetVisValue(i, j));
                        for(int k=0; k<4; ++k) { // four directions
                            int x = i + dir[k][0];
                            int y = j + dir[k][1];
                            if(x>=0 && x<H && y>=0 && y<W && plan[x][y]!='#') {
                                link[GetVisValue(i, j)].push_back(GetVisValue(x, y));
                            }
                        }
                    }
                    if (isupper(plan[i][j])) {
                        finalState.ghostPos[plan[i][j]-'A'] = GetVisValue(i, j);
                    } else if (islower(plan[i][j])) {
                        firstState.ghostPos[plan[i][j]-'a'] = GetVisValue(i, j);
                    }
                }
            }
        }
        
        void Work() {
            Read();
            MakeLink();
            cout << Bfs() << endl;
        }
        
        int main() {
            ios::sync_with_stdio(false);
            cin.tie(0);
            while(cin >> W >> H >> N) {
                if(!W && !H && !N) {
                    break;
                }
                Work();
            }
            return 0;
        }
        ```

## 外部链接

-   [What is meet in the middle algorithm w.r.t. competitive programming? - Quora](https://www.quora.com/What-is-meet-in-the-middle-algorithm-w-r-t-competitive-programming)
-   [Meet in the Middle Algorithm - YouTube](https://www.youtube.com/watch?v=57SUNQL4JFA)
