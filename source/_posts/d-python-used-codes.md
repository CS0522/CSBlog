---
title: 【文档小记】记录 Python 中使用过的有用代码
tags:
  - Python
  - matplotlib
toc: true
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2024-04-03 00:55:17
---

记录 Python 中使用过的有用代码，以后应该还用的上。

<!-- more -->

## 平滑对比曲线图

```python
import matplotlib.pyplot as plt
# 平滑曲线图
from scipy.interpolate import make_interp_spline
import numpy as np

def comparison_line_chart(self):
    x1 = [0.1, 0.2, 0.5, 0.6, 0.8, 0.9]
    x2 = [0.2, 0.3, 0.4, 0.7, 0.8, 0.9, 1]
    y1 = [1, 3, 4, 5, 7, 9]
    y2 = [2, 4, 5, 6, 7, 8, 9]
    # list 转换为 array
    # 需要 min()，max() 函数
    x1_array = np.array(x1)
    x2_array = np.array(x2)
    y1_array = np.array(y1)
    y2_array = np.array(y2)
    # 平滑
    x1_smooth = np.linspace(x1_array.min(), x1_array.max(), 300)
    x2_smooth = np.linspace(x2_array.min(), x2_array.max(), 300)
    y1_smooth = make_interp_spline(x1_array, y1_array)(x1_smooth)
    y2_smooth = make_interp_spline(x2_array, y2_array)(x2_smooth)
    # 子绘图 1
    # subplot(rows, cols, index)
    # 绘图一行两个
    # plt.subplot(1, 2, 1) 
    # 设置字体
    plt.rcParams['font.sans-serif'] = ['Times New Roman']  
    # x轴标题
    plt.xlabel('index')  
    plt.xlim(0, 1)
    # y轴标题
    plt.ylabel('search time')  
    # 绘制平滑曲线图，添加数据点，设置点的大小
    plt.plot(x1_smooth, y1_smooth, label = 'NSG')  
    plt.plot(x2_smooth, y2_smooth, label = 'WADG')
    # 曲线上打点
    plt.scatter(x1_array, y1_array)
    plt.scatter(x2_array, y2_array)
    # 设置曲线名称
    plt.legend(['NSG', 'WADG'],)
    plt.grid(linestyle = '--', alpha = 0.5)
    plt.title('SIFT Search Time Comparision')
    
    plt.show()
```

## 对比柱状图

```python
def comparision_histogram(self):
    base = ['1', '2', '3']
    x1 = [5, 10, 20]
    x2 = [6, 8, 15]

    # 设置字体
    plt.rcParams['font.sans-serif'] = ['Times New Roman']
    # 绘制柱状图
    # x轴标题
    # plt.xlabel('SIFT')  
    # y轴标题
    plt.ylabel('Avg Search Time') 
    base_ticks = range(len(base))
    plt.bar(base_ticks, x1, width = 0.1, label = 'NSG')
    plt.bar([i + 0.1 for i in base_ticks], x2, width = 0.1, label = 'WADG')
    # 修改 base 刻度
    plt.xticks(base_ticks, base)
    # 添加网格显示
    plt.grid(linestyle = '--', alpha = 0.5)
    #5、标题
    plt.title("SIFT Average Search Time Comparision")

    plt.show()
```

## 设置图例位置

```python
plt.legend(loc = 'upper right')
```

## plot 全局设定

```python
plt.rcParams['figure.dpi'] = 300
plt.rcParams['figure.figsize'] = (8, 6)
```

## 转换 array: reshape()

```python
import numpy as np

# 转换为 1 行
df.reshape(1, -1)
# 转换为 2 行
df.reshape(2, -1)
# 转换为 1 列
df.reshape(-1, 1)
# 转换为 2 列
df.reshape(-1, 2)
```

## 计算向量距离: pairwise_distances()

```python
from sklearn.metrics import pairwise_distances

# dist 得到的是 X, Y array 中的每对向量的距离
# dist 也是一个 array
dist = pairwise_distances(X, Y, metric = 'euclidean')
# sum(dist) 得到的是 X，Y 的距离
Dist = np.sum(dist)


# example
X = np.array([[2, 3], 
              [3, 5], 
              [5, 8]])
Y = np.array([[1, 0], 
              [2, 1]])

dist = pairwise_distances(X, Y, metric = 'euclidean')
print(dist)
Dist = np.sum(dist)
print(Dist)

# output
[[3.16227766 2.        ]
 [5.38516481 4.12310563]
 [8.94427191 7.61577311]]

31.230593108783612
```

## 计算向量距离: cdist(), pdist()

在 `scripy.spatial.distance` 库下。

`cdist()` 用于两个数组（矩阵）之间，`pdist()` 用于一个数组中。

* `cdist()`: 
  
  X = m * n, Y = a * n, res = `cdist(X, Y)` = m * a

  可以看作是 X 为 m 个 n 维向量，Y 为 a 个 n 维向量，
  `cdist()` 求 X 中每个向量到 Y 中每个向量的距离，可以用来计算分离度。

  ```python
  X=np.array([[9,3,7],[5,7,9]]) 
  # 即便是 X 中只有一组数据跟 Y 的两组数据分别求欧氏距离，
  # 也需要写成二维数组的形式，如 X=np.array([[5,3,6]])
  Y = np.array([[4,6,5],[4,2,3]])
  res = cdist(X, Y, metric='euclidean')
  print(res)

  # output
  [[a b]
   [c d]]

  # a 为向量 [9, 3, 7] 到 [4, 6, 5] 的距离
  # b 为向量 [9, 3, 7] 到 [4, 2, 3] 的距离
  # c 为向量 [5, 7, 9] 到 [4, 6, 5] 的距离
  # d 为向量 [5, 7, 9] 到 [4, 2, 3] 的距离
  ```

* `pdist()` and `squareform()`: 

  X = m * n, res = `pdist(X)` = m * m, 其中 res 为对称距离矩阵，对角线元素为 0。

  `pdist()` 可以看作是求 X 中 m 个 n 维向量之间的距离，可以用来计算内聚度。

  `squareform()` 和 `pdist()` 配套使用，用来压缩矩阵（或者还原矩阵）。

  ```python
  x = np.array([[ 0, 2, 3, 4],
                [ 2, 0, 7, 8],
                [ 3, 7, 0, 12],
                [ 4, 8, 12, 0]])
  y = dist.squareform(x)
  print(y)
  # output
  [ 2,  3,  4,  7,  8, 12]
  
  x = dist.squareform(y)
  print(x)
  # output
  [[ 0,  2,  3,  4],
   [ 2,  0,  7,  8],
   [ 3,  7,  0, 12],
   [ 4,  8, 12,  0]]
  ```


## 从列表或 array 中随机采样: random.choice()

```python
import numpy as np

'''
base = [[x1, x2, ..., xn]
        [y1, y2, ..., yn]
        ...
        [z1, z2, ..., zn]]
'''
# shape[0]: base 第一维的长度，多少行
# choice(选择的数字范围, 结果的个数，是否有放回抽取)
choices = np.random.choice(base.shape[0], size = len(queries), replace = False)
# choices 是长度为 size 列表，里面是抽取的数字（即抽取第几行）
# base[choices, :]: 抽取行数在 choices 中的行
base_random_sample = base[choices, :]


# example
import numpy as np

A = np.array([[0, 1, 2],
              [3, 4, 5],
              [6, 7, 8],
              [9, 7, 6],
              [3, 2, 2],
              [0, 1, 0],
              [1, 3, 1],
              [0, 4, 1],
              [2, 4, 2],
              [3, 6, 1]])

print(A.shape)
choices = np.random.choice(A.shape[0], size = 4, replace = False)
print('choices:', choices)
B = A[choices, :]
print(B)

# output
(10, 3)
choices: [3 6 4 9]
[[9 7 6]
 [1 3 1]
 [3 2 2]
 [3 6 1]]
```

## 从 array 中选取特定行

```python
import numpy as np
from sklearn.cluster import KMeans

queries = np.array([[1, 2, 3, 3],
                   [2, 2, 4, 5],
                   [3, 4, 5, 2],
                   [2, 3, 6, 7],
                   [5, 6, 4, 3],
                   [1, 1, 2, 1]])

kmeans = KMeans(n_clusters = 3)
kmeans.fit(queries)
cluster_centers = kmeans.cluster_centers_
labels = kmeans.labels_
print(labels)
# labels 下标与 queries 行下标对应
i = 2
# 选取 label 为 2 的 query
res = queries[labels == i]
print(res)

# output
[0 2 1 2 1 0]

[[2 2 4 5]
 [2 3 6 7]]
```

## 去重: unique()

```python
import numpy as np

A = [1, 2, 2, 5, 3, 4, 3]
a = np.unique(A)
print(a)

# output
[1 2 3 4 5]



# 返回新列表元素在旧列表中的位置（下标）
a, indices = np.unique(A, return_index=True)
print(a)		 # 列表
print(indices)	 # 下标

# output
[1 2 3 4 5]
[0 1 4 5 3]



# 旧列表的元素在新列表的位置
a, indices = np.unique(A, return_inverse=True)   
print(a)
print(indices)
print(a[indices])     # 使用下标重构原数组

# output
[1 2 3 4 5]
[0 1 1 4 2 3 2]
[1 2 2 5 3 4 3]



# 每个元素在旧列表里各自出现了几次
a, indices = np.unique(A, return_counts=True)    
print(a)
print(indices)

# output
[1 2 3 4 5]
[1 2 2 1 1]



# 向量去重
B = np.array([1, 2, 3, 4]
             [1, 2, 3, 4]
             [2, 3, 4, 6])
b = np.unique(B, axis = 0)

# output
[[1, 2, 3, 4]
 [2, 3, 4, 6]]
 
```

## 可遍历对象组合为索引序列: enurmerate()

```python
seasons = ['Spring', 'Summer', 'Fall', 'Winter']
list(enumerate(seasons))
# output
[(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]

list(enumerate(seasons, start=1))       # 下标从 1 开始
# output
[(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]
```