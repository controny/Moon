---
layout: post
title:  "[LeetCode] 3 Sum"
tag:
- C++
- LeetCode
- 算法
comments: true
---

题目描述：<https://leetcode.com/problems/3sum/description/>

这道题属于Medium，然而通过率只有20出头，比很多Hard题目还低。苦思了许久还是无法攻克，只好去Discuss中寻找灵感......

先上最终的代码：

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> ans;
        for (int i = 0; i < nums.size(); i++) {
            // if `nums[i]` is positive, there is no need to iterate
            if (nums[i] > 0)
                break;
            
            int target = -nums[i];
            int front = i+1, back = nums.size()-1;
            
            // find solution starting from `nums[i]`
            while (front < back) {
                int sum = nums[front] + nums[back];
                if (sum < target)
                    front++;
                else if (sum > target)
                    back--;
                else {
                    // the whole sum is 0
                    vector<int> triplet{nums[i], nums[front], nums[back]};
                    ans.push_back(triplet);

                    // skip duplicate of number 2 and number 3
                    while (front < back && nums[front] == triplet[1])
                        front++;
                    while (front < back && nums[back] == triplet[2])
                        back--;
                }
            }
            // skip duplicate of number 1
            while (i < nums.size() && nums[i+1] == nums[i])
                i++;
        }
        return ans;
    }
};
```
解释一下：
* **4**: 首先要对数组排序，这是为了便于后续的搜索和去重。
* **6**: 分别以每个数(`nums[i]`)为首，向后搜寻以找出所有可能的组合。
* **8-9**: 注意当`nums[i]`大于0时循环即可结束，因为往后的数字必定大于0，从而以`nums[i]`为首的组合之和必定大于0，所以均可跳过。
* **11**: 寻找的目标是后两个数之和为`nums[i]`的相反数。
* **12**: 以两个变量`front`、`back`分别标记后两个数的位置，并分别初始化为`i`的后一位和最后一位。
* **17-20**: 根据后两个数之和与`target`（`nums[i]`的相反数）的大小关系调整`front`或`back`。这样最终一定会找到所有符合要求的组合，可用反证法证明之。
* **23-24**: 找到符合要求的组合，加入答案集合中。
* **27-30**: 跳过相同的数以过滤重复的组合。
* **34-35**: 同样要令第一个数跳过相同的。

理解了上述的解题思路，LeetCode上其他类似的题如[3 Sum Closest](https://leetcode.com/problems/3sum-closest/description/)、[4 Sum](https://leetcode.com/problems/4sum/description/)也迎刃而解了。

