---
layout: post
title:  "关于STL的sort实现"
tag:
- C++
- STL
- 算法
comments: true
---

原来始终以为作为C++标准库的STL`sort`函数使用的只是纯粹的Quick Sort，没想到这其实是STL算法库中最为复杂的一个：以Quick Sort为主，Heap Sort、Insertion Sort为辅，无所不用其极地优化使用效率。

## Quick Sort
Quick Sort是目前已知最快的排序算法，平均复杂度为O(NlogN)：平均情况下，可假设每次分割都把数组一分为二，则总共需要进行logN次分割，每次分割耗费O(N)复杂度，所以可以得出该结果。

但最坏情况下，算法的复杂度是O(N^2)：最坏情况下，每次选择的pivot（枢轴）都为最大值或最小值，这样每次只把一个元素分隔开，于是总共需要进行N次分割。

所以pivot的选取就显得尤为关键，STL的实现采用**Median-of-Three**的原则：取出头、尾、中央三个位置的元素，以其中值（median）作为pivot，如序列`{1, 3, 4, 5, 2}`，头、尾、中央三个位置的元素分别为1、2、4，那么就选择中间值2作为pivot。

## Heap Sort
但是上述原则只能在统计学上尽可能避免最坏情况，仍旧无法完全回避它。所以STL的解决办法是，当算法递归层次过深（即分割次数过多）时，就断定算法有向最坏情况演变的倾向，因而转而采用Heap Sort——这种结合了Quick Sort和Heap Sort的算法又被称为**Introspective Sort**（introspective有内省之意，意思是该算法有“自我反省”的意味），简称**IntroSort**。

不过这样的设定也不免令人疑惑：为什么不一开始就使用Heap Sort？毕竟Heap Sort的最坏情况和平均情况复杂度都是O(NlogN)。可以这么来解释，排序算法最耗费时间大概要属“交换两个元素的位置”了，但是Heap Sort无论什么情况下都得把所有元素交换过一次（想想把max-heap的最大值持续往后移动的排序过程），这一特点在应对糟糕情况时有优势，面对较优的情况则成了劣势。而Quick Sort就胜在它的最快情况：在比较好的情况下可以不用怎么交换元素，参考<https://stackoverflow.com/questions/2467751/quicksort-vs-heapsort>。

## Insertion Sort
当然STL做的还不止以上这些，考虑到Insertion Sort虽然平均复杂度为O(N^2)，但在面对“几近排序但未完成”的序列时有着极高的效率，所以，STL的sort还会判断某个子序列的个数是否小于提前设定的阈值（默认是16），若是，则直接采用Insertion Sort。
