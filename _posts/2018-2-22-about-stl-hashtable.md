---
layout: post
title:  "关于STL的hash table"
tag:
- C++
- STL
- 算法
comments: true
---

- C++11以后才正式将hash table相关的容器纳入标准库中，命名为`unordered_set`、`unordered_map`等，这是为了与非标准的`hash_set`、`hash_map`等区分开，但两者的用法和实现大同小异。

- 采用**separate chaining**解决碰撞，即为每个bucket（表格的单元）维护一个list。
![](https://controny.github.io/assets/images/posts/20180222183451.png)
其中的`buckets vector`就以STL的`vector`来实现。

- 以质数来设计表格大小：预先计算好一定数量的质数，同时提供一个函数查询这些质数中“最接近某数且大于某数的质数”；初始化时，指定所需的大小`n`，构造出的表格大小为一个接近`n`且大于`n`的质数。

- 插入一个元素时，若**load factor**大到一定程度，则将表格大小扩大为一个更大的质数（基本是原来的两倍），然后重新安排所有元素的位置。这么做的目的是控制表格的**load factor**不至于过大，以维护算法的效率。测试代码如下：

```cpp
#include <unordered_set>
#include <iostream>

using namespace std;
int main(void) {
    unordered_set<int> s;
    cout << "bucket count: " << s.bucket_count() << endl;
    for (int i = 0; i < 10; i++) {
        cout << "insert: " << i << endl;
        s.insert(i);
        cout << "bucket count: " << s.bucket_count() << endl;
        cout << "load factor: " << s.load_factor() << endl;
    }
    return 0;
}
```

运行结果为：
![](https://controny.github.io/assets/images/posts/20180223141716.png)
可以看到，当**load factor**接近1时，表格大小（`bucket count`）扩大至下一个约为原来两倍的质数。
