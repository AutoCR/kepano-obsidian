---
categories:
  - "[[Papers]]"
title: Pseudo-Simulation for Autonomous Driving
authors: []
venue:
year: 2025
doi:
url:
pdf: https://arxiv.org/pdf/2506.04218
field: []
keywords: []
status: to-read
rating:
dataset: []
method: []
task:
created: 2026-04-01
updated:
tags:
  - paper
---
Stage-2 的每个场景只有一帧输入观测；  
这些观测是基于“该场景中人类专家在 4s 时刻的 GT 位姿”，  
对 ego 位置在前后左右进行物理可达的偏移采样，  
再通过神经渲染生成的。  
Stage-1 的输出不参与 Stage-2 场景生成，只参与权重计算。

> **Stage-2 的 history 不是从 Stage-1 rollout 得到的，  
> 也不是 planner 自己生成的，  
> 而是：  
> 👉 从真实人类驾驶数据中，选一条“与该 Stage-2 起点最相近、物理一致”的历史轨迹，拷贝过来用。**

这是一个**离线、planner-agnostic 的构造过程**。