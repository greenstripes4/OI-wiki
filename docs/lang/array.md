数组是存放相同类型对象的容器，数组中存放的对象没有名字，而是要通过其所在的位置访问。数组的大小是固定的，不能随意改变数组的长度。

## 定义数组

数组的声明形如 `a[d]`，其中，`a` 是数组的名字，`d` 是数组中元素的个数。在编译时，`d` 应该是已知的，也就是说，`d` 应该是一个整型的常量表达式。

```cpp
unsigned int d1 = 42;
const int d2 = 42;
int arr1[d1];  // 错误：d1 不是常量表达式
int arr2[d2];  // 正确：arr2 是一个长度为 42 的数组
```

不能将一个数组直接赋值给另一个数组：

```cpp
int arr1[3];
int arr2 = arr1;  // 错误
arr2 = arr1;      // 错误
```

应该尽量将较大的数组定义为全局变量。因为局部变量会被创建在栈区中，过大（大于栈的大小）的数组会爆栈，进而导致 RE。如果将数组声明在全局作用域中，就会在静态区中创建数组。

## 初始化数组

```cpp
int a[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
int f1[101];			   //没赋初始值的情况下，数组f[]内的各元素值未知
memset(f1, 0, sizeof(f1)); //将数组f[]内的数组元素全部赋值为0
memset(f1, 0x3f, sizeof(f1)); //将数组f[]内的数组元素赋值为较大值且防止溢出
int a[3][4] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};
int b[][4] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12}; //第一维的下标值可省略，第二维的下标值不可省略
int c[][4] = {{0, 0, 3}, {}, {0, 10}};				  //未赋值的数组元素均自动初始化为0
char c1[] = {"I am ok"}; //定义字符数组时全部元素均赋值，数组大小定义可省略
char c2[] = "I am ok";
```

## 访问数组元素

可以通过下标运算符 `[]` 来访问数组内元素，数组的索引（即方括号中的值）从 0 开始。以一个包含 10 个元素的数组为例，它的索引为 0 到 9，而非 1 到 10。但在 OI 中，为了使用方便，我们通常会将数组开大一点，不使用数组的第一个元素，从下标 1 开始访问数组元素。

例 1：从标准输入中读取一个整数 $n$，再读取 $n$ 个数，存入数组中。其中，$n\leq 1000$。

```cpp
#include <iostream>
using namespace std;

int arr[1001];  // 数组 arr 的下标范围是 [0, 1001)

int main() {
  int n;
  cin >> n;
  for (int i = 1; i <= n; ++i) {
    cin >> arr[i];
  }
}
```

例 2：（接例 1）求和数组 `arr` 中的元素，并输出和。满足数组中所有元素的和小于等于 $2^{31} - 1$

```cpp
#include <iostream>
using namespace std;

int arr[1001];

int main() {
  int n;
  cin >> n;
  for (int i = 1; i <= n; ++i) {
    cin >> arr[i];
  }

  int sum = 0;
  for (int i = 1; i <= n; ++i) {
    sum += arr[i];
  }

  printf("%d\n", sum);
  return 0;
}
```

### 越界访问下标

数组的下标 $\mathit{idx}$ 应当满足 $0\leq \mathit{idx}< \mathit{size}$，如果下标越界，则会产生不可预料的后果，如段错误（Segmentation Fault），或者修改预期以外的变量。

## 数组排序

```cpp
int sa[] = {-1, 9, -34, 100, 45, 2, 98, 32}; //无序整型数组元素
int lensa = sizeof(sa) / sizeof(int);
sort(sa, sa + lensa);				  //由小到大排序
sort(sa, sa + lensa, greater<int>()); //由大到小排序

struct student //声明一个名为student的结构体类型
{
	int ID;		   //学号
	char name[20]; //姓名
	char sex;	   //性别
	float score;   //成绩
	bool operator<(student other) const //语法要求必须重载成常成员函数
	{
		if(score!=other.score) return score>other.score;
		else return ID<other.ID;
	}
};				   //声明结束，注意此处不可省略分号

//cmp return true的关系表示不需要交换
bool cmp(student a, student b)
{
	if (a.score != b.score)
		return a.score < b.score; //如果a.score不等于b.score，就按x从小到大排序
	return a.ID < b.ID;			  //如果score相等则按ID从小到大排序
}

student a[100];

sort(a, a+n-1, cmp); //student自定义比较操作符的时候可以直接sort
```

## 数组查找

```cpp
// lower_bound(起始地址first,结束地址last,要查找的数值val)：
// 在first和last中的前闭后开区间进行二分查找，返回大于或等于val的第一个元素地址。
// 如果区间内所有元素都小于val，则返回last的地址，且last的地址是越界的。
// upper_bound(起始地址first,结束地址last,要查找的数值val)：
// 在first和last中的前闭后开区间进行二分查找，返回大于val的第一个元素地址。
// 如果val大于区间内所有元素，则返回last的地址，且last的地址是越界的。
int a[] = {1, 1, 2, 2, 3, 3, 3, 4, 4, 4};
cout << lower_bound(a, a + 10, 0) - a << endl; //输出下标 0
cout << lower_bound(a, a + 10, 1) - a << endl; //输出下标 0
cout << lower_bound(a, a + 10, 3) - a << endl; //输出下标 4
cout << lower_bound(a, a + 10, 4) - a << endl; //输出下标 7
cout << lower_bound(a, a + 10, 5) - a << endl; //输出下标 10
cout << upper_bound(a, a + 10, 0) - a << endl; //输出下标 0
cout << upper_bound(a, a + 10, 1) - a << endl; //输出下标 2
cout << upper_bound(a, a + 10, 3) - a << endl; //输出下标 7
cout << upper_bound(a, a + 10, 4) - a << endl; //输出下标 10
cout << upper_bound(a, a + 10, 5) - a << endl; //输出下标 10

// 降序数组直接使用lower_bound/upper_bound二分查找的结果是错误的
// 这是因为lower_bound/upper_bound默认二分查找的区间是升序序列。
// 在降序序列lower_bound的正确写法为lower_bound(first, last, val, greater<int>())
// upper_bound的正确写法为upper_bound(first, last, val, greater<int>())
int a1[] = {4, 4, 3, 3, 2, 2, 1, 1};
cout << lower_bound(a1, a1 + 8, 0, greater<int>()) - a1 << endl; //输出 8
cout << lower_bound(a1, a1 + 8, 4, greater<int>()) - a1 << endl; //输出 0
cout << lower_bound(a1, a1 + 8, 1, greater<int>()) - a1 << endl; //输出 6
cout << lower_bound(a1, a1 + 8, 3, greater<int>()) - a1 << endl; //输出 2
cout << lower_bound(a1, a1 + 8, 5, greater<int>()) - a1 << endl; //输出 0
cout << upper_bound(a1, a1 + 8, 0, greater<int>()) - a1 << endl; //输出 8
cout << upper_bound(a1, a1 + 8, 4, greater<int>()) - a1 << endl; //输出 2
cout << upper_bound(a1, a1 + 8, 1, greater<int>()) - a1 << endl; //输出 8
cout << upper_bound(a1, a1 + 8, 3, greater<int>()) - a1 << endl; //输出 4
cout << upper_bound(a1, a1 + 8, 5, greater<int>()) - a1 << endl; //输出 0
```

## 多维数组

多维数组的实质是「数组的数组」，即外层数组的元素是数组。一个二维数组需要两个维度来定义：数组的长度和数组内元素的长度。访问二维数组时需要写出两个索引：

```cpp
int arr[3][4];  // 一个长度为 3 的数组，它的元素是「元素为 int 的长度为的 4
                // 的数组」
arr[2][1] = 1;  // 访问二维数组
```

我们经常使用嵌套的 for 循环来处理二维数组。

例：从标准输入中读取两个数 $n$ 和 $m$，分别表示黑白图片的高与宽，满足 $n,m\leq 1000$。对于接下来的 $n$ 行数据，每行有用空格分隔开的 $m$ 个数，代表这一位置的亮度值。现在我们读取这张图片，并将其存入二维数组中。

```cpp
const int maxn = 1001;
int pic[maxn][maxn];
int n, m;

cin >> n >> m;
for (int i = 1; i <= n; ++i)
  for (int j = 1; j <= m; ++j) cin >> pic[i][j];
```

同样地，你可以定义三维、四维，以及更高维的数组。
