---
layout: post
title:  "浅谈数据库的索引"
tag:
- MySQL
- 数据库
comments: true
---

## 索引

在学校的课程学习中，对一些基础课程的理解往往是后知后觉的，比如去年修的数据库原理这门课。如今重新对数据库进行学习，才发觉以前对某些概念的理解极其空洞，其中之一便是**索引(index)**。

在数据库中，索引的含义与日常意义上的“索引”一词并没有太大差别，它避免了全表扫描，从而极大地提高数据表的访问速度。索引主要利用B/B+树来实现，又可细分为**聚簇索引(clustered index)**和**非聚簇索引(unclustered index)**。

- 聚簇索引：叶节点就是数据节点，索引中键值的逻辑顺序决定了表中相应行的物理顺序，也就是说，所有数据行的存储顺序与索引的存储顺序一致。因为B/B+树本身是排序的，所以建立了聚簇索引的数据表也是自动有序的，而一个数据表的数据排列只能有一种物理顺序，所以也只能有一个聚簇索引。
![](https://controny.github.io/assets/images/posts/20180226171444.png)
- 非聚簇索引：与聚簇索引不同，其叶节点仍然是索引节点，只不过有一个指针指向对应的数据块，所以该索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同。在这样的结构下，新插入的元素统统会被置于数据末端，因此数据不是自动有序的。
![](https://controny.github.io/assets/images/posts/20180226171518.png)

如果把利用索引查找数据类比为查字典的话，那么利用聚簇索引查找就是“根据拼音查字”，而利用非聚簇索引查找则是“根据偏旁部首查字”，有人正是这样作出解释的：
>其实，我们的汉语字典的正文本身就是一个聚集索引。比如，我们要查“安”字，就会很自然地翻开字典的前几页，因为“安”的拼音是“an”，而按照拼音排序汉字的字典是以英文字母“a”开头并以“z”结尾的，那么“安”字就自然地排在字典的前部。如果您翻完了所有以“a”开头的部分仍然找不到这个字，那么就说明您的字典中没有这个字；同样的，如果查“张”字，那您也会将您的字典翻到最后部分，因为“张”的拼音是“zhang”。也就是说，字典的正文部分本身就是一个目录，您不需要再去查其他目录来找到您需要找的内容。我们把这种正文内容本身就是一种按照一定规则排列的目录称为“聚集索引”。
    　　如果您认识某个字，您可以快速地从自动中查到这个字。但您也可能会遇到您不认识的字，不知道它的发音，这时候，您就不能按照刚才的方法找到您要查的字，而需要去根据“偏旁部首”查到您要找的字，然后根据这个字后的页码直接翻到某页来找到您要找的字。但您结合“部首目录”和“检字表”而查到的字的排序并不是真正的正文的排序方法，比如您查“张”字，我们可以看到在查部首之后的检字表中“张”的页码是672页，检字表中“张”的上面是“驰”字，但页码却是63页，“张”的下面是“弩”字，页面是390页。很显然，这些字并不是真正的分别位于“张”字的上下方，现在您看到的连续的“驰、张、弩”三字实际上就是他们在非聚集索引中的排序，是字典正文中的字在非聚集索引中的映射。我们可以通过这种方式来找到您所需要的字，但它需要两个过程，先找到目录中的结果，然后再翻到您所需要的页码。我们把这种目录纯粹是目录，正文纯粹是正文的排序方式称为“非聚集索引”。

## MySQL中的索引

MySQL目前常用的存储引擎有InnoDB和MyISAM，两者在索引的使用上是有差异的:

* InnoDB会以下列规则默认建立聚簇索引:
	- 如果一个主键被定义了，那么这个主键就作为聚集索引。
	- 如果没有主键被定义，那么该表的第一个唯一非空索引被作为聚集索引。
	- 如果没有主键也没有合适的唯一索引，那么InnoDB内部会生成一个隐藏的主键作为聚集索引，这个隐藏的主键是一个6个字节的列，改列的值会随着数据的插入自增。

除此之外，其余用户建立的索引都属于非聚簇索引，又属于**次级索引(secondary index)**。次级索引存储的是主键的值，叶节点并不存储行数据的物理地址，而是存储该行的主键值，因此，换句话说，主键扮演了行数据的指针。由此可作出如下结论，次级索引包含了两次查找：先查找次级索引自身，然后查找主键（聚集索引）。

* MyISAM只支持非聚簇索引。

下面就实际测试一番。如图，我建立了一个InnoDB数据表并以`id`为主键：
![](https://controny.github.io/assets/images/posts/20180226174258.png)
现在，数据表中有了两行数据：
![](https://controny.github.io/assets/images/posts/20180226175154.png)
当我插入`id`为6的数据行时，它会被插入到第二行：
![](https://controny.github.io/assets/images/posts/20180226175242.png)
这说明，该表是以`id`这个主键作为聚簇索引的，因此自动按照`id`的顺序排列。

现在，我把表的引擎变更为MyISAM：
![](https://controny.github.io/assets/images/posts/20180226175542.png)
再插入`id`为4的数据行，结果却没有按顺序排列，而只是粗暴地把新数据置于最后：
![](https://controny.github.io/assets/images/posts/20180226175706.png)
这说明，该表中已没有了聚簇索引。

可进一步通过下面这张示意图理解两种表的索引结构：
![](https://controny.github.io/assets/images/posts/20180226180605.png)

## 参考：
- <http://www.cnblogs.com/aspnethot/articles/1504082.html>
- <https://www.cnblogs.com/morvenhuang/archive/2009/03/30/1425534.html>
- <https://www.jianshu.com/p/2879225ba243>
- <https://www.linuxidc.com/Linux/2018-02/150809.htm>