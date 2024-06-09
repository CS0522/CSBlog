---
title: 【论文随记】HNSW 论文 + hnswlib 代码浅读
tags:
  - 论文
  - HNSW
  - ANN
toc: true
languages:
  - zh-CN
categories:
  - 论文随记
  - ANN
comments: false
cover: false
date: 2024-05-16 20:17:11
---

文章主要是记录 HNSW 原理以及 Search 过程代码的学习。

<!-- more -->

## 论文浅读

目前已有较详细的解析，附上参考链接：

[大白话理解 HNSW](https://juejin.cn/post/7082579087776022541)

[【论文笔记】HNSW（Hierarchical Navigable Small World graphs）](https://zhuanlan.zhihu.com/p/397628955)

[近似最近邻算法 HNSW 学习笔记（二） 主要算法伪代码分析](https://www.ryanligod.com/2018/11/29/2018-11-29%20HNSW%20%E4%B8%BB%E8%A6%81%E7%AE%97%E6%B3%95/)



## Search 过程代码

从 [hnswlib](https://github.com/nmslib/hnswlib) 库中截取部分代码。

```cpp
// hnswlib/hnswalg.h

// KNN 搜索函数
/**
 * 目前来看，这里与伪代码中不太一致
 * 伪代码中将每次搜索 layer 单独写成一个函数
 * 实现中没有单独写成一个函数，仅写了一个 search base layer 函数
 * 实现中通过取消 check deletions 和忽略 stop conditions 提高性能
*/
std::priority_queue<std::pair<dist_t, labeltype >>
searchKnn(const void *query_data, size_t k, BaseFilterFunctor* isIdAllowed = nullptr) const {
    std::priority_queue<std::pair<dist_t, labeltype >> result;
    if (cur_element_count == 0) return result;

    // currObj: 进入 base layer 的 enterpoint
    tableint currObj = enterpoint_node_;
    dist_t curdist = fstdistfunc_(query_data, getDataByInternalId(enterpoint_node_),dist_func_param_);

    // 与 pseudo code 中从最高层遍历至倒数第二层的逻辑一致
    for (int level = maxlevel_; level > 0; level--) {
        bool changed = true;
        while (changed) {
            changed = false;
            unsigned int *data;

            // 获取了当前 level 的数据
            data = (unsigned int *) get_linklist(currObj, level);
            int size = getListCount(data);
            metric_hops++;
            metric_distance_computations+=size;

            tableint *datal = (tableint *) (data + 1);
            // 比较当前 level 的点
            for (int i = 0; i < size; i++) {
                tableint cand = datal[i];
                if (cand < 0 || cand > max_elements_)
                    throw std::runtime_error("cand error");
                dist_t d = fstdistfunc_(query_data, getDataByInternalId(cand), dist_func_param_);

                // 如果当前层（非 base layer）存在更近的点
                // 改变 currObj，继续遍历当前层
                /**
                    没搞懂：changed = true 后，while 继续循环，这一层重新遍历一遍？
                */
                if (d < curdist) {
                    curdist = d;
                    currObj = cand;
                    changed = true;
                }
            }
        }
    }

    std::priority_queue<std::pair<dist_t, tableint>, std::vector<std::pair<dist_t,tableint>>, CompareByFirst> top_candidates;
    bool bare_bone_search = !num_deleted_ && !isIdAllowed;
    if (bare_bone_search) {
        // 搜索底层函数
        top_candidates = searchBaseLayerST<true>(
                currObj, query_data, std::max(ef_, k), isIdAllowed);
    } else {
        top_candidates = searchBaseLayerST<false>(
                currObj, query_data, std::max(ef_, k), isIdAllowed);
    }

    while (top_candidates.size() > k) {
        top_candidates.pop();
    }
    while (top_candidates.size() > 0) {
        std::pair<dist_t, tableint> rez = top_candidates.top();
        result.push(std::pair<dist_t, labeltype>(rez.first, getExternalLabel(rez.second)));
            top_candidates.pop();
    }
    return result;
}


// 搜索底层函数
// bare_bone_search means there is no check for deletions and stop condition is ignoredin return of extra performance
template <bool bare_bone_search = true, bool collect_metrics = false>
std::priority_queue<std::pair<dist_t, tableint>, std::vector<std::pair<dist_t,tableint>>, CompareByFirst>
searchBaseLayerST(
    tableint ep_id,
    const void *data_point,
    size_t ef,
    BaseFilterFunctor* isIdAllowed = nullptr,
    BaseSearchStopCondition<dist_t>* stop_condition = nullptr) const {
    
    VisitedList *vl = visited_list_pool_->getFreeVisitedList();
    vl_type *visited_array = vl->mass;
    vl_type visited_array_tag = vl->curV;

    std::priority_queue<std::pair<dist_t, tableint>, std::vector<std::pair<dist_t,tableint>>, CompareByFirst> top_candidates;
    std::priority_queue<std::pair<dist_t, tableint>, std::vector<std::pair<dist_t,tableint>>, CompareByFirst> candidate_set;

    dist_t lowerBound;
    if (bare_bone_search || 
        (!isMarkedDeleted(ep_id) && ((!isIdAllowed) || (*isIdAllowed)(getExternalLabel(ep_id))))) {
        char* ep_data = getDataByInternalId(ep_id);
        dist_t dist = fstdistfunc_(data_point, ep_data, dist_func_param_);
        lowerBound = dist;
        top_candidates.emplace(dist, ep_id);
        if (!bare_bone_search && stop_condition) {
            stop_condition->add_point_to_result(getExternalLabel(ep_id), ep_data, dist);
        }
        candidate_set.emplace(-dist, ep_id);
    } else {
        lowerBound = std::numeric_limits<dist_t>::max();
        candidate_set.emplace(-lowerBound, ep_id);
    }

    visited_array[ep_id] = visited_array_tag;

    while (!candidate_set.empty()) {
        std::pair<dist_t, tableint> current_node_pair = candidate_set.top();
        dist_t candidate_dist = -current_node_pair.first;

        bool flag_stop_search;
        if (bare_bone_search) {
            flag_stop_search = candidate_dist > lowerBound;
        } else {
            if (stop_condition) {
                flag_stop_search = stop_condition->should_stop_search(candidate_dist, lowerBound);
            } else {
                flag_stop_search = candidate_dist > lowerBound && top_candidates.size() == ef;
            }
        }
        if (flag_stop_search) {
            break;
        }
        candidate_set.pop();

        tableint current_node_id = current_node_pair.second;
        int *data = (int *) get_linklist0(current_node_id);
        size_t size = getListCount((linklistsizeint*)data);
        // bool cur_node_deleted = isMarkedDeleted(current_node_id);
        if (collect_metrics) {
            metric_hops++;
            metric_distance_computations+=size;
        }
    
    // 未完...
    }
}
```