---
title: 【文档小记】记录 Python 中使用过的有用代码
tags:
  - WADG
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

## 全局设定

```python
plt.rcParams['figure.dpi'] = 300
plt.rcParams['figure.figsize'] = (8, 6)
```