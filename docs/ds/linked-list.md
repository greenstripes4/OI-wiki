本页面将简要介绍链表。

## 引入

链表是一种用于存储数据的数据结构，通过如链条一般的指针来连接元素。它的特点是插入与删除数据十分方便，但寻找与读取数据的表现欠佳。

## 与数组的区别

链表和数组都可用于存储数据。与链表不同，数组将所有元素按次序依次存储。不同的存储结构令它们有了不同的优势：

链表因其链状的结构，能方便地删除、插入数据，操作次数是 $O(1)$。但也因为这样，寻找、读取数据的效率不如数组高，在随机访问数据中的操作次数是 $O(n)$。

数组可以方便地寻找并读取数据，在随机访问中操作次数是 $O(1)$。但删除、插入的操作次数是 $O(n)$ 次。

## 构建链表

???+ tip
    构建链表时，使用指针的部分比较抽象，光靠文字描述和代码可能难以理解，建议配合作图来理解。

### 单向链表

单向链表中包含数据域和指针域，其中数据域用于存放数据，指针域用来连接当前结点和下一节点。

![](images/list.svg)

???+ note "实现"
    === "C++"
        ```c++
        struct Node {
          int value;
          Node *next;
        };
        ```
    
    === "Python"
        ```python
        class Node:
            def __init__(self, value = None, next = None): 
                self.value = value
                self.next = next
        ```

### 双向链表

双向链表中同样有数据域和指针域。不同之处在于，指针域有左右（或上一个、下一个）之分，用来连接上一个结点、当前结点、下一个结点。

![](images/double-list.svg)

???+ note "实现"
    === "C++"
        ```c++
        struct Node {
          int value;
          Node *left;
          Node *right;
        };
        ```
    
    === "Python"
        ```python
        class Node:
            def __init__(self, value = None, left = None, right = None): 
                self.value = value
                self.left = left
                self.right = right
        ```

## 向链表中插入（写入）数据

### 单向链表

流程大致如下：

1.  初始化待插入的数据 `node`；
2.  将 `node` 的 `next` 指针指向 `p` 的下一个结点；
3.  将 `p` 的 `next` 指针指向 `node`。

具体过程可参考下图：

1.  ![](./images/list-insert-1.svg)
2.  ![](./images/list-insert-2.svg)
3.  ![](./images/list-insert-3.svg)

代码实现如下：

???+ note "实现"
    === "C++"
        ```c++
        void insertNode(int i, Node *p) {
          Node *node = new Node;
          node->value = i;
          node->next = p->next;
          p->next = node;
        }
        ```
    
    === "Python"
        ```python
        def insertNode(i, p):
            node = Node()
            node.value = i
            node.next = p.next
            p.next = node
        ```

### 单向循环链表

将链表的头尾连接起来，链表就变成了循环链表。由于链表首尾相连，在插入数据时需要判断原链表是否为空：为空则自身循环，不为空则正常插入数据。

大致流程如下：

1.  初始化待插入的数据 `node`；
2.  判断给定链表 `p` 是否为空；
3.  若为空，则将 `node` 的 `next` 指针和 `p` 都指向自己；
4.  否则，将 `node` 的 `next` 指针指向 `p` 的下一个结点；
5.  将 `p` 的 `next` 指针指向 `node`。

具体过程可参考下图：

1.  ![](./images/list-insert-cyclic-1.svg)
2.  ![](./images/list-insert-cyclic-2.svg)
3.  ![](./images/list-insert-cyclic-3.svg)

代码实现如下：

???+ note "实现"
    === "C++"
        ```c++
        void insertNode(int i, Node *p) {
          Node *node = new Node;
          node->value = i;
          node->next = NULL;
          if (p == NULL) {
            p = node;
            node->next = node;
          } else {
            node->next = p->next;
            p->next = node;
          }
        }
        ```
    
    === "Python"
        ```python
        def insertNode(i, p):
            node = Node()
            node.value = i
            node.next = None
            if p == None:
                p = node
                node.next = node
            else:
                node.next = p.next
                p.next = node
        ```

### 双向循环链表

在向双向循环链表插入数据时，除了要判断给定链表是否为空外，还要同时修改左、右两个指针。

大致流程如下：

1.  初始化待插入的数据 `node`；
2.  判断给定链表 `p` 是否为空；
3.  若为空，则将 `node` 的 `left` 和 `right` 指针，以及 `p` 都指向自己；
4.  否则，将 `node` 的 `left` 指针指向 `p`;
5.  将 `node` 的 `right` 指针指向 `p` 的右结点；
6.  将 `p` 右结点的 `left` 指针指向 `node`；
7.  将 `p` 的 `right` 指针指向 `node`。

代码实现如下：

???+ note "实现"
    === "C++"
        ```c++
        void insertNode(int i, Node *p) {
          Node *node = new Node;
          node->value = i;
          if (p == NULL) {
            p = node;
            node->left = node;
            node->right = node;
          } else {
            node->left = p;
            node->right = p->right;
            p->right->left = node;
            p->right = node;
          }
        }
        ```
    
    === "Python"
        ```python
        def insertNode(i, p):
            node = Node()
            node.value = i
            if p == None:
                p = node
                node.left = node
                node.right = node
            else:
                node.left = p
                node.right = p.right
                p.right.left = node
                p.right = node
        ```

## 从链表中删除数据

### 单向（循环）链表

设待删除结点为 `p`，从链表中删除它时，将 `p` 的下一个结点 `p->next` 的值覆盖给 `p` 即可，与此同时更新 `p` 的下下个结点。

流程大致如下：

1.  将 `p` 下一个结点的值赋给 `p`，以抹掉 `p->value`；
2.  新建一个临时结点 `t` 存放 `p->next` 的地址；
3.  将 `p` 的 `next` 指针指向 `p` 的下下个结点，以抹掉 `p->next`；
4.  删除 `t`。此时虽然原结点 `p` 的地址还在使用，删除的是原结点 `p->next` 的地址，但 `p` 的数据被 `p->next` 覆盖，`p` 名存实亡。

具体过程可参考下图：

1.  ![](./images/list-delete-1.svg)
2.  ![](./images/list-delete-2.svg)
3.  ![](./images/list-delete-3.svg)

代码实现如下：

???+ note "实现"
    === "C++"
        ```c++
        void deleteNode(Node *p) {
          p->value = p->next->value;
          Node *t = p->next;
          p->next = p->next->next;
          delete t;
        }
        ```
    
    === "Python"
        ```python
        def deleteNode(p):
            p.value = p.next.value
            p.next = p.next.next
        ```

### 双向循环链表

流程大致如下：

1.  将 `p` 左结点的右指针指向 `p` 的右节点；
2.  将 `p` 右结点的左指针指向 `p` 的左节点；
3.  新建一个临时结点 `t` 存放 `p` 的地址；
4.  将 `p` 的右节点地址赋给 `p`，以避免 `p` 变成悬垂指针；
5.  删除 `t`。

代码实现如下：

???+ note "实现"
    === "C++"
        ```c++
        void deleteNode(Node *&p) {
          p->left->right = p->right;
          p->right->left = p->left;
          Node *t = p;
          p = p->right;
          delete t;
        }
        ```
    
    === "Python"
        ```python
        def deleteNode(p):
            p.left.right = p.right
            p.right.left = p.left
            p = p.right
        ```

## 技巧

### 异或链表

异或链表（XOR Linked List）本质上还是 **双向链表**，但它利用按位异或的值，仅使用一个指针的内存大小便可以实现双向链表的功能。

我们在结构 `Node` 中定义 `lr = left ^ right`，即前后两个元素地址的 **按位异或值**。正向遍历时用前一个元素的地址异
或当前节点的 `lr` 可得到后一个元素的地址，反向遍历时用后一个元素的地址异或当前节点的 `lr` 又可得到前一个的元素地址。
这样一来，便可以用一半的内存实现双向链表同样的功能。

## 例题

???+note "[约瑟夫问题](https://www.luogu.com.cn/problem/P1996)"
    **题目描述：**n个人围成一圈，从第一个人开始报数，数到 m 的人出列，再由下一个人重新从 1 开始报数，数到 m 的人再出圈，依次类推，直到所有的人都出圈，请输出依次出圈人的编号。

    **输入输出：**输入两个整数 n,m。输出一行n个整数，按顺序输出每个出圈人的编号。1≤m,n≤100。

    **输入输出样例：**
    
    输入
    
    10 3
    
    输出
    
    3 6 9 2 7 1 8 5 10 4

    ??? tip
        下面给出动态链表、静态链表、STL链表等5种实现方案。其中有单向链表，也有双向链表。在竞赛中，为加快编码速度，一般用静态链表或者STL list。

        本文给出的5种代码，逻辑和流程完全一样，看懂一个，其他的完全类似，可以把注意力放在不同的实现方案上，便于学习。

    ??? note "参考代码"
        === "动态链表"
            教科书都会讲动态链表，它需要临时分配链表节点、使用完毕后释放链表节点。这样做，优点是能及时释放空间，不使用多余内存。缺点是很容易出错。下面代码实现了动态单向链表。

            ```cpp
            #include <bits/stdc++.h>
            struct node{          //链表结构
                int data;
                node *next;
            };
            int main(){
                int n,m;
                scanf("%d %d",&n,&m);
                node *head,*p,*now,*prev;   //定义变量
                head = new node; head->data = 1; head->next=NULL; //分配第一个节点，数据置为1        
                now = head;                 //当前指针是头
                for(int i=2;i<=n;i++){
                    p = new node;  p->data = i; p->next = NULL;  //p是新节点
                    now->next = p;        //把申请的新节点连到前面的链表上
                    now = p;              //尾指针后移一个
                }    
                now->next = head;            //尾指针指向头：循环链表建立完成
                //以上是建立链表，下面是本题的逻辑和流程。后面4种代码，逻辑流程完全一致。
                now = head, prev=head;      //从第1个开始数
                while((n--) >1 ){ 
                    for(int i=1;i<m;i++){       //数到m，停下
                        prev = now;             //记录上一个位置，用于下面跳过第m个节点
                        now = now->next; 
                    }
                    printf("%d ", now->data);       //输出第m节点，带空格
                    prev->next = now->next;         //跳过这个节点
                    delete now;                     //释放节点
                    now = prev->next;               //新的一轮        
                }
                printf("%d", now->data);            //打印最后一个，后面不带空格
                delete now;                         //释放最后一个节点
                return 0;
            }
            ```
            
        === "用结构体实现单向静态链表"
            上面的动态链表，需要分配和释放空间，虽然对空间的使用很节省，但是容易出错。在竞赛中，对内存管理要求不严格，为加快编码速度，一般就静态分配，省去了动态分配和释放的麻烦。这种静态链表，使用预先分配的大数组来存储链表。

            静态链表有两种做法，一是定义一个链表结构，和动态链表的结构差不多；一种是使用一维数组，直接在数组上进行链表操作。

            ```cpp
            #include <bits/stdc++.h>
            const int maxn = 105;        //定义静态链表的空间大小
            struct node{                 //单向链表
                int id;
                //int data;   //如有必要，定义一个有意义的数据
                int nextid;
            }nodes[maxn];
            int main(){
                int n, m;
                scanf("%d%d", &n, &m);
                nodes[0].nextid = 1;
                for(int i = 1; i <= n; i++){
                    nodes[i].id = i;
                    nodes[i].nextid = i + 1;
                }
                nodes[n].nextid = 1;                     //循环链表：尾指向头
                int now = 1, prev = 1;                   //从第1个开始
                while((n--) >1){
                    for(int i = 1; i < m; i++){          //数到m，停下
                        prev = now;  
                        now = nodes[now].nextid;
                    }
                    printf("%d ", nodes[now].id);        //带空格
                    nodes[prev].nextid = nodes[now].nextid;  //跳过节点now，即删除now
                    now = nodes[prev].nextid;            //新的now
                }    
                printf("%d", nodes[now].nextid);         //打印最后一个，后面不带空格
                return 0; 
            }
            ```

        === "用结构体实现双向静态链表"

            ```cpp
            #include <bits/stdc++.h>
            const int maxn = 105;
            struct node{      //双向链表
                int id;       //节点编号
                //int data;   //如有必要，定义一个有意义的数据
                int preid;    //前一个节点
                int nextid;   //后一个节点
            }nodes[maxn];
            int main(){
                int n, m;
                scanf("%d%d", &n, &m);
                nodes[0].nextid = 1;
                for(int i = 1; i <= n; i++){  //建立链表
                    nodes[i].id = i;
                    nodes[i].preid = i-1;     //前节点
                    nodes[i].nextid = i+1;    //后节点
                }
                nodes[n].nextid = 1;          //循环链表：尾指向头
                nodes[1].preid = n;           //循环链表：头指向尾
                int now = 1;                  //从第1个开始
                while((n--) >1){
                    for(int i = 1; i < m; i++)     //数到m，停下
                        now = nodes[now].nextid;
                    printf("%d ", nodes[now].id);  //打印，后面带空格
                    int prev = nodes[now].preid;   
                    int next = nodes[now].nextid;
                    nodes[prev].nextid = nodes[now].nextid;  //删除now
                    nodes[next].preid = nodes[now].preid;   
                    now = next;                    //新的开始
                }    
                printf("%d", nodes[now].nextid);   //打印最后一个，后面不带空格
                return 0; 
            }
            ```

        === "用数组实现单向静态链表"
            这是最简单的实现方法。定义一个一维数组nodes[]，nodes[i]的i是节点的值，nodes[i]的值是下一个节点。
            
            从上面描述可以看出，它的使用环境也很有限，因为它的节点只能存一个数据，就是i。

            ```cpp
            #include<bits/stdc++.h>
            int nodes[150];
            int main(){   
                int n, m;
                scanf("%d%d", &n, &m); 
                for(int i=1;i<=n-1;i++)          //nodes[i]的值就是下一个节点
                    nodes[i]=i+1;
                nodes[n]=1;                      //循环链表：尾指向头
                int now = 1, prev = 1;           //从第1个开始
                while((n--) >1){
                    for(int i = 1; i < m; i++){   //数到m，停下
                        prev = now;  
                        now = nodes[now];         //下一个
                    }
                    printf("%d ", now);  //带空格
                    nodes[prev] = nodes[now];     //跳过节点now，即删除now
                    now = nodes[prev];            //新的now
                }    
                printf("%d", now);                //打印最后一个，不带空格
                return 0;
            }
            ```

        === "STL List"
            竞赛或工程中，常常使用C++ STL list。list 是双向链表，它的内存空间可以是不连续的，通过指针来进行数据的访问，它能高效率地在任意地方删除和插入，插入和删除操作是常数时间的。
            
            请读者自己熟悉list的初始化、添加、遍历、插入、删除、查找、排序、释放 [参考](https://blog.csdn.net/zhouzhenhe2008/article/details/77428743)。

            ```cpp
            #include <bits/stdc++.h>
            using namespace std;
            int main(){
                int n, m;
                cin>>n>>m;
                list<int>node;
                for(int i=1;i<=n;i++)         //建立链表
                    node.push_back(i);     
                list<int>::iterator it = node.begin();
                while(node.size()>1){         //list的大小由STL自己管理
                    for(int i=1;i<m;i++){     //数到m
                        it++; 
                        if(it == node.end()) //循环链表，end()是list末端下一位置
                            it = node.begin();                                              
                    }
                    cout << *it <<"";
                    list<int>::iterator next = ++it;
                    if(next==node.end())  next=node.begin();  //循环链表
                    node.erase(--it);         //删除这个节点，node.size()自动减1
                    it = next;
                }
                cout << *it;
                return 0;
            }
            ```

## 习题
- [线性表题单 - 洛谷](https://www.luogu.com.cn/training/113)
