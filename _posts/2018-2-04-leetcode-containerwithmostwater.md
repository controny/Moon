---
layout: post
title:  "[LeetCode] Container With Most Water——贪心算法"
tag:
- C++
- LeetCode
comments: true
---

题目描述：<https://leetcode.com/problems/container-with-most-water/description/>

题目虽然只是Medium难度，但算法实在很难想出来，看了看解答，意识到最佳算法是贪心算法。

## 算法

算法的思路非常简单。我们用两个变量`left`和`right`分别表示容器的左右两端并初始化为最左边的线和最右边的线，然后，每一次迭代都**令短的一端向对方移动一格**（如`left`比`right`短，就令`left`向右移动一格），直到两者相遇。

下面是动态的演示图：

![](https://leetcode.com/media/original_images/11_Container_Water.gif)

## 实现

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int ans = 0;
        int left = 0, right = height.size()-1;
        while (left < right) {
            ans = max( ans, min(height[left], height[right])*(right-left) );
            if (height[left] > height[right])
                right--;
            else
                left++;
        }
        return ans;
    }
};
```

## 理解

算法很神奇，乍一看也颇令人费解，不过我们可以以贪心算法的角度来理解它。根据维基百科，

>(贪心算法)是一种在每一步选择中都采取在当前状态下最好或最优（即最有利）的选择，从而希望导致结果是最好或最优的算法。

就本题而言，每一步的最佳选择必定不是**令长的一端向短的一端移动**，因为所得矩形的面积只与短边的长度及两端的距离有关，如果我们保留着短边，就只会带来更短的宽度，导致更小的面积。而如果是**令短的一端向长的一端移动**，决定矩形高度的“短板”就有望获得提升，以抵消宽度减小的副作用，从而有机会使矩形面积增大。如此分析，后者自然是每一步的最佳选择。

当然，以上只是直观上的理解，严谨的证明可以利用反证法来完成，参考<https://discuss.leetcode.com/topic/503/anyone-who-has-a-o-n-algorithm/2>
