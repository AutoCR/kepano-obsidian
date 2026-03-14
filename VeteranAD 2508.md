---
categories:
  - "[[Papers]]"
title: "Perception in plan: coupled perception and planning for end to end autonomous driving."
authors:
  - Bozhou Zhang
venue:
year: 2025
doi:
url:
pdf: "[[2508 VeteranAD perception in plan coupled perception and planning for end to end autonomous driving.pdf]]"
field: []
keywords:
  - e2e
  - resnet-34
  - LSS
  - transformer
  - k-means
  - fudan
status:
  - read
rating:
dataset: []
method: []
task:
created: 2026-03-12
updated:
tags:
  - paper
---

提出了perception-in-plan框架。感知通过特定时刻轨迹上的点来做该点附近的局部注意力，plan根据感知结果修正该时刻的轨迹点，从0时刻到T，以auto-regressive的方式生成所有轨迹点。

感知模型：backbone（resnet-34), bev(lss), 障碍物识别(transformer)，基于guide points应道的局部自注意力。

规划模型：基于trajectory qu eries的自回归解码器，MLP+motion-aware layernorm修正anchored trajectories.

anchored trajectories是在数据集上通过kmeans聚类出来的20中典型轨迹。模型的输出轨迹都是基于这20条轨迹修正出来，在最优一个轨迹点query上，会有一个mlp输出该轨迹的分数，最终输出一个分数最高的轨迹。

ps：MALN就是在 LayerNorm 基础上，引入轨迹运动信息，通过 MLP 动态生成归一化参数，使轨迹 query 在时间维度上的演化更平滑。