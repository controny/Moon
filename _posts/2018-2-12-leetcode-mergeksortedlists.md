---
layout: post
title:  "[LeetCode] Merge k Sorted Lists——巧用Heap"
tag:
- C++
- LeetCode
- 算法
comments: true
---

题目描述：<https://leetcode.com/problems/merge-k-sorted-lists/description/>

面对这道题，最朴素的算法是每次都纵向遍历取得最小值，假设每个list的平均大小是`N`，则时间复杂度为**O(NK^2)**。LeetCode设置的难度都不会很大，我试了一下，对于一道Hard的题，虽然时间很难看，但这样竟也能AC。

后来领悟到可以用Heap来优化时间，才想起上学期的算法课老师经常赞叹Heap的巧妙：每次仅用极小的代价就可以得出最小（大）值。这个代价只有O(log(n))，因此针对这道题的算法就可以优化到**O(NKlogK)**。

在C++标准库中，有Heap相关的函数可以直接使用，我用到的有[`make_heap`](http://www.cplusplus.com/reference/algorithm/make_heap/)、[`pop_heap`](http://www.cplusplus.com/reference/algorithm/pop_heap/)、[`push_heap`](http://www.cplusplus.com/reference/algorithm/push_heap/)。具体代码及相关解释如下：

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        if (lists.empty())
            return nullptr;
        ListNode* toAdd, * head = nullptr, * current;   
        make_heap(lists.begin(), lists.end(), heapComp);
        while (true) {
            // since we regard null pointer as the 'biggest'
            // when the front is null, the subsequent elements are all null
            if (lists.front() == nullptr)
                break;
            toAdd = lists.front();
            // add it to the result
            if (!head) {
                head = toAdd;
            } else {
                current->next = toAdd;
            }
            current = toAdd;
            // pop and push
            pop_heap(lists.begin(), lists.end(), heapComp);
            lists.pop_back();
            lists.push_back(toAdd->next);
            push_heap(lists.begin(), lists.end(), heapComp);
        }
        return head;
    }

private:
    static bool heapComp(ListNode* lhs, ListNode* rhs) {
        // regard null pointer as the biggest
        if (lhs == nullptr)
            return true;
        else if (rhs == nullptr)
            return false;
        else
            return lhs->val > rhs->val;
    }
};
```

* **15**: 将`lists`建立成一个Heap，注意我们需要传递自定义的比较函数`heapComp`，因为我们需要令Heap的首元素是`val`最小的元素。
* **19-20**: 循环的终止条件显然是所有的链表指针都到达了末尾，而后面会看到，由于我们默默地把空指针当做了“最大值”，当`lists.front()`为空指针时，后面的元素也一定都是空指针，因此可以结束循环。
* **30-33**: 更新Heap时，如果每次都调用`make_heap`重新建Heap，消耗的时间是O(K)，结果会超时，反而比朴素算法还糟糕。因此我们采用先pop后push的做法，`pop_heap`和`push_heap`的复杂度均为O(logK)，才使得总的时间复杂度得到了优化。
* **39-47**: 自定义比较函数，我们把空指针当做最大值，因此当`lhs`为空指针时，`lhs`比`rhs`大，应当返回`true`，后面的条件可以此类推。

