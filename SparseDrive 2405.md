---
categories:
  - "[[Papers]]"
title: "SparseDrive: End-to-End Autonomous Driving via Sparse Scene Representation"
authors:
  - Wenchao Sun
  - Xuewu Lin
  - Yining Shi
  - Chuang Zhang
  - Haoran Wu
  - Sifa Zheng
venue: ICRA
year: 2025
doi:
url: https://arxiv.org/abs/2405.19620
pdf: https://arxiv.org/pdf/2405.19620v2
field:
  - autonomous-driving
  - end-to-end-driving
keywords:
  - sparse-scene-representation
  - planning
  - motion-prediction
  - nuscenes
status:
  - read
rating:
dataset:
  - nuScenes
method:
  - sparse scene representation
  - parallel motion planner
  - collision-aware rescore
task: end-to-end autonomous driving
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[SparseDriveV2 2603]]"
  - "[[Hydra-MDP 2406]]"
  - "[[transfuser]]"
  - "[[TransFuser 2205]]"
---

一句话：**SparseDrive 是一个以 sparse scene representation 为核心的端到端自动驾驶框架，把 detection / tracking / online mapping / motion prediction / planning 统一到同一个 sparse-centric 范式里。**

# 核心 takeaways

- SparseDrive 关注的不只是 planning，而是一个更完整的端到端 AD 系统。
- 它的关键主张是：
  - BEV 特征计算昂贵；
  - prediction 和 planning 的设计也常常不够高效；
  - 因此应该把整个系统重构为 **sparse-centric** 形式。
- 论文提出两个核心模块：
  1. **symmetric sparse perception module**
  2. **parallel motion planner**
- planning 部分使用 **hierarchical planning selection strategy**，并带有 **collision-aware rescore module** 来提升安全性。

# 模型结构

## 1. 整体框架

SparseDrive 大致分成三层：

1. multi-view image encoder
2. symmetric sparse perception
3. parallel motion planner

README 对它的概括很直接：
- 先编码多视角图像；
- 再学习 sparse scene representation；
- 最后并行执行 motion prediction 与 planning。

## 2. sparse perception

论文和 README 都强调，它用一个对称结构统一：
- detection
- tracking
- online mapping

也就是说，SparseDrive 的重点不是单独造一个 planner，而是想把环境表示和下游任务统一起来。

## 3. parallel motion planner

SparseDrive 认为：
- motion prediction 和 planning 有很强相似性；
- 因此可以做并行设计。

planning 不是简单单峰回归，而是一个小规模多模态候选选择问题，然后再通过层级选择与 rescore 得到最终轨迹。

## 4. 安全机制

SparseDrive 的 planning 里一个很重要的部分是：
- **collision-aware rescore**

这意味着它会对候选轨迹根据碰撞风险进行再打分，而不是只依赖 imitation / regression 输出本身。

这一点和后续 Hydra-MDP、SparseDriveV2 的“metric-aware / score-aware”路线是相关的，但 SparseDrive 更像显式后处理安全筛选。

# 结果与定位

SparseDrive 的主结果主要基于 **nuScenes**，而不是 NAVSIM / Bench2Drive。

README 中强调：
- 在 detection / tracking / mapping / motion / planning 多任务上都取得强结果；
- 尤其 planning 的 collision rate 很低；
- 同时训练和推理效率较高。

它更像一篇“完整系统设计”的论文，而不是只盯住 planning leaderboard 的论文。

# 和 SparseDriveV2 的关系

这是最重要的阅读关系。

## 相同点

- 两篇都来自同一条研究脉络；
- 都是 end-to-end autonomous driving；
- 都非常在意 planning 的安全与效率；
- 都不是“直接回归一条轨迹就结束”的简单范式。

## 最大区别

### SparseDrive
- 核心是 **sparse scene representation**；
- 目标是统一 perception + planning；
- 更系统、更全栈。

### SparseDriveV2
- 核心是 **factorized trajectory vocabulary + scalable scoring**；
- 目标是把 planning candidate space 做大并高效筛选；
- 更 planner-centric。

一句话总结：
- **SparseDrive**：先把“场景”表示好，再规划；
- **SparseDriveV2**：先把“动作空间”建得足够密，再高效打分。

# 和 Hydra-MDP 的关系

SparseDrive 与 Hydra-MDP 的差别也很明显：

### SparseDrive
- 强调 scene representation；
- planning 候选规模较小；
- 带 collision-aware rescore。

### Hydra-MDP
- 强调 fixed trajectory vocabulary；
- 用 human teacher + rule-based teacher 进行 hydra-distillation；
- 更像 score-based planning over monolithic anchors。

所以从研究脉络上看，大致可以理解成：

- **SparseDrive**：系统层面的 sparse end-to-end AD
- **Hydra-MDP**：teacher-distilled score-based multimodal planning
- **SparseDriveV2**：factorized dense-vocabulary scoring-based planning

# 我的理解

SparseDrive 更像是这一系列工作的“世界观起点”：
- 自动驾驶端到端系统应该减少密集中间表示，转向 sparse representation；
- planning 应该显式关注安全，不只是 imitation。

而后续 Hydra-MDP / SparseDriveV2 则是在 planning 这一块继续深化：
- Hydra-MDP 深化了多目标蒸馏；
- SparseDriveV2 深化了 candidate space 表示与筛选机制。

# 关联阅读

- [[SparseDriveV2 2603]]
- [[Hydra-MDP 2406]]
- [[transfuser]]
- [[TransFuser 2205]]
