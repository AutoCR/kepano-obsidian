---
categories:
  - "[[Papers]]"
title: "DriveSuprim: Towards Precise Trajectory Selection for End-to-End Planning"
authors:
  - Wenhao Yao
  - Zhenxin Li
affiliation:
  - "Shanghai Key Lab of Intell. Info. Processing, School of CS, Fudan University"
  - "Shanghai Collaborative Innovation Center of Intelligent Visual Computing"
  - "NVIDIA"
venue:
year: 2025
doi:
url: https://arxiv.org/abs/2506.06659
pdf: "[[2506 DriveSuprim Towards precise trajectory selection for end to end planning.pdf]]"
field: []
keywords:
  - nvidia
  - score-based
status:
  - read
rating:
dataset: []
method: []
task:
created: 2026-03-13
updated:
tags:
  - paper
---
现有Selected-based  e2e方法存在hard negatives轨迹的筛选、轨迹方向分布不均、二值标签导致训练不问题的问题。提出了三大创新：

1. coarse to fine trajectory selection. 先从8192条轨迹中粗选出topk（256），再通过3层transformer decoder精选，每层输出中间打分，最后一层输出最终分数。
2. 输入图像旋转增强，解决转向偏置（轨迹方向分布不均）
3. 自蒸馏软标签，teacher-student框架进行训练。Teacher通过student权重的EMA更新，只看原始输入，输出稳定评分。student同时输入原始和旋转增强，teacher的输出通过截断生成soft label作为student的训练真值，推理的时候用的是teacher.

j轨迹精选，对每层输出都做监督的原因，防止梯度消失，深层监督（Deep supervision）的典型应用。

Refinement Decoder 的作用是：

- 第一层：粗粒度对比
- 第二层：更精细辨别
- 第三层：非常精细的精准鉴别

如果只有最后一层接受监督：

- 前几层没有梯度信号，无法学习“逐级加精度”的结构
- 会导致“所有 refinement 都挤在最后一层”，效果变差

**这是和 Deformable DETR、Cascade R-CNN 完全同样的设计哲学。**