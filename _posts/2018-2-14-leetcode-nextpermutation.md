---
layout: post
title:  "[LeetCode] Next Permutation —— 找规律解题"
tag:
- C++
- LeetCode
comments: true
---

题目描述：<https://leetcode.com/problems/next-permutation/description/>

这种题目的特点是，乍看上去毫无头绪，但可通过举例子找出其中的规律，从而发现突破口。

比如，我们试着写出`1, 2, 3, 4, 5`的排列：

- 1 2 3 *4* **5**
- 1 2 *3* 5 **4**
- 1 2 4 *3* **5**
- 1 2 *4* **5** 3
- 1 2 5 *3* **4**
- ......

其中，斜体代表“从左往右数第一个即将发生变化的位置”，不妨用变量`flipPos`来表示；粗体代表“将与`flipPos`交换的位置。

我们可以总结出以下规律：

1. 从右往左看，斜体总是第一个数值变小的位置，也就是说，斜体右边的数字都是按照**降序**排列好的。
2. 粗体数字是斜体右边的数中比斜体数字大的最小值，即，从右往左看，第一个比斜体数字大的位置。
3. 斜体与粗体的数字交换后，原斜体位置的右边应按照**升序**排列好。

有了上述规律，我们很容易把代码实现：

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        if (nums.empty() || nums.size() == 1)
            return;
        int flipPos = -1;
        // elements after `flipPos` would be in descending order
        for (int i = nums.size()-2; i >= 0; i--) {
            if (nums[i] < nums[i+1]) {
                flipPos = i;
                break;
            }
        }
        if (flipPos == -1) {
            sort(nums.begin(), nums.end());
            return;
        }

        for (int i = nums.size()-1; i > flipPos; i--) {
            if (nums[i] > nums[flipPos]) {
                swap(nums[i], nums[flipPos]);
                reverse(nums.begin()+flipPos+1, nums.end());
                return;
            }
        }
    }
};
```

