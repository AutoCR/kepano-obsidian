---
categories:
  - "[[Papers]]"
title: "Hydra-MDP: End-to-end Multimodal Planning with Multi-target Hydra-Distillation"
authors:
  - Zhenxin Li
venue:
year: 2024
doi:
url:
pdf: "[[2024 hydra-mdp nvidia.pdf]]"
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
related:
  - "[[SparseDriveV2 2603]]"
  - "[[SparseDrive 2405]]"
  - "[[transfuser]]"
---
感知使用[[transfuser]]，融合视觉和雷达，输出环境token。

通过nuplan数据集构架词表库，70w条轨迹通过k-mean压缩成8192条轨迹词表。后面模型根据环境token和自车状态，通过transformer输出8192条轨迹的分数，选一条分数最高的轨迹输出。

训练包括两类loss：轨迹词表跟专家轨迹的匹配程度，前向仿真器对每条轨迹打分（舒适度、碰撞等等）

论文称其训练过程为知识蒸馏：**专家教师+规则教师（前向仿真器），规则教师是可以看到GT感知的，能够更精确的打分。学生模型在推理的时候不用前向仿真和gt感知也可以打分**。这是个比较好的点。