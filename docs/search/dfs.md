## 引入

DFS 为图论中的概念，详见 [DFS（图论）](../graph/dfs.md) 页面。在 **搜索算法** 中，该词常常指利用递归函数方便地实现暴力枚举的算法，与图论中的 DFS 算法有一定相似之处，但并不完全相同。

## DFS代码框架
DFS的代码看起来比较简单，但是逻辑上难以理解，不容易编码。

读者在大量编码的基础上，再回头体会这个框架的作用。

```cpp
ans;                  //答案，用全局变量表示
void dfs(层数，其他参数){
    if (出局判断){    //到达最底层，或者满足条件退出 
        更新答案;     //答案一般用全局变量表示
        return;       //返回到上一层
    }
    (剪枝)            //在进一步DFS之前剪枝
    for (枚举下一层可能的情况)    //对每一个情况继续DFS 
        if (used[i] == 0) {       //如果状态i没有用过，就可以进入下一层
            used[i] = 1;   //标记状态i,表示已经用过，在更底层不能再使用
            dfs(层数+1，其他参数);    //下一层 
            used[i] = 0;   //恢复状态，回溯时，不影响上一层对这个状态的使用
            }
    return;          //返回到上一层
}
```

![](./images/search-1.jpg)

下面给出DFS遍历图二叉树的代码。分别给出了静态版和指针版二叉树的代码。

它们输出了图1二叉树的各种DFS操作，有时间戳、DFS序、树深度、子树节点数，另外还给出了二叉树的中序输出、先序输出、后序输出。

DFS访问节点，经常用到以下操作：

- 节点第一次被访问的时间戳。用dfn[i]表示节点i第一次被访问的时间戳，函数dfn_order()打印出了时间戳：

dfn[E]=1; dfn[B]=2; dfn[A]=3; dfn[D]=4; dfn[C]=5; dfn[G]=6; dfn[F]=7; dfn[I]=8; dfn[H]=9。

时间戳就是先序输出。

- DFS序。在递归时，每个节点会来回访问2次，即第1次访问和第2次回溯。函数visit_order()打印出了DFS序：

{E B A A D C C D B G F F I H H I G E}

这个序列有一个重要特征：每个节点出现2次，被这2次包围起来的，是以它为父节点的一棵子树。例如序列中的{B A A D C C D B}，就是B为父节点的一棵子树，又例如{I H H I}，是以I为父节点的一棵子树。这个特征是递归操作产生的。

- 树的深度。从根节点往子树DFS，每个节点第一次被访问时，深度加1，从这个节点回溯时，深度减1。用deep[i]表示节点i的深度，函数deep_node()打印出了深度：
deep[E]=1; deep[B]=2; deep[A]=3; deep[D]=3; deep[C]=4; deep[G]=2; deep[F]=3; deep[I]=3; deep[H]=4。

- 子树节点总数。用num[i]表示以i为父亲的子树上的节点总数，例如，以B为父节点的子树，共有4个节点{A B C D}。只需要简单地DFS一次就能完成，每个节点的数量等于它的2个子树的数量相加，再加1，即加它自己。函数num_node()做了计算并打印出了以每个节点为父亲的子树上的节点数量。
另外还有树的重心：在一棵中，找到一个节点，把树变为以该点为根的有根树，并且最大子树的结点数最小。本文没有给出代码。

竞赛中一般用静态版二叉树写代码。作为对照，后面也给出指针版二叉树的代码。

???+note "参考代码"
    === "静态版二叉树"

        ```cpp
        #include <bits/stdc++.h>
        using namespace std;
        const int maxn = 100005;
        struct Node{
            char value;
            int lchild, rchild;    
        }node[maxn];
        int index = 0;                 //记录节点
        int newNode(char val){         //新建节点
            node[index].value = val;
            node[index].lchild = -1;   //-1表示空
            node[index].rchild = -1;
            return index ++;
        }
        void insert(int &father, int child, int l_r){   //插入孩子
            if(l_r == 0)              //左孩子
                node[father].lchild = child;
            else                      //右孩子
                node[father].rchild = child;	
        }
        int dfn[maxn] = {0};              //dfn[i]是节点i的时间戳
        int dfn_timer = 0;
        void dfn_order (int father){      
            if(father != -1){
                dfn[father] = ++dfn_timer; 
                printf("dfn[%c]=%d; ", node[father].value, dfn[father]);    
                                            //打印访问节点的时间戳
                dfn_order (node[father].lchild);
                dfn_order (node[father].rchild);        
            }
        }
        int visit_timer = 0;     
        void visit_order (int father){        //打印DFS序
            if(father != -1){
                printf("visit[%c]=%d; ", node[father].value, ++visit_timer);  
                                            //打印DFS序：第1次访问节点 
                visit_order (node[father].lchild);
                visit_order (node[father].rchild);
                printf("visit[%c]=%d; ", node[father].value, ++visit_timer);  
                                            //打印DFS序：第2次回溯
            }
        }
        int deep[maxn] = {0};                 //deep[i]是节点i的深度
        int deep_timer = 0;         
        void deep_node (int father){      
            if(father != -1){
                deep[father] = ++deep_timer; 
                printf("deep[%c]=%d; ",node[father].value,deep[father]);    
                                            //打印树的深度，第一次访问时，深度+1 
                deep_node (node[father].lchild);
                deep_node (node[father].rchild);
                deep_timer--;                 //回溯时，深度-1
            }
        }
        int num[maxn] = {0};        //num[i]是以i为父亲的子树上的节点总数
        int num_node (int father){          
            if(father == -1)  return 0;
            else{
                num[father] = num_node (node[father].lchild) + 
                            num_node (node[father].rchild) + 1; 
                printf("num[%c]=%d; ", node[father].value, num[father]); //打印数量
                return num[father];
            }
        }
        void preorder (int father){                 //求先序序列
            if(father != -1){
                cout << node[father].value <<" ";   //先序输出
                preorder (node[father].lchild);
                preorder (node[father].rchild);
            }
        }
        void inorder (int father){                   //求中序序列
            if(father != -1){
                inorder (node[father].lchild);;
                cout << node[father].value <<" ";    //中序输出
                inorder (node[father].rchild);
            }
        }
        void postorder (int father){                 //求后序序列
            if(father != -1){
                postorder (node[father].lchild);;
                postorder (node[father].rchild);
                cout << node[father].value <<" ";    //后序输出
            }
        }
        int buildtree(){                             //建一棵树
            int A = newNode('A');int B = newNode('B');int C = newNode('C');
            int D = newNode('D');int E = newNode('E');int F = newNode('F');
            int G = newNode('G');int H = newNode('H');int I = newNode('I');
            insert(E,B,0);  insert(E,G,1);         //E的左孩子是B，右孩子是G
            insert(B,A,0);  insert(B,D,1);
            insert(G,F,0);  insert(G,I,1);
            insert(D,C,0);  insert(I,H,0);
            int root = E;
            return root;
        }
        int main(){
            int root = buildtree();
            cout <<"dfn order: ";     dfn_order(root); cout <<endl;     //打印时间戳
            cout <<"visit order: "; visit_order(root); cout <<endl;     //打印DFS序
            cout <<"deep order: ";    deep_node(root); cout <<endl;     //打印节点深度
            cout <<"num of tree: ";    num_node(root); cout <<endl;  //打印子树上的节点数
            cout <<"in order:   ";      inorder(root); cout << endl;    //打印中序序列
            cout <<"pre order:  ";     preorder(root); cout << endl;    //打印先序序列
            cout <<"post order: ";    postorder(root); cout << endl;    //打印后序序列
            return 0;
        }
        /*输出是：
        dfn order: dfn[E]=1; dfn[B]=2; dfn[A]=3; dfn[D]=4; dfn[C]=5; dfn[G]=6; dfn[F]=7; dfn[I]=8; dfn[H]=9;
        visit order: visit[E]=1; visit[B]=2; visit[A]=3; visit[A]=4; visit[D]=5; visit[C]=6; visit[C]=7; visit[D]=8; visit[B]=9; visit[G]=10; visit[F]=11; visit[F]=12; visit[I]=13; visit[H]=14; visit[H]=15; visit[I]=16; visit[G]=17; visit[E]=18;
        deep order: deep[E]=1; deep[B]=2; deep[A]=3; deep[D]=3; deep[C]=4; deep[G]=2; deep[F]=3; deep[I]=3; deep[H]=4;
        num of tree: num[A]=1; num[C]=1; num[D]=2; num[B]=4; num[F]=1; num[H]=1; num[I]=2; num[G]=4; num[E]=9;
        in order:   A B C D E F G H I
        pre order:  E B A D C G F I H
        post order: A C D B F H I G E
        */
        ```
    === "指针版二叉树"

        ```cpp
        #include "bits/stdc++.h"
        using namespace std;

        struct node{
            char value;
            node *l, *r;
            node(char value = '#', node *l = NULL, node r = NULL):value(value), l(l), r(r){}
        };

        void preorder (node root){          //求先序序列
            if(root != NULL){
                cout << root->value <<" ";   //先序输出
                preorder (root ->l);
                preorder (root ->r);
            }
        }
        void inorder (node root){           //求中序序列
            if(root != NULL){
                inorder (root ->l);
                cout << root->value <<" ";   //中序输出
                inorder (root ->r);
            }
        }
        void postorder (node root){          //求后序序列
            if(root != NULL){
                postorder (root ->l);
                postorder (root ->r);
                cout << root->value <<" ";    //后序输出
            }
        }
        void remove_tree(node root){         //释放空间
            if(root == NULL) return;
            remove_tree(root->l);
            remove_tree(root->r);
            delete root;
        }
        int main(){
            node  A, B,C,D,E,F,G,H,I;
            A = new node('A'); B = new node('B'); C = new node('C');
            D = new node('D'); E = new node('E'); F = new node('F');
            G = new node('G'); H = new node('H'); I = new node('I');
            E->l = B; E->r = G;      B->l = A; B->r = D;
            G->l = F; G->r = I;      D->l = C;       I->l = H;
            cout <<"in order:   ";    inorder(E); cout << endl;    //打印中序序列
            cout <<"pre order:  ";   preorder(E); cout << endl;    //打印先序序列
            cout <<"post order: ";  postorder(E); cout << endl;    //打印后序序列
            remove_tree(E); 
            return 0;
        }
        ```

## 例题

考虑这个例子：

???+note "例题"
    把正整数 $n$ 分解为 $3$ 个不同的正整数，如 $6=1+2+3$，排在后面的数必须大于等于前面的数，输出所有方案。

对于这个问题，如果不知道搜索，应该怎么办呢？

当然是三重循环，参考代码如下：

???+note "实现"
    === "C++"
    
        ```cpp
        for (int i = 1; i <= n; ++i)
          for (int j = i; j <= n; ++j)
            for (int k = j; k <= n; ++k)
              if (i + j + k == n) printf("%d = %d + %d + %d\n", n, i, j, k);
        ```
    
    === "Python"
    
        ```python
        for i in range(1, n + 1):
            for j in range(i, n + 1):
                for k in range(j, n + 1):
                    if i + j + k == n:
                        print("%d = %d + %d + %d" % (n, i, j, k))
        ```

那如果是分解成四个整数呢？再加一重循环？

那分解成小于等于 $m$ 个整数呢？

这时候就需要用到递归搜索了。

该类搜索算法的特点在于，将要搜索的目标分成若干「层」，每层基于前几层的状态进行决策，直到达到目标状态。

考虑上述问题，即将正整数 $n$ 分解成小于等于 $m$ 个正整数之和，且排在后面的数必须大于等于前面的数，并输出所有方案。

设一组方案将正整数 $n$ 分解成 $k$ 个正整数 $a_1, a_2, \ldots, a_k$ 的和。

我们将问题分层，第 $i$ 层决定 $a_i$。则为了进行第 $i$ 层决策，我们需要记录三个状态变量：$n-\sum_{j=1}^i{a_j}$，表示后面所有正整数的和；以及 $a_{i-1}$，表示前一层的正整数，以确保正整数递增；以及 $i$，确保我们最多输出 $m$ 个正整数。

为了记录方案，我们用 arr 数组，第 i 项表示 $a_i$. 注意到 arr 实际上是一个长度为 $i$ 的栈。

代码如下：

???+note "实现"
    === "C++"
    
        ```cpp
        int m, arr[103];  // arr 用于记录方案
    
        void dfs(int n, int i, int a) {
          if (n == 0) {
            for (int j = 1; j <= i - 1; ++j) printf("%d ", arr[j]);
            printf("\n");
          }
          if (i <= m) {
            for (int j = a; j <= n; ++j) {
              arr[i] = j;
              dfs(n - j, i + 1, j);  // 请仔细思考该行含义。
            }
          }
        }
    
        // 主函数
        scanf("%d%d", &n, &m);
        dfs(n, 1, 1);
        ```
    
    === "Python"
    
        ```python
        arr = [0] * 103  # arr 用于记录方案
    
        def dfs(n, i, a):
            if n == 0:
                print(arr[1:i])
            if i <= m:
                for j in range(a, n + 1):
                    arr[i] = j
                    dfs(n - j, i + 1, j)  # 请仔细思考该行含义。
    
        # 主函数
        n, m = map(int, input().split())
        dfs(n, 1, 1)
        ```

???+note "[Luogu P1706 全排列问题](https://www.luogu.com.cn/problem/P1706)"
    === "C++ 代码："
        ```cpp
        --8<-- "docs/search/code/dfs/dfs_1.cpp"
        ```

## 习题
- [力扣的DFS题](https://leetcode-cn.com/tag/depth-first-search/)
- [Red and Black](http://poj.org/problem?id=1979)
- [Ball](https://vjudge.net/problem/Aizu-0033#author=baobaobear)
- [Curling 2.0](http://poj.org/problem?id=3009)