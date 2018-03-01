---
layout: post
title:  "关于STL的heap算法"
tag:
- C++
- 算法
comments: true
---

STL没有heap这个容器，但有一系列与heap操作相关的函数（默认是max-heap），适用于带有随机访问迭代器（`RandomAccessIterator`）的容器，如数组、`vector`、`deque`等。

## push_heap

### 概述
顾名思义，该函数的作用是向heap中插入新元素，显然先决条件之一就是要有一个现成的heap。另外还要注意，为了维持heap完全二叉树的特点，该函数规定把新元素置于传入的迭代器范围的尾部，即二叉树最右边的叶节点上。

### 实现细节
由上述使用条件可知，新元素初始情况下位于二叉树最右边的叶节点上，所以要从该节点开始逐步执行“上溯”步骤：拿新节点和其父节点比较，如果该节点的值更大，就将两者对调，如此一直上溯，直到不需对调或到达根节点为止。

### 时间复杂度
最糟糕的情况就是一直从叶节点上溯到根节点而已，比较次数就等于树的高度，因此时间复杂度为O(logN)。

## pop_heap

### 概述
和上一个函数相反，这个函数的作用是把heap根节点pop出去，但不删除，而是放置到尾部，然后对剩下的元素继续维持heap的特点。

### 实现细节
为了始终维持heap完全二叉树的特点，但又必须从树中拿走一个元素，最好的办法就是将最右边的叶节点（对应数组的最后一个元素）和根节点交换，然后调整heap。这里的调整采用“下溯”的策略：把根节点看成一个“洞”，拿来和它两个子节点比较，并与其中键值较大的节点交换位置，然后把“洞”更新为这个子节点的位置，往下迭代，直到不需交换或到达叶节点为止。

### 时间复杂度
最糟糕的情况是从根节点下溯到叶节点，时间复杂度同样是O(logN)。

## sort_heap

### 概述
就是利用建好的heap排序，注意，排完序后，原来的heap就不是合法的heap了。

### 实现细节
有了上述基础，实现排序就简单多了：持续对heap做`pop_heap`操作，每次把操作范围向前缩减一个元素，最终就能得到递增的序列。

### 时间复杂度
有N个元素就需要做N次`pop_heap`操作，不难推出时间复杂度为O(NlogN)。

## make_heap

### 概述
建立heap，是上述函数的基础。值得一提的是，STL有个`partial_sort`函数用于部分排序，它的实现就调用了`make_heap`，以及调用`sort_heap`进行最终的排序。

### 实现细节
采用**bottom-up**的思想，从最靠近底层的非叶节点开始建立一个个的小heap，逐步往上层迭代时，每个节点的左右子树均已是合法的heap，在此基础上进行微调即可。

### 时间复杂度
假设所有节点的数量是N，那么非叶节点的数量则是N/2，对每个非叶节点的微调最多花费O(logN)的开销，综合来看，总的时间复杂度是O(N/2logN)。

## 测试代码
下面是对上述四个函数的测试代码，摘自[cplusplus](http://www.cplusplus.com/reference/algorithm/make_heap/?kw=make_heap)。

```cpp
// range heap example
#include <iostream>     // std::cout
#include <algorithm>    // std::make_heap, std::pop_heap, std::push_heap, std::sort_heap
#include <vector>       // std::vector

int main () {
  int myints[] = {10,20,30,5,15};
  std::vector<int> v(myints,myints+5);

  std::make_heap (v.begin(),v.end());
  std::cout << "initial max heap   : " << v.front() << '\n';

  std::pop_heap (v.begin(),v.end()); v.pop_back();
  std::cout << "max heap after pop : " << v.front() << '\n';

  v.push_back(99); std::push_heap (v.begin(),v.end());
  std::cout << "max heap after push: " << v.front() << '\n';

  std::sort_heap (v.begin(),v.end());

  std::cout << "final sorted range :";
  for (unsigned i=0; i<v.size(); i++)
    std::cout << ' ' << v[i];

  std::cout << '\n';

  return 0;
}
```

输出结果：
```
nitial max heap   : 30
max heap after pop : 20
max heap after push: 99
final sorted range : 5 10 15 20 99
```
