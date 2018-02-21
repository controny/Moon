---
layout: post
title:  "[LeetCode] Longest Palindromic Substring——动态规划解题"
tag:
- C++
- LeetCode
- 算法
comments: true
---

题目描述：<https://leetcode.com/problems/longest-palindromic-substring/description/>

题目的意思很简单，就是在一个给定的字符串中找出最长的回文（palindromic）子字符串。用一般的思路貌似也可以解，不过为了练习，我还是决定用动态规划来解题。

用动态规划解题的思路是，先以bottom-up的递归思维思考，再以top-down的形式来实现。

所以我们首先要列出递归式。用i和j分别表示一个回文子字符串左右边界的索引号，p(i,j)表示该回文子字符串的长度：

![](https://controny.github.io/assets/images/posts/20180202111834.png)

各情况的解释如下：

* i == j时，显然该字符串是长度为1的回文字符串。
* i+1 == j且s[i] == s[j]时，字符串包含两个相同的字符（如**aa**），所以是长度为2的回文字符串。
* p(i+1,j-1) > 0意味着`s[i+1]...s[j-1]`是一个回文字符串，此时如果s[i] == s[j]，那么该字符串也是回文字符串，其长度等于上述回文字符串加2。
* 其他情况下，`s[i]...s[j]`不是回文字符串，因此长度为0。

实现的代码如下：

```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        // 字符串为空或只含一个字符时，可将字符串直接返回
        if (s.empty() || s.length() == 1)
            return s;

        // 为动态规划所需的表格申请内存空间
        int len = s.length();
        int** table = new int*[len];
        for (int i = 0; i < len; i++)
            table[i] = new int[len];

        int maxLen = 0, start = 0;
        // 由于以下将引用第`i+1`行的元素，因此注意`i`要从大到小迭代
        // 同时还要保证`i <= j`，所以`j`的初始值是`i`
        for (int i = len-1; i >= 0; i--) {
            for (int j = i; j < len; j++) {
                if (i == j)
                    table[i][j] = 1;
                else if (j == i+1 && s[j] == s[i])
                    table[i][j] = 2;
                else if (table[i+1][j-1] && s[i] == s[j])
                    table[i][j] = table[i+1][j-1] + 2;
                else
                    table[i][j] = 0;

                // 每次都更新最大长度`maxLen`以及该子字符串的起始位置`start`
                if (table[i][j] > maxLen) {
                    maxLen = table[i][j];
                    start = i;
                }
            }
        }
        
        return s.substr(start, maxLen);
    }
};
```
