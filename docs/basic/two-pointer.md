本页面将简要介绍双指针。

## 引入

尺取法（又称为：双指针、two pointers）是一种简单而又灵活的技巧和思想，单独使用可以轻松解决一些特定问题，和其他算法结合也能发挥多样的用处。

双指针顾名思义，就是同时使用两个指针，在序列、链表结构上指向的是位置，在树、图结构中指向的是节点，通过或同向移动，或相向移动来维护、统计信息。

如果区间是单调的，也常常用二分法来求解，所以很多问题用尺取法和二分法都行。

另外，尺取法的的操作过程和分治算法的步骤很相似，有时候也用在分治中。

## 理论
为什么尺取法能优化呢？考虑下面的应用背景：

1. 给定一个序列。有时候需要它是有序的，先排序。
2. 问题和序列的区间有关，且需要操作2个变量，可以用两个下标（指针）i、j扫描区间。

对于上面的应用，一般的做法，是用i、j分别扫描区间，有两重循环，复杂度O(n^2)。以反向扫描（即i、j方向相反，后文有解释）为例，代码是：

```cpp
for(int i = 0; i < n; i++)           //i从头扫到尾
	for(int j = n-1; j >= 0; j--){   //j从尾扫到头
        ......
    }
```

下面用尺取法来优化上面的算法。

实际上，尺取法就是把两重循环变成了一个循环，在这个循环中一起处理i和j。复杂度也就从O(n^2)变成了O(n)。仍以上面的反向扫描为例，代码是：

```cpp
//用while实现：
int i = 0, j = n - 1;
while (i < j) {      //i和j在中间相遇。这样做还能防止i、j越界
        ......       //满足题意的操作
        i++;         //i从头扫到尾
        j--;         //j从尾扫到头
}
//用for实现：
for (int i = 0, j = n - 1; i < j; i++, j--) {
    ......
}
```

在尺取法中，这两个指针i、j，有两种扫描方向：

1. 反向扫描。i、j方向相反，i从头到尾，j从尾到头，在中间相会。
2. 同向扫描。i、j方向相同，都从头到尾，可以让j跑在i前面。

在leetcode的一篇文章中 常用的双指针技巧 https://leetcode-cn.com/circle/article/GMopsy/，把同向扫描的i、j指针称为“快慢指针”，把反向扫描的i、j指针称为“左右指针”，更加形象。快慢指针在序列上产生了一个大小可变的“滑动窗口”，有灵活的应用，例如3.1的“寻找区间和”问题。

## 反向扫描

???+note "找指定和的整数对"
    输入n ( n≤100,000)个整数，放在数组a[]中。找出其中的两个数，它们之和等于整数m(假定肯定有解)。题中所有整数都是int型。
    
    样例输入：
    
    21 4 5 6 13 65 32 9 23
    
    28
    
    样例输出：
    
    5 23
    
    说明：样例输入的第一行是数组a[]，第2行是m = 28。样例输出5和23，相加得28。

???+note "解题思路"
    为了说明尺取法的优势，下面给出四种方法：

    1. 用两重循环暴力搜，枚举所有的取数方法，复杂度O(n^2)，超时。暴力法不需要排序。
    2. 二分法。首先对数组从小到大排序，复杂度O(nlogn)；然后，从头到尾处理数组中的每个元素a[i]，在大于a[i]的数中二分查找是否存在一个等于 m - a[i]的数，复杂度也是O(nlogn)。两部分相加，总复杂度仍然是O(nlogn)。
    3. Hash。分配一个hash空间s，把n个数放进去。逐个检查a[]中的n个数，例如a[i]，检查m - a[i]在s中是否有值，如果有，那么存在一个答案。复杂度是O(n)。hash方法很快，但是需要一个额外的、可能很大的hash空间。
    4. 尺取法。这是标准解法。首先对数组从小到大排序；然后，设置两个变量i和j，分别指向头和尾，i初值是0，j初值是n-1，然后让i和j逐渐向中间移动，检查a[i]+a[j]，如果大于m，就让j减1，如果小于m，就让i加1，直至a[i]+a[j] = m。排序复杂度O(nlogn)，检查的复杂度O(n)，合起来总复杂度O(nlogn)。
    
    尺取法代码如下，注意可能有多个答案：

    ```cpp
    void find_sum(int a[], int n, int m){ 
        sort(a, a + n - 1);      //先排序，复杂度O(nlogn)
        int i = 0, j = n - 1;    //i指向头，j指向尾
        while (i < j){           //复杂度O(n)
                int sum = a[i] + a[j];
                if (sum > m)   j--;
                if (sum < m)   i++;
                if (sum == m){     
                    cout << a[i] << " " << a[j] << endl;  //打印一种情况
                    i++;          //可能有多个答案，继续
                }
        }
    }
    ```

    在这个题目中，尺取法不仅效率高，而且不需要额外的空间。
    
    把题目的条件改变一下，可以变化为类似的问题，例如：判断一个数是否为两个数的平方和。
    
    这个题目，其实也能用同向扫描来做。请读者思考。

???+note "[判断回文串](http://acm.hdu.edu.cn/showproblem.php?pid=2029)"
    “回文串”是一个正读和反读都一样的字符串，比如“level”或者“noon”就是回文串。写一个程序判断读入的字符串是否是“回文”。如果是，输出“yes”，否则输出“no”。

???+note "参考代码"

    ```cpp
    #include <bits/stdc++.h>
    using namespace std;
    int main(){
        int n;
        cin >> n;                         //n是测试用例个数
        while(n--){
            string s;  cin >> s;          //读一个字符串
            bool ans = true;
            int i = 0, j = s.size() - 1;  //双指针
            while(i < j){ 
                if(s[i] != s[j]){
                    ans = false;
                    break;
                }
                i++;   j--;               //移动双指针
            }
            if(ans)   cout << "yes" << endl;
            else      cout << "no"  << endl;
        }
        return 0;
    }
    ```

    稍微改变一下，类似的题目有：

    1. 不区分大小写，忽略非英文字母，判断是否回文串。http://www.lintcode.com/problem/valid-palindrome/
    2. 允许删除（或插入，本题只考虑删除）最多1个字符，判断是否能构成回文字符串。http://www.lintcode.com/problem/valid-palindrome-ii/
    
    设反向扫描双指针为i、j。如果 s[i]和s[j]相同，i++、j--；如果s[i]和s[j]不同，那么，或者删除s[i]，或者删除s[j]，看剩下的字符串是否是回文串即可。

## 同向扫描

???+note "寻找区间和"
    给定一个长度为n的数组a[]和一个数s，在这个数组中找一个区间，使得这个区间之和等于s。输出区间的起点和终点位置。
    
    样例输入：
    
    15
    
    6 1 2 3 4 6 4 2 8 9 10 11 12 13 14
    
    6
    
    样例输出：
    
    0 0
    
    1 3
    
    5 5
    
    6 7
    
    说明：样例输入的第1行是n=15，第2行是数组a[]，第3行是区间和s=6。样例输出，共有4个情况。

???+note "解题思路"
    指针i和j，i<=j，都从头向尾扫描，判断区间[i,j]的和是否等于s。
    
    如何寻找区间和等于s的区间？如果简单地对i和j做二重循环，复杂度是O(n2)。用尺取法，复杂度O(n)，操作步骤是：

    1. 初始值i=0、j=0，即开始都指向第一个元素a[0]。定义sum是区间[i, j]的和，初始值sum = a[0]。
    2. 如果sum等于s，输出一个解。继续，把sum减掉元素a[i]，并把i往后移动一位。
    3. 如果sum大于s，让sum减掉元素a[i]，并把i往后移动一位。
    4. 如果sum小于s，把j往后挪一位，并把sum的值加上这个新元素。

    在上面的步骤中，有2个关键技巧：
    
    1. 滑动窗口的实现。窗口就是区间[i,j]，随着i和j从头到尾移动，窗口就“滑动”扫描了整个序列，检索了所有的数据。i和j并不是同步增加的，窗口像一只蚯蚓伸缩前进，它的长度是变化的，这个变化，正对应了对区间和的计算。
    2. sum的使用。如何计算区间和？暴力的方法是从a[i]到a[j]累加，但是，这个累加的复杂度是O(n)的，会超时。如果利用sum，每次移动i或j的时候，只需要把sum加或减一次，就得到了区间和，复杂度是O(1)。这是“前缀和”递推思想的应用。

    “滑动窗口”的例子还有：

    1. 给定一个序列，以及一个整数M；在序列中找M个连续递增的元素，使它们的区间和最大。
    2. 给定一个序列，以及一个整数K；求一个最短的连续子序列，其中包含至少K个不同的元素。

???+note "参考代码"

    ```cpp
    void findsum(int *a, int n, int s){
        int i = 0, j = 0;
        int sum = a[0];
        while(j < n){   //下面代码中保证 i<=j
            if(sum >= s){
                if(sum == s) printf("%d %d\n", i, j);
                sum -= a[i];
                i++;
                if(i>j) {sum = a[i]; j++;}  //防止i超过j
            }
            if(sum < s){
                j++;
                sum += a[j];
            }
        }
    }
    ```

???+note "数组去重"
    给定数组a[]，长度为n，把数组中重复的数去掉。

???+note "解题思路"
    下面给出两种解法：hash和尺取法。
    
    hash。hash函数的特点是有冲突，利用这个特点去重。把所有的数插到 hash表里，用冲突过滤重复的数，就能得到不同的数。缺点是会耗费额外的空间。
    
    尺取法。步骤是：

    1. 将数组排序，这样那些重复的整数就会挤在一起。
    2. 定义双指针i、j，初始值都指向a[0]。i和j都从头到尾扫描数组a[]。i指针走得快，逐个遍历整个数组；j指针走得慢，它始终指向当前不重复部分的最后一个数。也就是说，j用于获得不重复的数。
    3. 扫描数组。快指针i++，如果此时a[i]不等于慢指针j指向的a[j]，就把j++，并且把a[i]复制到慢指针j的当前位置a[j]。
    4. i扫描结束后，a[0]到a[j]就是不重复数组。

## 维护区间信息

如果不和其他数据结构结合使用，双指针维护区间信息的最简单模式就是维护具有一定单调性，新增和删去一个元素都很方便处理的信息，就比如正数的和、正整数的积等等。

### 例题 1

???+ note "例题 1 [leetcode 713. 乘积小于 K 的子数组](https://leetcode-cn.com/problems/subarray-product-less-than-k/)"
    给定一个长度为 $n$ 的正整数数组 $\mathit{nums}$ 和整数 $k$, 找出该数组内乘积小于 $k$ 的连续子数组的个数。$1 \leq n \leq 3 \times 10^4, 1 \leq nums[i] \leq 1000, 0 \leq k \leq 10^6$

#### 过程

设两个指针分别为 $l,r$, 另外设置一个变量 $\mathit{tmp}$ 记录 $[l,r]$ 内所有数的乘积。最开始 $l,r$ 都在最左面，先向右移动 $r$，直到第一次发现 $\mathit{tmp}\geq k$,  这时就固定 $r$，右移 $l$，直到 $\mathit{tmp}\lt k$。那么对于每个 $r$，$l$ 是它能延展到的左边界，由于正整数乘积的单调性，此时以 $r$ 为右端点的满足题目条件的区间个数为 $r-l+1$ 个。

#### 实现

```cpp
int numSubarrayProductLessThanK(vector<int>& nums, int k) {
  long long ji = 1ll, ans = 0;
  int l = 0;
  for (int i = 0; i < nums.size(); ++i) {
    ji *= nums[i];
    while (l <= i && ji >= k) ji /= nums[l++];
    ans += i - l + 1;
  }
  return ans;
}
```

使用双指针维护区间信息也可以与其他数据结构比如差分、单调队列、线段树、主席树等等结合使用。另外将双指针技巧融入算法的还有莫队，莫队中将询问离线排序后，一般也都是用两个指针记录当前要处理的区间，随着指针一步步移动逐渐更新区间信息。

### 例题 2

接下来看一道在树上使用双指针并结合树上差分的例题：

???+ note "例题 2 [luogu P3066 Running Away From the Barn G](https://www.luogu.com.cn/problem/P3066)"
    给定一颗 $n$ 个点的有根树，边有边权，节点从 1 至 $n$ 编号，1 号节点是这棵树的根。再给出一个参数 $t$，对于树上的每个节点 $u$，请求出 $u$ 的子树中有多少节点满足该节点到 $u$ 的距离不大于 $t$。数据范围：$1\leq n \leq 2\times 10^5,1 \leq t \leq 10^{18},1 \leq p_i \lt i,1 \leq w_i \leq 10^{12}$

#### 过程

从根开始用 dfs 遍历整棵树，使用一个栈来记录根到当前节点的树链，设一个指针 $u$ 指向当前节点，另一个指针 $p$ 指向与 $u$ 距离不大于 $t$ 的节点中深度最小的节点。记录到根的距离，每次二分查找确定 $p$。此时 $u$ 对 $p$ 到 $u$ 路径上的所有节点都有一个贡献，可以用树上差分来记录。  
注意不能直接暴力移动 $p$，否则时间复杂度可能会退化至 $O(n^2)$。

## 子序列匹配

???+ note "例题 3 [leetcode 524. 通过删除字母匹配到字典里最长单词](https://leetcode-cn.com/problems/longest-word-in-dictionary-through-deleting/)"
    给定一个字符串 $s$ 和一个字符串数组 $\mathit{dictionary}$ 作为字典，找出并返回字典中最长的字符串，该字符串可以通过删除 $s$ 中的某些字符得到。

### 过程

此类问题需要将字符串 $s$ 与 $t$ 进行匹配，判断 $t$ 是否为 $s$ 的子序列。解决这种问题只需先将两个指针一个 $i$ 放在 $s$ 开始位置，一个 $j$ 放在 $t$ 开始位置，如果 $s[i]=t[j]$ 说明 $t$ 的第 $j$ 位已经在 $s$ 中找到了第一个对应，可以进而检测后面的部分了，那么 $i$ 和 $j$ 同时加一。如果上述等式不成立，则 $t$ 的第 $j$ 位仍然没有被匹配上，所以只给 $i$ 加一，在 $s$ 的后面部分再继续寻找。最后，如果 $j$ 已经移到了超尾位置，说明整个字符串都可以被匹配上，也就是 $t$ 是 $s$ 的一个子序列，否则不是。

### 实现

```cpp
string findLongestWord(string s, vector<string>& dictionary) {
  sort(dictionary.begin(), dictionary.end());
  int mx = 0, r = 0;
  string ans = "";
  for (int i = dictionary.size() - 1; i >= 0; i--) {
    r = 0;
    for (int j = 0; j < s.length(); ++j) {
      if (s[j] == dictionary[i][r]) r++;
    }
    if (r == dictionary[i].length()) {
      if (r >= mx) {
        mx = r;
        ans = dictionary[i];
      }
    }
  }
  return ans;
}
```

这种两个指针指向不同对象然后逐步进行比对的方法还可以用在一些 dp 中。

## 利用序列有序性

很多时候在序列上使用双指针之所以能够正确地达到目的，是因为序列的某些性质，最常见的就是利用序列的有序性。

???+ note "例题 4 [leetcode 167. 两数之和 II - 输入有序数组](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/)"
    给定一个已按照 **升序排列** 的整数数组 `numbers`，请你从数组中找出两个数满足相加之和等于目标数 `target`。

### 过程

这种问题也是双指针的经典应用了，虽然二分也很方便，但时间复杂度上多一个 $\log{n}$，而且代码不够简洁。

接下来介绍双指针做法：既然要找到两个数，且这两个数不能在同一位置，那其位置一定是一左一右。由于两数之和固定，那么两数之中的小数越大，大数越小。考虑到这些性质，那我们不妨从两边接近它们。

首先假定答案就是 1 和 n，如果发现 $num[1]+num[n]\gt \mathit{target}$，说明我们需要将其中的一个元素变小，而 $\mathit{num}[1]$ 已经不能再变小了，所以我们把指向 $n$ 的指针减一，让大数变小。

同理如果发现 $num[1]+num[n]\lt \mathit{target}$，说明我们要将其中的一个元素变大，但 $\mathit{num}[n]$ 已经不能再变大了，所以将指向 1 的指针加一，让小数变大。

推广到一般情形，如果此时我们两个指针分别指在 $l,r$ 上，且 $l\lt r$, 如果 $num[l]+num[r]\gt \mathit{target}$，就将 $r$ 减一，如果 $num[l]+num[r]\lt \mathit{target}$，就将 $l$ 加一。这样 $l$ 不断右移，$r$ 不断左移，最后两者各逼近到一个答案。

### 实现

```cpp
vector<int> twoSum(vector<int>& numbers, int target) {
  int r = numbers.size() - 1, l = 0;
  vector<int> ans;
  ans.clear();
  while (l < r) {
    if (numbers[l] + numbers[r] > target)
      r--;
    else if (numbers[l] + numbers[r] == target) {
      ans.push_back(l + 1), ans.push_back(r + 1);
      return ans;
    } else
      l++;
  }
  return ans;
}
```

在归并排序中，在 $O(n+m)$ 时间内合并两个有序数组，也是保证数组的有序性条件下使用的双指针法。

## 在单向链表中找环

### 过程

在单向链表中找环也是有多种办法，不过快慢双指针方法是其中最为简洁的方法之一，接下来介绍这种方法。

首先两个指针都指向链表的头部，令一个指针一次走一步，另一个指针一次走两步，如果它们相遇了，证明有环，否则无环，时间复杂度 $O(n)$。

如果有环的话，怎么找到环的起点呢？

我们列出式子来观察一下，设相遇时，慢指针一共走了 $k$ 步，在环上走了 $l$ 步（快慢指针在环上相遇时，慢指针一定没走完一圈）。快指针走了 $2k$ 步，设环长为 $C$，则有

$$
\begin{align}
& \ 2 k=n \times C+l+(k-l) \\
& \ k=n \times C \\
\end{align}
$$

第一次相遇时 $n$ 取最小正整数 1。也就是说 $k=C$。那么利用这个等式，可以在两个指针相遇后，将其中一个指针移到表头，让两者都一步一步走，再度相遇的位置即为环的起点。

### 习题

- [leetcode 15. 三数之和](https://leetcode-cn.com/problems/3sum/)
- [leetcode 1438. 绝对差不超过限制的最长连续子数组](https://leetcode-cn.com/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/)
- [Subsequence](http://poj.org/problem?id=3061)
- [Jessica's Reading Problem](http://poj.org/problem?id=3320)
- [Bound Found](http://poj.org/problem?id=2566)
- [First One](https://vjudge.net/problem/HDU-5358)
- [P1102 A-B 数对](https://www.luogu.com.cn/problem/P1102)
- [Cave](https://vjudge.net/problem/UVA-1442)