## 定义

二分图，又称二部图，英文名叫 Bipartite graph。

二分图是什么？节点由两个集合组成，且两个集合内部没有边的图。

换言之，存在一种方案，将节点划分成满足以上性质的两个集合。

![](./images/bi-graph.svg)

## 性质

-   如果两个集合中的点分别染成黑色和白色，可以发现二分图中的每一条边都一定是连接一个黑色点和一个白色点。

-   ??? question "二分图不存在长度为奇数的环"
        因为每一条边都是从一个集合走到另一个集合，只有走偶数次才可能回到同一个集合。

## 判定

如何判定一个图是不是二分图呢？

换言之，我们需要知道是否可以将图中的顶点分成两个满足条件的集合。

显然，直接枚举答案集合的话实在是太慢了，我们需要更高效的方法。

考虑上文提到的性质，我们可以使用 [DFS（图论）](./dfs.md) 或者 [BFS](./bfs.md) 来遍历这张图。如果发现了奇环，那么就不是二分图，否则是。

=== "DFS"

    ```cpp
    /*描述:给定一个无向图，判断是否是二分图*/
    /*思想:通过dfs染色的方法，从任意未遍历过的顶点出发进行染色,如果有与当前顶点染色相同的，则不是二分图
    否则继续搜索与当前顶点相连的未染色的节点，并且染上相反的颜色，最后遍历完所有的顶点，相邻顶点没有同色，则为二分图。
    时间复杂度O(n^2),空间复杂度O(n^2),若采用邻接表作为存储结构，则时间复杂度为O(n+m)
    */
    #include<bits/stdc++.h>
    using namespace std;
    const int maxn=1005;
    int edge[maxn][maxn];//邻接矩阵存储
    int color[maxn];//标记顶点颜色
    int n,m;
    bool dfs(int u,int c)
    {
        color[u]=c;//对u点进行染色
        for(int i=1;i<=n;i++)//遍历与u点相连的点
        {
            if(edge[u][i]==1)//如果i点与u点相连
            {
                if(color[i]==c) return false;//i点的染色重复，则不是二分图
                if(!color[i]&&!dfs(i,-c)) return false;//该点未染色，染上相反的颜色.dfs继续搜索
            }
        }
        return true;//所有点染色完成之后，并且相邻顶点没有同色，则为二分图
    }
    int main()
    {
        while(scanf("%d%d",&n,&m)!=EOF)
        {
            memset(edge,0,sizeof(edge));
            memset(color,0,sizeof(color));
            for(int i=0;i<m;i++)
            {
                int u,v;
                scanf("%d%d",&u,&v);
                edge[u][v]=1;//无向图，因此uv和vu都需要
                edge[v][u]=1;
            }
            bool flag=false; //默认为二分图
            for(int i=1;i<=n;i++)
            {
                if(!color[i])
                {
                    if(!dfs(i,1))
                    {
                        printf("No\n");
                        flag=true;
                    }
                }
            }
            if(!flag) printf("Yes\n");
        }
        return 0;
    }
    ```

=== "BFS"

    ```cpp
    #include<bits/stdc++.h>
    using namespace std;
    const int maxn=1005;
    int color[maxn];
    vector<int>edge[maxn];//存储采用vector，相当于邻接表
    int n,m;
    bool bfs(int x,int c)
    {
        queue<int>q;
        q.push(x);//从x号节点开始
        color[x]=c;
        while(!q.empty())
        {
            int now=q.front();
            q.pop();
            for(int i=0;i<edge[now].size();i++)
            {
                int v=edge[now][i];
                if(color[v]==0)//若v点未进行染色
                {
                    color[v]=-color[now];//将v点染色，并且不与当前节点now节点相同
                    q.push(v);//将v点放进队列继续访问
                }
                if(color[v]==color[now]) return false;//与当前节点颜色相同，该图不是二分图
            }
        }
        return true;
    }
    int main()
    {
        while(~scanf("%d%d",&n,&m))
        {
            memset(color,0,sizeof(color));
            for(int i=1;i<=m;i++)
            {
                int u,v;
                scanf("%d%d",&u,&v);
                edge[u].push_back(v);
                edge[v].push_back(u);
            }
            bool flag=true;
            for(int i=1;i<=n;i++)
            {
                if(color[i]==0&&!bfs(i,1))
                {
                    flag=false;
                    break;
                }
            }
            if(flag) printf("The Graph is two partite graph!\n");
            else     printf("The Graph is not two partite graph!\n");
        }
    }
    ```

=== "DSU"

    ```cpp
    #include<bits/stdc++.h>
    using namespace std;
    const int maxn=1005;
    int father[maxn];
    int findset(int x)//并查集路径压缩
    {
        if(x!=father[x]) father[x]=findset(father[x]);
        return father[x];
    }
    int main()
    {
        int n,m;
        while(~scanf("%d%d",&n,&m))
        {
            for(int i=1;i<=2*n;i++) father[i]=i;
            bool flag=true;
            for(int i=1;i<=m;i++)
            {
                int u,v;
                scanf("%d%d",&u,&v);
                int fa=findset(u);//查找根节点
                int fb=findset(v);
                int x=findset(u+n);//查找不同集合的根节点
                int y=findset(v+n);
                if(fa==fb) flag=false;
                else
                {
                    father[x]=fb;
                    father[y]=fa;
                }
            }
            if(flag) printf("YES\n");
            else     printf("NO\n");
        }
        return 0;
    }
    ```

## 应用

### 二分图最大匹配

详见 [二分图最大匹配](./graph-matching/bigraph-match.md) 页面。

### 二分图最大权匹配

详见 [二分图最大权匹配](./graph-matching/bigraph-weight-match.md) 页面。

### 一般图最大匹配

详见 [一般图最大匹配](./graph-matching/general-match.md) 页面。

### 一般图最大权匹配

详见 [一般图最大权匹配](./graph-matching/general-weight-match.md) 页面。
