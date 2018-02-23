---
layout: post
title:  "[LeetCode] Unique Paths"
tag:
- C++
- LeetCode
- 算法
comments: true
---

题目描述：<https://leetcode.com/problems/unique-paths/description/>

## 普通版

很容易想到动态规划：用`numPaths[i][j]`表示(i, j)到终点的路线数，则每个`numPaths[i][j]`都等于其右方的值与下方的值之和（在界外的都视为0）；令右下角终点处的值`numPaths[m-1][n-1]`等于1，从右下往左上迭代，`numPaths[0][0]`就是最终的解。

写出来的代码也非常简短：

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> numPaths(m, vector<int>(n));
        for (int i = m-1; i >= 0; i--)
            for (int j = n-1; j >= 0; j--) {
                if (i == m-1 && j == n-1)
                    numPaths[i][j] = 1;
                else
                    numPaths[i][j] = 
                        ( i+1 < m ? numPaths[i+1][j] : 0 ) +
                        ( j+1 < n ? numPaths[i][j+1] : 0 );
            }
        return numPaths[0][0];
    }
};
```

## 优化版

上述算法已经很简洁了，但在空间效率上仍有优化的余地。让我们仔细回顾一下上述二维数组`numPaths`的求值过程，不妨假设`m` = 3，`n` = 4，数组元素间的依赖关系可用箭头表示，如图：

![](https://controny.github.io/assets/images/posts/20180223190443.png)

如果我们以“列”的视角来观察这个矩阵，可以发现除了最右边的那一列以外，其余每一列的值只依赖于它右边那一列和自身，比如最左边的`(10, 4, 1)`就只依赖于右边的`(6, 3, 1)`和自身：

![](https://controny.github.io/assets/images/posts/20180223191021.png)

然而，我们完全可以把`(6, 3, 1)`当作是`(10, 4, 1)`的“前身”，也就是说，每一列都只是自身演变的结果。这意味着我们完全可以用一维数组完成动态规划，如果`numPaths`现在表示每一列的值，则上述思想用代码表示就是：`numPaths[i] += numPaths[i+1]`。现在，空间复杂度就由O(mn)优化为了O(m)，代码也更为简洁了：

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<int> numPaths(m, 1);
        for (int j = n-2; j >= 0; j--)
            for (int i = m-2; i >= 0; i--)
                numPaths[i] += numPaths[i+1];
        return numPaths[0];
    }
};
```
