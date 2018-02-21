---
layout: post
title:  "[LeetCode] Longest Substring Without Repeating Characters——Sliding Window思想"
tag:
- C++
- LeetCode
- 算法
comments: true
---

题目描述：<https://leetcode.com/problems/longest-substring-without-repeating-characters/description/>。

看到题目时，没什么特别的想法，只想到了比最糟糕的Brute Force稍微好一点的算法：分别“以每个字符为开端”，逐步向后遍历，遇到有重复出现的就跳到“以下一个字符为开端”的循环。其中用数组标记各个字符是否出现过以判断字符重复。

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int ans = 0;
        
        // Naive method
        // Use flags to judge whether a ascii character has appeared
        char flags[128];
        for (int i = 0; i < s.length(); i++) {
            int pos = i, count = 0;
            memset(flags, 0, sizeof(flags));
            int index;
            while (pos < s.length() && !flags[ index = s[pos]]) {
                flags[index] = 1;
                count++;
                pos++;
            }
            if (count > ans)
                ans = count;
        }


        return ans;
    }
};
```

该算法时间复杂度为O(n^2)，比最糟糕的O(n^3)要好。题目设置的难度不大，所以这样的方法也可以成功accept。

但追求卓越总是人的天性。仔细想想，题目只要求得到最大长度，并不要求把所有长度最大的子字符串输出出来，所以上述算法其实作了许多无用功。后来了解到可以用一种叫做Sliding window的抽象思维来解决问题，即把一个子字符串看做一个window，通过这个window的伸缩和移动来求得最大值。

这么说起来确实还挺抽象的，下面就先看看具体的代码吧：

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int ans = 0;

        // Improved with `sliding window'
        int lastPos[128];   // used to record the last position of each character
        memset(lastPos, -1, sizeof(lastPos));
        for (int i = 0, j = 0; j < s.length(); j++) {
            if (lastPos[ s[j] ] >= i) {
                i = lastPos[ s[j] ] + 1;
            }
            ans = max(ans, j - i + 1);
            lastPos[ s[j] ] = j;
        }
        
        return ans;
    }
};
```

`lastPos`数组用来记录每个字符上一次出现在字符串中的位置，均初始化为-1,。由于测试数据的字符串仅包括128个ASCII字符，所以把数组大小设置为128完全够用。

然后我们会用到两个辅助的标记变量`i`和`j`，分别表示window的左边和右边。注意for循环是以`j`作迭代的，也就是每次都令window的右边向前伸展。每次循环都会更新最佳解`ans`，以及`lastPos`中对应字符的位置信息（`lastPos[ s[j] ]`），当前字符上次出现的位置（`lastPos[ s[j] ]`）大于或等于`i`时，就表示window中有重复的字符（很容易想象），这时我们直接令`i`跳到前者的下一个位置（`lastPos[ s[j] ] + 1`）。这是因为，就算我们逐步增大`i`，window的长度都不可能超过遇到重复字符之前的状态，因而不可能是最佳解，所以我们干脆略过，为了搜索下一个无重复的子字符串，`i`可被设为`lastPos[ s[j] ] + 1`。
