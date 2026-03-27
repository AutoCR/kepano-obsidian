---
categories:
  - "[[Papers]]"
title: "LaST-VLA: Thinking in Latent Spatio-Temporal Space for Vision-Language-Action in Autonomous Driving"
authors:
  - xiaomi
venue:
year: 2026
doi:
url:
pdf: https://arxiv.org/pdf/2603.01928
field: []
keywords:
  - e2e
  - VLA
  - InternVL3
status:
  - read
rating:
dataset: []
method: []
task:
created: 2026-03-20
updated:
tags:
  - paper
---
navsim SOTA v1 91.3, v2 87.1

direct-VLA(no reasoning): 推理很快，但是可解释性差、鲁棒性差
text reasoning VLA：推理慢
当前的latent reasoning VLA: 训练不稳定，且推理过程没有监督
本文的核心就是有监督的latent reasoning VLA

latent reasoning就是在llm中固定一段特定数量的token，这段token不用decoding成text。一方面数量有限，另一方面不用decoding，所有省时间。

把模型的hidden states区分出geometry和dynamics tokens，通过geometry adaptor和dynamics adaptor把两部分token分别映射到teacher model的hidden states上，并用teacher model的hidden states做监督。geometry teacher: VGGT, dynamics teacher: cosmos。
最佳latent reasoning数量：dynamics 3x12, geometry 12
Structured Causal Masking:
tokens : img, text, dyn, geo, trajectory.
1. 标准的因果mask，后面只能看到前面
2. dyn和geo相互看不到
3. trajectory只能看到dyn和geo

轨迹以text的形式输出
VLM模型使用的InternVL3
训练过程：两阶段微调+GRPO RL
两阶段微调：一阶段调大latent reasoning代价的权重，二阶段调到trajecotry代价权重
感觉从一定程度，latent reasoning VLA丧失了可解释性，先对于text reasoning vla

这片论文不止是知识蒸馏：
1. 它改变了reasoning模式
2. 引入了新的token
3. 设计了课学习的adpator，虽然知识训练的时候用
4. 修改了attention,增加mask，让trajectory看不到img
