## 引入

何为单调栈？顾名思义，单调栈即满足单调性的栈结构。与单调队列相比，其只在一端进行进出。

为了描述方便，以下举例及伪代码以维护一个整数的单调递增栈为例。

## 过程

### 插入

将一个元素插入单调栈时，为了维护栈的单调性，需要在保证将该元素插入到栈顶后整个栈满足单调性的前提下弹出最少的元素。

例如，栈中自顶向下的元素为 $\{0,11,45,81\}$。

![](images/monotonous-stack-before.svg)

插入元素 $14$ 时为了保证单调性需要依次弹出元素 $0,11$，操作后栈变为 $\{14,45,81\}$。

![](images/monotonous-stack-after.svg)

用伪代码描述如下：

???+note "实现"
    ```text
    insert x
    while !sta.empty() && sta.top()<x
        sta.pop()
    sta.push(x)
    ```

### 使用

自然就是从栈顶读出来一个元素，该元素满足单调性的某一端。

例如举例中取出的即栈中的最小值。

## 例题

??? note "[向右看齐](https://www.luogu.com.cn/problem/P2947)"
    **题目描述：** N(1≤N≤10^5)头奶牛站成一排，奶牛i的身高是Hi(l≤Hi≤1,000,000)。现在，每只奶牛都在向右看齐。对于奶牛i，如果奶牛j满足$i\lt j$且$Hi\lt Hj$，我们说奶牛i仰望奶牛j。求出每只奶牛离她最近的仰望对象。
    
    **输入输出：** 第 1 行输入 N，之后每行输入一个身高 Hi。输出共 N 行，按顺序每行输出一只奶牛的最近仰望对象，如果没有仰望对象，输出 0。

??? note "解题思路"
    从后往前遍历奶牛，并用一个栈保存从低到高的奶牛，栈顶的奶牛最矮，栈底的最高。具体操作是：遍历到奶牛i时，与栈顶的奶牛比较，如果不比i高，就弹出栈顶，直到栈顶的奶牛比i高，这就是i的仰望对象；然后把i放进栈顶，栈里的奶牛仍然保持从低到高。

    复杂度：每个奶牛只进出栈一次，所以是O(n)的。

??? note "参考代码"
    === "STL stack"

        ```cpp
        #include<bits/stdc++.h>
        using namespace std;
        int h[100001], ans[100001];
        int main(){
            int n;
            scanf("%d",&n);
            for (int i=1;i<=n;i++)  scanf("%d",&h[i]);
            stack<int>st; 
            for (int i=n;i>=1;i--){
                while (!st.empty() && h[st.top()] <= h[i])  //栈顶奶牛没我高，弹出它，直到栈顶奶牛更高
                    st.pop();
                if (st.empty())       //栈空，没有仰望对象
                    ans[i]=0; 
                else                  //栈顶奶牛更高，是仰望对象
                    ans[i]=st.top();
                st.push(i);
            }
            for (int i=1;i<=n;i++) 
                printf("%d\n",ans[i]);
            return 0;
        }
        ```

    === "手写栈"

        ```cpp
        #include<bits/stdc++.h>
        using namespace std;
        const int maxn = 100000 + 100;
        struct mystack{
            int a[maxn];                        //存放栈元素，int型
            int t = 0;                          //栈顶位置
            void push(int x){ a[++t] = x;  }    //送入栈
            int  top()      { return a[t]; }    //返回栈顶元素
            void pop()      { t--;         }    //弹出栈顶
            int empty()     { return t==0?1:0;} //返回1表示空
        }st;
        int h[maxn], ans[maxn];
        int main(){
            int n;
            scanf("%d",&n);
            for (int i=1;i<=n;i++)  scanf("%d",&h[i]);
            for (int i=n;i>=1;i--){
                while (!st.empty() && h[st.top()] <= h[i])  //栈顶奶牛没我高，弹出它，直到栈顶奶牛更高
                    st.pop();
                if (st.empty())       //栈空，没有仰望对象
                    ans[i]=0; 
                else                  //栈顶奶牛更高，是仰望对象
                    ans[i]=st.top();
                st.push(i);
            }
            for (int i=1;i<=n;i++) 
                printf("%d\n",ans[i]);
            return 0;
        }
        ```

??? note "[POJ3250 Bad Hair Day](http://poj.org/problem?id=3250)"
    有 $N$ 头牛从左到右排成一排，每头牛有一个高度 $h_i$，设左数第 $i$ 头牛与「它右边第一头高度 $≥h_i$」的牛之间有 $c_i$ 头牛，试求 $\sum_{i=1}^{N} c_i$。

比较基础的应用有这一题，就是个单调栈的简单应用，记录每头牛被弹出的位置，如果没有被弹出过则为最远端，稍微处理一下即可计算出题目所需结果。

另外，单调栈也可以用于离线解决 RMQ 问题。

我们可以把所有询问按右端点排序，然后每次在序列上从左往右扫描到当前询问的右端点处，并把扫描到的元素插入到单调栈中。这样，每次回答询问时，单调栈中存储的值都是位置 $\le r$ 的、可能成为答案的决策点，并且这些元素满足单调性质。此时，单调栈上第一个位置 $\ge l$ 的元素就是当前询问的答案，这个过程可以用二分查找实现。使用单调栈解决 RMQ 问题的时间复杂度为 $O(q\log q + q\log n)$，空间复杂度为 $O(n)$。

## 习题

- [洛谷 P5788【模板】单调栈](https://www.luogu.com.cn/problem/P5788)
- [洛谷 P1901 发射站](https://www.luogu.com.cn/problem/P1901)
- [Largest Rectangle in a Histogram](http://poj.org/problem?id=2559)
- [Areas on the Cross-Section Diagram](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/3/ALDS1_3_D)
