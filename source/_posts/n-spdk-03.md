---
title: 【学习笔记-存储】SPDK（三）：spdk_nvme_perf 代码浅析
tags:
  - 存储
  - SPDK
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - 存储
comments: false
cover: false
date: 2024-09-01 19:35:45
---

SPDK 自带的性能测试应用 spdk_nvme_perf 的源代码 perf.c 的粗浅分析，进一步熟悉 SPDK 用户态驱动的主要工作流程和方式。

<!-- more -->

## 简介

perf 是 SPDK 用来测试 NVMe SSD 性能的工具，最新版本的 SPDK 中 perf 源代码在 `spdk/app/spdk_nvme_perf/` 路径下。perf 主要用来测试 NVMe SSD 的 IOPS，Bandwidth 和 Latency，它既可以测本地的 target，也可以测远端的 target。

## 