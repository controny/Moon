---
layout: post
title:  "[LeetCode] Edit Distance"
tag:
- C++
- LeetCode
- 算法
comments: true
---

题目描述：<https://leetcode.com/problems/edit-distance/description/>

又是一道用动态规划解的题。

## 基本设定

用`p[i][j]`表示`word1[0...i-1]`转化为`word2[0...j-1]`所需的最小步数，注意右边是开区间，不包括`word1[i]`和`word2[j]`。

## 边界条件

动态规划总要确定一个边界条件，在这里，边界条件是“将一个字符串（空或非空）转化为空字符串，以及反过来”，而所需的最小步数就是原字符串的长度。显然，就是`i`或`j`为0时:

- `p[i][0] = i`
- `p[0][j] = j`

## 一般情况

一般情况就是，已知`p[i-1][j]`、`p[i][j-1]`、`p[i-1][j-1]`等，求`p[i][j]`。当`word1[i-1] == word2[j-1]`时，不需要额外的变换，因此`p[i][j] = p[i-1][j-1]`。否则，根据题目的意思，这时有三种策略可以选择：

- **改变**
这是在“将`word1[0...i-2]`转化为`word2[0...j-2]`”的基础上多增加一个步骤，即`p[i][j] = p[i-1][j-1]+1`。
- **插入**
考虑如下情况：
    * `word1[0...i-1]`等于`"abc"`
    * `word2[0...j-1]`等于`"bbcd"`
此时我们处在`word1[i-1]`和`word2[j-1]`的位置，要在`word1[i-1]`后插入`'d'`，而在此之前，我们需要知道如何将`word1[0...i-1]`转化为`word2[0...j-2]`，即将`"abc"`转化为`"bbc"`，然后在此基础上增加一个插入的步骤，所以`p[i][j] = p[i][j-1]+1`。
- **删除**
这种情况的分析与插入大致类似，这里便不再赘述：`p[i][j] = p[i-1][j]+1`。

分析完了所有的策略，那么如何确定什么情况下采用哪种策略呢？答案是，我们无法对此进行准确的分类，直截了当的办法是，选择最小值。

## 代码

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        if (!m || !n)
            return max(m, n);
        // `table[i][j]` denote the distance between 
        // `word1[0...i-1]` and `word2[0...j-1]`
        vector<vector<int>> table(m+1, vector<int>(n+1));
        // boundary case: convert a string to an empty one and vice versa
        for (int i = 0; i <= m; i++)
            table[i][0] = i;
        for (int j = 0; j <= n; j++)
            table[0][j] = j;

        // general case
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++) {
                if (word1[i-1] == word2[j-1])
                    table[i][j] = table[i-1][j-1];
                else
                    table[i][j] = min(
                        table[i-1][j-1]+1,      // change
                        min( 
                            table[i-1][j]+1,    // delete
                            table[i][j-1]+1     // insert
                        ));
            }
        return table[m][n];
    }
};
```

参考：<https://leetcode.com/problems/edit-distance/discuss/25846/20ms-Detailed-Explained-C++-Solutions-(O(n)-Space)>