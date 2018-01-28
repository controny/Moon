---
layout: post
title:  "[LeetCode] Two Sum——巧用哈希解题"
tag:
- C++
- LeetCode
comments: true
---

LeetCode的Two Sum是一道easy题，题目描述为<https://leetcode.com/problems/two-sum/description/>。

很容易想到用暴力破解法（Brute Force）解题：

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        for (int i = 0; i < nums.size(); i++)
             for (int j = 0; j < nums.size(); j++)
                 if (i == j)
                     continue;
                 else
                     if (nums[i] + nums[j] == target)
                         return vector<int>( {i, j} );
    }
};
```

这样的时间复杂度是O(n^2)，虽然丑陋，不过也确实可以Accepted。但是总感觉一定能有巧妙的方法提高效率。果不其然，看了解答后，知道可以用哈希表来改善时间复杂度。

大概思路就是，每次扫描一个数字的时候，判断它的补数（即`target - nums[i]`）是不是在哈希表里，如果是，那么显然这两个数就是我们要找的答案；否则把这个数字放入哈希表中。由于哈希表读取和插入元素的时间复杂度都可看成O(1)，整个算法也只需要把数字扫描一遍，因此时间复杂度可提升至O(n)。

具体代码如下：

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> map;
        for (int i = 0; i < nums.size(); i++) {
            auto complement = target - nums[i];
            auto it = map.find(complement);
            if (it != map.end())
                return vector<int>( {it->second, i} );
            map.insert( pair<int, int>(nums[i], i) );
        }
    }
};
```

值得注意的是，C++自C++11后才将哈希表纳入标准库，为了跟以前就存在的非标准`hash_map`相区分，C++标准库中的哈希表命名为`unordered_map`。因此该哈希表实际上是一个map，为了便于查找数字，在我的代码中，map的key是nums的元素，value才是其中的索引号。

以前只是懵懵懂懂地知道哈希表的实现原理，但对它的实际应用体会并不深刻，通过这道题，我总算明白了哈希表的价值所在。