---
layout: post
title:  "[LeetCode] Subsets"
tag:
- Java
- LeetCode
- 算法
comments: true
---

题目描述：<https://leetcode.com/problems/subsets/description/>

比起回溯法，我的想法就看起来朴素很多：依次对每个元素都问一句，拿，还是不拿？

以`[1, 2, 3]`为例。

1. 以空集合开始：
```
[]
```
2. 遇到`1`，可以选择把它拿进来或者不拿进来：
```
[]      // 不拿1
[1]     // 拿1
```
3. 遇到`2`，对已有的子集合继续做抉择：
```
[]      // 不拿1
[1]     // 拿1
[2]     // 不拿1，拿2
[1, 2]  // 拿1，拿2
```
4. 遇到`3`，重复上述步骤：
```
[]
[1]
[2]
[1, 2]
[3]
[1, 3]
[2, 3]
[1, 2, 3]
```

然后不需要用递归也能很容易写出代码：
```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        // start with an empty set
        result.add(new ArrayList());
        for (int i = 0; i < nums.length; i++) {
            // copy every subsets and add nums[i] to each of them
            int size = result.size();
            for (int j = 0; j < size; j++) {
                List<Integer> subset = new ArrayList(result.get(j));
                subset.add(nums[i]);
                // System.out.println("add: " + nums[i]);
                result.add(subset);
            }
        }
        return result;
    }
}
```
