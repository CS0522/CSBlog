向量检索

欧式距离、余弦距离、内积、投影...

避免全量数据匹配（当向量数量、维度特别大，计算量不可估计）
近似最近邻，以最小的计算量获取更可靠的结果

在业界，ANN算法常常被分为四个大类：

基于树结构的
基于Hash的
基于量化 (Quantization) 的
基于图 (Graph) 的
目前一个普遍的看法是，基于图的算法一般要优于其它三种类型的算法，表现在查询的速度和召回率上面。

在基于图的ANN算法中，如何构造图的结构至关重要，更准确一些来说的话，是边的构造至关重要。在后面的章节中，本文会针对一些图算法和结构，例如Delaunay Graph，Monotonic Search Network，Relative Neighborhood Graphs等进行深入的探讨。


单调图 (Monotonic Graph) 或者单调搜索网络 (Monotonic Search Network)：对于图 
 上的一条路径$V_{1}, V_{2}, V_{3}, ..., V_{n}$
 ，如果对任意$i$
 有$Dist(V_{i}, V_{n}) > Dist(V_{i}, V_{n})$
 ，其中 $Dist()$
 表示两点之间的距离（实际距离），那么这是一条单调路径。如果对于图上的任意两点$U$
 和$V$
 ，总存在至少一条单调路径，那么$G$是一个单调图。单调图也被称为单调搜索网络，即Monotonic Search Network。


 ## 单调搜索网络 (Monotonic Search Network)

 https://zhuanlan.zhihu.com/p/610454162