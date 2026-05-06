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
affiliation:
  - "Tsinghua University"
  - "Horizon"
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
  - "[[FeaXDrive 2604]]"
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

# 模型细节（读 config + 代码整理）

以下来自官方 repo `swc-17/SparseDrive`，主要配置 `projects/configs/sparsedrive_small_stage2.py` 和 `projects/mmdet3d_plugin/models/motion/*`。

## 顶层结构

- 输入：6 路相机，704×256。
- Backbone：ResNet-50（ImageNet 预训练），`frozen_stages=-1`，同时训练。
- Neck：FPN，4 层，strides `[4, 8, 16, 32]`，统一 `C = 256`。
- 主干 embed_dims = 256，num_heads = 8，dropout 0.1，FP16 训练。
- 附加 `DenseDepthNet` 辅助深度监督（3 层，loss_weight 0.2）。

## Symmetric Sparse Perception

"对称" 是指 detection 头和 map 头用的是**同一个 `Sparse4DHead` 骨架**，只是 anchor 几何不同。实际上就是 Sparse4Dv3。

**Detection head**
- **900 个物体 anchors**，用 nuScenes GT 框做 k-means 初始化。
- **600 个 temporal instances** 跨帧传播（`confidence_decay=0.6`），tracking 由 query propagation 自然产生，没有显式 tracker。
- 10 类，带 yaw refinement 和 quality estimation（centerness + yawness）。
- Key-point generator：6 个可学习点 + 7 个固定点（0.45 半径十字架）用于 deformable aggregation。
- Decoder 6 层：第 1 层 single-frame `[gnn, norm, deformable, ffn, norm, refine]`，其余 5 层前面加 `temp_gnn` 去读 temporal instance bank。
- 全部用 FlashAttention；`decouple_attn=True`（content 和 position 拼接而非相加，embed_dims×2）。

**Map head**
- **100 条 polyline anchors**（GT 向量化地图的 k-means），**33 个 temporal**。
- 3 类：`ped_crossing`, `divider`, `boundary`。
- ROI 30m × 60m，每条 polyline 20 个采样点。
- `SparsePoint3DKeyPointsGenerator` 用 3 个可学习点 + 固定高度 `(0, ±0.5, ±1) m`（地面在 lidar 系 `-1.84023`）把 2D polyline 抬到 3D，用于图像 cross-attn。
- Hungarian line assignment，L1 cost 允许方向翻转。
- Loss：FocalLoss (cls, w=1) + LinesL1Loss (reg, w=10)。

## Parallel Motion Planner

这是论文核心新模块。Agent 和 ego **共用一个 decoder stack**：

```
operation_order = [temp_gnn, gnn, norm, cross_gnn, norm, ffn, norm] × 3 + [refine]
```

- `temp_gnn`：在 instance queue（长度 4 = 3 历史 + 当前）上做时间自注意力。
- `gnn`：agent↔agent、agent↔ego 交互。
- `cross_gnn`：trajectory queries 对 **top-50 detections + top-10 map polylines** 做 cross-attn（只用置信度最高的环境 token）。

**Motion prediction（其他车）**
- 输入：det head 挑出的 top-50 agent instances。
- **6 modes × 12 steps** 未来轨迹（6 s @ 2 Hz）。
- Anchor bank `kmeans_motion_6.npy`：**按类别**做 k-means 的 GT future 轨迹。用 `classification.argmax()` 选类别，再根据 agent 自身 yaw 旋转到 lidar 系 (`_agent2lidar`)。
- Loss：FocalLoss (w=0.2) + L1 (w=0.2)。

**Ego planning**
- ego 就是**一个额外的 instance token**，拼到 agent tokens 后面 — 自注意力天然建模 ego-agent 博弈，没有独立 planner 网络。
- ego 特征来自 stride-32 feature map 的 `ego_feature_encoder`（每帧一个向量）；ego anchor 是可学习参数，每帧用 `prev_ego_status` 更新 `VY`。
- **Plan anchors 按 driving command 因子化**：
  - `ego_fut_mode = 6`（每个命令下的候选数）
  - `ego_fut_ts = 6`（3 s @ 2 Hz）
  - 输出 reshape 成 `(bs, 3, 6, 6, 2)`，那个 `3` 就是 `{left, straight, right}` — **总共 18 个候选**。
- 额外有 `status` 头预测 ego 当前速度/加速度（L1, w=1.0），用于下一帧 anchor 初始化。
- Loss：FocalLoss (w=0.5) + L1 reg (w=1.0) + status L1 (w=1.0)。

## Hierarchical Planning Selection

"层级" 实际上是 **cmd → mode** 两级剪枝，不是深树。见 `HierarchicalPlanningDecoder.select`：

1. **Command gating**：`cmd = data['gt_ego_fut_cmd'].argmax()` 切掉 cmd 维，18 → 6 候选。
2. **Collision-aware rescore**：`plan_cls = plan_cls + (-999) * collision_flag`。
3. **Argmax**：挑最高分那条作为 `final_planning`。

> 注意：eval 时用的是 **GT command**，和 UniAD / VAD 一样。open-loop nuScenes 上这是已知的 leak，读 collision rate 要带着这个注脚看。

## Collision-aware Rescore（具体算法）

实现在 `decoder.py` 的 `HierarchicalPlanningDecoder.rescore`，是**纯后处理几何检测**，无学习。

对每条 ego 候选：

1. **构造 ego 框序列**：尺寸 `[4.08, 1.73, 1.56] × 1.1`（1.1 倍缩放做 margin），yaw 由轨迹有限差分得到，并对 center 加 **0.5 m 前向偏移**（`ego_box[0] += 0.5 * cos(yaw)`）。
2. **构造 agent 框序列**：每个 agent 只取 **top-1 motion mode**（`num_motion_mode=1`），前 6 步（对齐 ego horizon），yaw 同样用差分。`det_confidence < 0.5` 的 agent 置为 `1e6`（等价于剔除）。
3. `check_collision`：两方向 corners-in-box 测试（box1 的角在 box2 内，或反之），粗略但够快。
4. 若 **所有** 模式都碰撞则跳过 rescore（没有可选的了）。

关键超参（硬编码）：`score_thresh=0.5`, `static_dis_thresh=0.5`, `dim_scale=1.1`, `num_motion_mode=1`, `offset=0.5`。

## 训练

**两阶段训练：**
- Stage 1：只训 perception（det + map + tracking）。
- Stage 2：加上 motion + planning，`load_from = ckpt/sparsedrive_stage1.pth`。

每阶段：
- **10 epochs**，AdamW，lr `3e-4`，weight decay `1e-3`，**backbone lr ×0.1**。
- Cosine anneal，500 iter warmup。
- **Batch 48，8 GPU**（每卡 6）。
- Grad clip norm 25，FP16 loss scale 32。
- 输入 704×256，resize 0.40–0.47，random flip，photometric distortion。
- Sequence-aware 采样：`sequences_split_num=2`，同 clip 内保持一致增广。

## 主要结果（nuScenes，README 更新版）

**SparseDrive-S**（ResNet-50，704×256）
- NDS 0.5257，map mAP 0.5656，AMOTA 0.372，motion EPA_car 0.492，minADE_car 0.61。
- Planning：L2 avg 0.61 m（1/2/3 s: 0.30/0.58/0.95），Col. 0.10 %。
- **9.0 FPS，训练 20 h**（vs. UniAD 144 h）。

**SparseDrive-B**：NDS 0.588，AMOTA 0.501，L2 0.58 m，Col. 0.06 %，7.3 FPS。

## 读完代码补充的判断

- **本质上是 Sparse4Dv3 + 新 planner。** 感知侧没造新算法，novelty 集中在 `MotionPlanningHead` 和把 map head 做成对称结构。
- **"Parallel" 真的就是一个 decoder**：agent 和 ego 是并列 token，cross-attn 共享。
- **"Hierarchical" = 2 层（cmd → mode）**，总共只有 18 个候选 — 这正是 SparseDriveV2 批评并扩展的点（factorized dense vocabulary）。
- **Rescore 是 ~30 行几何 + 手调阈值**，没有 learned safety critic。论文主结果里超低 collision rate 很大程度归功于这个后处理筛选 + GT command 泄露。
- **Tracking 隐式**：没有专门的 tracker 模块，纯靠 temporal query propagation + confidence decay。

# 关联阅读

- [[SparseDriveV2 2603]]
- [[Hydra-MDP 2406]]
- [[transfuser]]
- [[TransFuser 2205]]
