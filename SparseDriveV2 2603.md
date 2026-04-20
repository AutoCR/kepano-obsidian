---
categories:
  - "[[Papers]]"
title: "SparseDriveV2: Scoring is All You Need for End-to-End Autonomous Driving"
authors:
  - Wenchao Sun
  - Xuewu Lin
  - Keyu Chen
  - Zixiang Pei
  - Xiang Li
  - Yining Shi
  - Sifa Zheng
venue: arXiv
year: 2026
doi:
url: https://arxiv.org/abs/2603.29163
pdf: https://arxiv.org/pdf/2603.29163v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - score-based
  - trajectory-vocabulary
  - navsim
  - bench2drive
  - multimodal-planning
status:
  - read
rating:
dataset:
  - NAVSIM
  - Bench2Drive
method:
  - factorized trajectory vocabulary
  - coarse-to-fine scoring
  - metric distillation
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[Hydra-MDP 2406]]"
  - "[[transfuser]]"
  - "[[TransFuser 2205]]"
  - "[[navsim评估方法]]"
  - "[[SparseDrive 2405]]"
  - "[[FeaXDrive 2604]]"
---

一句话：**这篇论文的核心观点是，端到端规划不一定需要动态生成轨迹；只要候选轨迹空间足够稠密、并且打分足够高效，纯 scoring-based planning 也可以做到 SOTA。**

# 核心 takeaways

- 论文先用 **Hydra-MDP** 做了一个 scaling study，结论很直接：**静态轨迹词表越密，性能越好，在算力打满前还没有看到明显饱和。**
- 所以问题不在于 static vocabulary 这个范式不行，而在于：
  1. 轨迹词表覆盖不够；
  2. 直接对超大词表逐条打分的成本太高。
- SparseDriveV2 的解法是把轨迹拆成两部分：
  - **path**：往哪里走；
  - **velocity profile**：以什么速度推进。
- 然后把两者做组合，得到远比传统词表更稠密的动作空间，再用 **coarse-to-fine** 的方式高效筛选。
- 它的论文标题“Scoring is All You Need”本质上是在说：**生成不是唯一出路，scoring 范式在足够好的表示和筛选机制下仍然能继续扩展。**

# 模型细节

## 1. 输入与 backbone

- 使用 **3 路相机**输入。
- 分辨率：**256 × 512**。
- backbone：**ResNet-34**。
- 代码里是 image-only planner，没有像 Hydra-MDP 那样显式依赖 LiDAR perception 分支。

## 2. 轨迹表示：从整条轨迹改为 path + velocity

传统方法通常把一条未来轨迹当成一个完整 anchor。SparseDriveV2 则把轨迹拆成：

- **Geometric path**：空间几何形状；
- **Velocity profile**：时间上的速度变化。

这两个部分可以组合回完整轨迹，因此可以用比较小的两个词表，组合出非常大的候选集。

## 3. 词表规模

NAVSIM 主设置里：

- **1024 个 path anchors**
- **256 个 velocity anchors**

组合后总候选数：

- **1024 × 256 = 262,144**

论文明确说这比 prior scoring-based 方法常用的 **8192 anchors** 稠密 **32×**。

## 4. path / velocity 的具体定义

- path anchors：
  - spatial interval = **1m**
  - max spatial horizon = **50m**
- velocity anchors：
  - temporal interval = **0.5s**
  - horizon = **4s**

也就是说，它在空间和时间两个维度上分别建模，再做组合。

## 5. scoring 结构

### coarse stage
先分别打分：
- path score
- velocity score

### fine stage
再把 top path 和 top velocity 组合成少量高质量候选，对这些组合轨迹做更细的 joint scoring。

论文给出的 NAVSIM 推理筛选流程：
- 第一层保留：**128 path + 64 velocity**
- 第二层保留：**20 path + 20 velocity**
- 最后精排：**400 trajectories**

NAVSIM v2 为了加速 metric GT 计算，第二层 velocity 从 20 改成了 **10**。

## 6. trajectory re-conditioning

如果直接把 path embedding 和 velocity embedding 相加，会隐含一个假设：path 和 speed 是独立的。

现实里显然不成立，比如：
- 急转弯 + 很高速度，往往不合理。

所以它又加了一个 **trajectory re-conditioning** 模块，让组合后的轨迹再和 scene feature 做一次交互，从而补上时空耦合关系。

## 7. metric heads

SparseDriveV2 不是只输出一个总分，而是直接预测多个 benchmark-aware 子分数，用同样的聚合方式选最终轨迹。

这使得它的训练目标更贴近最终评测指标，而不是只做 imitation。

# 训练方式

训练目标分成四部分：

1. **path-level soft classification loss**
2. **velocity-level soft classification loss**
3. **trajectory-level imitation loss**
4. **metric distillation loss**

总体可以理解为：

- 先教会模型粗筛 path；
- 再教会模型粗筛 velocity；
- 再教会模型在组合轨迹上做 imitation；
- 最后再用 rule-based metric teacher 去对齐 safety / progress / comfort / rule compliance。

这一点和 Hydra-MDP 有相似性：两者都使用 rule-based teacher，但重点不一样。Hydra-MDP 更强调 multi-target distillation，本篇更强调 factorized vocabulary + scalable scoring。

# 结果

论文中报告的代表结果：

- **NAVSIM v1**：**92.0 PDMS**
- **NAVSIM v2**：**90.1 EPDMS**
- **Bench2Drive**：**89.15 Driving Score**，**70.00 Success Rate**

比较有说服力的一点是：这些结果是在 **ResNet-34** 这种相对轻量 backbone 下取得的，说明收益主要来自规划结构，而不是单纯靠更大的视觉 backbone 堆出来。

# 推理效率 / inference

这里我专门查过：**论文没有明确给出 SparseDriveV2 的 FPS、latency 或单帧 ms 数字。**

论文只给了几类“间接证据”：

- 它强调 coarse-to-fine 筛选使得超大候选空间在推理时变得可行；
- 只需要对最后 400 条（或 200 条）候选做精排；
- backbone 采用的是 ResNet-34。

所以目前可以确定的是：
- **它在方法设计上是为高效推理服务的**；
- **但论文没有公开明确的 inference time 指标。**

这一点和原始 SparseDrive 不一样，SparseDrive 在 README/结果里会更直接地报 FPS。

# 和 SparseDrive 的区别

## 核心目标不同

### SparseDrive
- 是一个 **全链路 sparse scene-centric E2E AD 框架**；
- 统一做 detection / tracking / online mapping / motion prediction / planning；
- 重点在 **sparse scene representation**。

### SparseDriveV2
- 更纯粹地聚焦在 **planning**；
- 核心问题变成：怎么在超稠密轨迹候选空间中高效打分。

## planning 方式不同

### SparseDrive
- planning 和 motion prediction 是并行设计；
- 候选模式数比较小；
- 还用了 **hierarchical planning selection + collision-aware rescore**。

### SparseDriveV2
- 不再是“小规模直接预测几个 mode”；
- 而是“**构造一个极大的候选词表，再高效打分**”。

## 稀疏性的含义不同

- **SparseDrive** 的 sparse 主要指 **scene representation sparse**；
- **SparseDriveV2** 的 sparse / scalable 主要指 **trajectory vocabulary representation 与 scoring process**。

## 我自己的理解

SparseDrive 更像：
- “把环境理解做好，再带着 scene representation 去规划”。

SparseDriveV2 更像：
- “把动作空间建得足够密，再通过 scoring 选出最优动作”。

所以两者其实不是简单的“v2 把 v1 小修小补”，而是规划视角发生了明显转移。

# 和 Hydra-MDP 的区别

SparseDriveV2 和 Hydra-MDP 的关系更紧密，因为论文本身就是从 Hydra-MDP 的 scaling study 出发的。

## 1. 输入模态

### Hydra-MDP
- **LiDAR + Camera**
- perception network 基于 **Transfuser**
- LiDAR 来自 **4 帧点云 splat 到 BEV**
- image 默认输入分辨率 **256 × 1024**

### SparseDriveV2
- **Camera-only**
- 3 camera，**256 × 512**
- backbone 为 ResNet-34 + FPN

=> V2 在感知输入上更轻、更纯 planning。

## 2. anchor / vocabulary 设计

### Hydra-MDP
- 使用 **monolithic trajectory vocabulary**
- 从 nuPlan 随机采样 **70 万条轨迹**
- 每条轨迹：
  - **40 timestamps**
  - (x, y, heading)
  - **10Hz**
  - **4s horizon**
- 然后 K-means 压缩成：
  - **V4096** 或 **V8192**

### SparseDriveV2
- 不直接聚类完整轨迹；
- 先拆成：
  - path vocabulary = 1024
  - velocity vocabulary = 256
- 组合成 **262,144** 条候选轨迹。

=> 最大区别：
- **Hydra-MDP 是整条轨迹 anchor**；
- **SparseDriveV2 是 factorized anchor**。

## 3. 模型重点

### Hydra-MDP
- 核心创新在 **multi-target hydra-distillation**；
- 用 human teacher + rule-based teachers；
- 用多头输出不同 metric 对应的 sub-scores。

### SparseDriveV2
- 也有 metric distillation；
- 但核心创新不在 supervision，而在：
  1. factorized vocabulary
  2. coarse-to-fine scoring
  3. trajectory re-conditioning

=> 简单说：
- **Hydra-MDP 的创新更偏 training target / supervision**；
- **SparseDriveV2 的创新更偏 representation / search / scoring。**

## 4. 推理机制

### Hydra-MDP
- 直接对固定词表里的每条 trajectory 打分；
- 输出 imitation score + metric sub-scores；
- 再通过 hand-tuned weighted confidence 组合成最终 cost。

### SparseDriveV2
- 不会一上来对 26 万条完整轨迹全部精打分；
- 而是先 path / velocity 独立粗筛，再对小规模组合轨迹做精排。

=> Hydra-MDP 的瓶颈是：
- 词表越大，直接打分越重。

SparseDriveV2 的核心就是解决这个瓶颈。

## 5. scaling 路线

### Hydra-MDP
- 扩大词表（4096 → 8192）
- 放大 backbone（ViT-L / V2-99）
- 最终 submission 用 **多模型 ensemble**

### SparseDriveV2
- 主要靠：
  - factorized vocabulary scaling
  - hierarchical scoring
- 在轻量 ResNet-34 下就能得到很强结果。

=> 我对这两篇关系的理解是：
**SparseDriveV2 本质上是在回答 Hydra-MDP 暴露出来的一个问题：既然 Hydra-MDP 词表越密越好，那为什么不重新设计词表表示与筛选方式，把“更密”真正做到底？**

# 我当前的总结结论

1. **SparseDriveV2 的论文价值很大一部分在于重新给 scoring-based planning 正名。**
2. 它并不是简单反对生成式规划，而是指出：
   - 之前 scoring-based 方法弱，很多时候不是范式弱，
   - 而是 candidate space 不够密、scoring 也不够 scalable。
3. 跟 SparseDrive 比：
   - 从“稀疏场景表示”转向了“稠密动作空间打分”。
4. 跟 Hydra-MDP 比：
   - 从“多教师蒸馏强化固定词表”转向了“重构词表本身 + 重构筛选流程”。

# 关联阅读

- [[Hydra-MDP 2406]]
- [[transfuser]]
- [[TransFuser 2205]]
- [[navsim评估方法]]
- [[SparseDrive 2405]]
