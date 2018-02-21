---
layout: post
title:  "基于二分法和剪枝搜索的二维数组查找"
tag:
- C++
- 算法
comments: true
---

## 概述

《剑指offer》一书中的编程题纳入了牛客网的在线编程题库中，其中有一题是[二维数组中的查找](https://www.nowcoder.com/practice/abc3fe2ce8e146608e868a70efebf62e?tpId=13&tqId=11154&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)。

讨论区中讨论的算法主要有两种：

- 从左下搜索，遇大上移，遇小右移，时间复杂度为O(m+n)。
- 每行都进行二分查找，时间复杂度O为(mlogn)。

而我要介绍的是一种基于**二分查找**和**剪枝搜索(prune and search)**思想的算法，时间复杂度为O( log(mn)/log(4/3) )，比上述两种算法更出色。

## 基本思路

数组的行和列都是按照升序排列好的，因此很自然想到二分法，找到二维数组最中间的那个元素并设其为`a`：

![](https://controny.github.io/assets/images/posts/20180221125719.png)

根据数组排列的特点，我们可以把数组划分为几个区域：

![](https://controny.github.io/assets/images/posts/20180221130757.png)

其中，A区表示“小于`a`”的区域，B区表示“大于`a`”的区域，C区则是与`a`大小关系未知的区域。

假设搜寻目标`target`大于`a`，那么，我们显然没有必要在A区中寻找答案，因此可以把A区“剪”掉，只对剩余有希望的区域进行下一步的搜寻，这便是**剪枝搜索**的思想。

所以下一步可以把BC区域分成两个二维数组，分别进行递归的搜索：

![](https://controny.github.io/assets/images/posts/20180221132239.png)

当`target`小于`a`时，可以此类推。

## 时间复杂度分析

每一次递归，我们都会把问题的规模大致缩减为原来的3/4：第一次为`3/4mn`，第二次为`(3/4)^2mn`，第三次为`(3/4)^3mn`......

假设一共需要进行`k`次递归，则终止条件是![](https://controny.github.io/assets/images/posts/20180221132928.png)，解得![](https://controny.github.io/assets/images/posts/20180221133837.png)。

所以算法的时间复杂度为O( log(mn)/log(4/3) )。

乍看上去，这好像没什么了不起，但是，由于对数函数增长极其缓慢，随着问题规模成千上万的扩大，该算法的优越性就显现出来了。如，当m = 10000，n = 10000时，上述两种普通算法的时间复杂度分别为O(20000)，O(133000)，而我的算法仅有O(64)的时间复杂度。

## 代码

```cpp
class Solution {
public:
    bool Find(int target, vector<vector<int> > array) {
        int m = array.size(), n = array[0].size();
        return search2D(0, n-1, 0, m-1, array, target);
    }

private:
    bool search2D(int left, int right, int up, int down, vector<vector<int> >& array, int target) {
        if (left > right || up > down
            || right >= array[0].size() || down >= array.size())
            return false;
        int x = (down-up)/2+up;
        int y = (right-left)/2+left;
        //cout << "x: " << x << " y: " << y << endl;
        if (array[x][y] == target)
            return true;
        else if (array[x][y] > target)   // skip those bigger than target
            return search2D(left, right, up, x-1, array, target)
                || search2D(left, y-1, x, down, array, target);
        else
            return search2D(y+1, right, up, x, array, target)
                || search2D(left, right, x+1, down, array, target);
    }
};
```

