---
created: 2026-06-15
tags:
  - autonomous-driving
  - flow-matching
  - paper-review
---

# Flow matching for AV 2026 总结

本文梳理 [[Flow matching for AV 2026]] 中列出的 6 篇论文。严格来说，`2501.15564` 是 diffusion planner，不是 flow matching；其余 5 篇都直接使用 flow matching / MeanFlow / flow policy。

## 论文列表

| arXiv                                          | 模型                    | 任务             | 生成变量                  | 主要目的                                                             |
| ---------------------------------------------- | --------------------- | -------------- | --------------------- | ---------------------------------------------------------------- |
| [2501.15564](https://arxiv.org/pdf/2501.15564) | Diffusion Planner     | 闭环规划 + 周车预测    | ego 轨迹 + 邻车轨迹         | 用 diffusion 建模多模态行为，并用 classifier guidance 加安全/舒适/速度约束           |
| [2606.00519](https://arxiv.org/pdf/2606.00519) | DriveAnchor           | 生产级规划          | anchor 条件下的轨迹集合       | 用结构化 anchor prior + Energy Field + zeroth-order RL 兼顾多样性、可控性、安全性 |
| [2605.19771](https://arxiv.org/pdf/2605.19771) | BeyondDrive           | 安全模仿学习         | hard negative 轨迹      | 用 flow matching 生成“接近专家但不安全”的负样本，训练 planner 远离危险模仿               |
| [2603.02613](https://arxiv.org/pdf/2603.02613) | DACER-F               | 在线 RL 控制策略     | 动作 `a`                | 用 flow matching 替代 diffusion policy，实现单步低延迟 generative policy    |
| [2602.20060](https://arxiv.org/pdf/2602.20060) | MeanFuser             | 端到端规划          | 多模态轨迹 proposal + 最终轨迹 | 用 Gaussian Mixture Noise + MeanFlow 一步生成多模态轨迹                    |
| [2602.10285](https://arxiv.org/pdf/2602.10285) | Adaptive Time Step FM | 交互式运动规划 + 周车预测 | ego 轨迹 + 周车未来轨迹       | 用条件 flow matching + 方差估计自适应选择 ODE 步数                             |

## 1. Diffusion Planner

![[Attachments/flow-matching-av-2026/figures/diffusion-planner-architecture.png]]

**输入**：自车历史状态、周围车辆历史/检测结果、矢量化地图、route/目标相关信息。论文强调当前实现依赖 vectorized map 和检测到的邻车，而不是直接端到端图像输入。

**输出**：自车未来规划轨迹，同时预测若干邻车未来轨迹。nuPlan 设置中规划约 8 秒、10 Hz 的未来轨迹，并支持闭环 replanning。

**实现细节**：

- 主体是 transformer / DiT 风格的 diffusion planner，在轨迹空间迭代去噪。
- 统一建模 prediction 和 planning：decoder 不只输出 ego trajectory，也输出 neighboring vehicles prediction，便于在 guidance 中计算交互安全。
- 训练目标是标准 diffusion 噪声/轨迹恢复目标，学习专家轨迹分布。
- 推理阶段支持 classifier guidance。guidance energy 包括 collision avoidance、drivable area、target speed、comfort 等，不需要重新训练主模型。
- 通过高阶 ODE solver 与低温采样加速/稳定推理，论文报告单 A6000 约 20 Hz。

**定位**：这是这组论文里的 diffusion baseline/前驱思路：优点是 guidance 灵活、能建模多模态；缺点是仍需要多步采样，实时性和部署复杂度弱于单步 flow/MeanFlow 类模型。

## 2. DriveAnchor

![[Attachments/flow-matching-av-2026/figures/driveanchor-framework.png]]

**输入**：感知/场景编码特征、结构化 anchor vocabulary、可选 corridor polygon 控制条件、anchor 邻域信息。论文使用 2,398 个由 farthest-point sampling 构造的 trajectory shape anchors。

**输出**：每个 anchor 对应一条候选轨迹；推理时对完整 anchor vocabulary 并行前向，生成多样轨迹集合，再由下游 cost selector 选择 top-K。

**实现细节**：

- Stage 1 Demonstration Flow Pretraining：把高斯 prior 替换成 anchor prior。FM 学习从 anchor/noisy anchor 到专家轨迹的条件映射，用 anchor 覆盖行为多样性。
- Stage 2 Guided Flow Post-training：增加 Energy Field (EF)，根据静态道路几何和 corridor polygon 先移动 anchor，再交给 FM 生成轨迹。新 corridor preset 主要更新 EF，不必重训 FM。
- Stage 3 Reward-Refined Flow Fine-tuning：利用单步 FM 的确定性特征，把“每个 anchor 唯一决定输出轨迹”转化为 anchor space 的 zeroth-order 方向搜索，优化碰撞规避 reward，不需要 log-likelihood 或 ODE-to-SDE。
- 生产部署目标明确：NVIDIA Drive Orin float16，论文报告约 2.06 ms 推理。

**定位**：DriveAnchor 是最工程化的方案。它不把多模态性完全交给随机噪声，而是显式用 anchor vocabulary 管理行为覆盖；再用 EF 做可控性，用 RL fine-tuning 做安全对齐。

## 3. BeyondDrive

![[Attachments/flow-matching-av-2026/figures/beyonddrive-framework.png]]

**输入**：训练集专家轨迹、场景 context feature、baseline planner 的预测轨迹。flow matching 生成器以 context 为条件，围绕专家轨迹生成 candidate negatives。

**输出**：hard negative trajectories，即几何上接近专家但安全指标较差的轨迹；最终输出仍由被训练的 planner 产生。

**实现细节**：

- Stage 1 训练 flow matching negative trajectory generator。使用 classifier-free guidance 和 noise standard deviation scaling，在专家附近采样多样 candidate。
- 对候选轨迹做 safety-aware + distance-aware filtering：保留“不安全但接近专家”的 hard negatives，避免负样本过远导致训练信号失真。
- Stage 2 训练 planner：原始 imitation loss 把预测拉向专家；Repulsive Distance Loss 把预测从 hard negatives 推开。
- 可插到不同 planner 上：论文主要作用于 Latent TransFuser，也测试多模态规划器和 world model 类方法。

**定位**：BeyondDrive 的 flow matching 不是最终 planner，而是“训练数据增强/安全边界挖掘器”。它补足纯 imitation learning 只知道“什么是好轨迹”、不知道“什么是危险相似轨迹”的问题。

## 4. DACER-F

> 原论文没有给出单独的模型结构图，只有 Langevin Optimize Action 和 DACER-F 两个算法表。下图是按论文 Algorithm 1/2 与 Section IV 重绘的结构示意图。

![[Attachments/flow-matching-av-2026/figures/dacer-f-redrawn-architecture.png]]

**输入**：RL 状态 `s`、从 replay buffer 取出的动作 `a_B`、高斯 prior 动作 `a_0`、Q 网络梯度、环境 reward/transition。

**输出**：单步生成的连续控制动作 `a`，以及训练后的 flow policy `v_theta` 和双 Q critic。

**实现细节**：

- 把 policy 表示为 conditional flow model：从简单 prior `a_0 ~ N(0,I)` 生成动作。
- 在线 RL 没有固定专家数据分布，因此论文用 Q 函数定义动态目标分布：`p(a|s) ∝ exp(Q(s,a)/alpha)`。
- 用 Langevin dynamics 从 replay buffer 动作出发，沿 `grad_a Q` 优化并加噪，得到 target action `a*`。
- flow policy 用 CFM loss 学习 `a_0 -> a*` 的映射，同时 actor loss 中包含 `-Q(s, pi_theta(s))` 来提升回报。
- critic 是 double Q-network + target network，整体类似 actor-critic 训练。
- 推理只需一次 flow policy 前向，论文报告约 0.28 ms，明显快于多步 diffusion actor。

**定位**：DACER-F 与其他轨迹规划论文不同，它不是直接输出 8 秒轨迹，而是在线 RL policy 输出动作。它展示了 flow matching 在低延迟控制策略中的用法。

## 5. MeanFuser

![[Attachments/flow-matching-av-2026/figures/meanfuser-architecture.png]]

**输入**：多相机图像/BEV 感知特征、自车状态，经过 encoder 得到 context feature；训练时带辅助 map reconstruction supervision。

**输出**：多模态 ego trajectory proposals，以及由 Adaptive Reconstruction Module 选择或重构出的最终轨迹。

**实现细节**：

- 用 Gaussian Mixture Noise (GMN) 替代离散 anchor vocabulary。每个高斯分量对应一种连续轨迹区域，可并行采样 conservative/aggressive 等风格。
- 使用 MeanFlow Identity：模型学习区间平均速度场 `u_theta`，而不是 vanilla flow matching 的瞬时速度场 `v_theta`。
- 一步采样：从 GMN 采样噪声后，经 MeanFlow 直接生成候选轨迹，减少 ODE solver 数值误差和推理时间。
- ARM 是轻量 attention/reconstruction 模块：如果候选 proposal 中已有满意轨迹则隐式选择；否则基于 proposals 重构新轨迹。
- 不依赖 PDM Score supervision，在 NAVSIM closed-loop benchmark 上强调效率与鲁棒性。

**定位**：MeanFuser 的关键是把“多模态 prior”从离散 anchor 改成连续 GMN，再用 MeanFlow 实现单步生成。它比 GoalFlow/DiffusionDrive 这类 anchor-guided 方法更少受离散 vocabulary 覆盖限制。

## 6. Adaptive Time Step Flow Matching

![[Attachments/flow-matching-av-2026/figures/adaptive-time-step-flow-matching-architecture.png]]

**输入**：自车过去 1 秒轨迹、最多 5 个 10m 范围内周车历史、HD map polyline tensor、目标 ego pose。WOMD 设置中状态包含 `(x, y, v_x, v_y)`，未来 horizon 为 80 step，即 8 秒。

**输出**：自车 8 秒规划轨迹 `S_plan_e`，同时输出周围 agent 的 8 秒预测轨迹 `S_pred_obj`。

**实现细节**：

- scene encoder 复用 Motion Transformer encoder 思路：agent history、map polyline 和 goal pose 经 MLP/Transformer 融合成 context vector `c`。
- flow generator 是 U-Net，base width 128，dim multipliers `(1,2,4)`，在 `(x,y,v_x,v_y)` 轨迹空间预测 velocity field。
- 训练时采样 `z_0 ~ N(0,I)` 和 ground-truth future trajectory `z_1`，用线性插值 `z_t=(1-t)z_0+t z_1`，目标速度是 `z_1-z_0`。
- 方差头 `sigma_phi` 接在 U-Net bottleneck features 上，是 4 层 MLP、hidden dim 512、SiLU，估计局部不确定性。
- 推理时根据 `sigma_phi` 自适应设置积分步长：高不确定场景用更小步长，低不确定场景用更大步长，平均 NFE 约 4.7。
- 最后用凸 QP 后处理 ego 轨迹，约束 lateral acceleration、angular velocity 和 final goal tolerance，OSQP 约 1 ms。

**定位**：这是最标准的“条件 flow matching 轨迹生成器”。和 MeanFuser/DriveAnchor 的单步思路不同，它保留 ODE integration，但把步数从固定超参改成模型预测的不确定性调度。

## 实现异同对比

| 维度 | Diffusion Planner | DriveAnchor | BeyondDrive | DACER-F | MeanFuser | Adaptive Time Step FM |
|---|---|---|---|---|---|---|
| 生成方法 | Diffusion denoising | Anchor-conditioned FM | FM negative generator | Flow policy | MeanFlow | Conditional FM |
| prior | Gaussian noise | 2,398 anchors | noise around expert | Gaussian action prior | Gaussian Mixture Noise | Gaussian trajectory prior |
| 是否单步 | 否，多步 | 是，单步 FM | 采样可多候选 | 是，单步 policy | 是，一步 MeanFlow | 否，自适应少步 ODE |
| 主要输入 | vector map + agents + route | scene feature + anchor + corridor | expert/context + planner output | RL state + replay action | camera/BEV + ego state | agent history + map + goal |
| 主要输出 | ego + neighbor trajectories | top-K candidate trajectories | hard negatives | control action | proposals + final trajectory | ego plan + neighbor predictions |
| 多模态来源 | diffusion sampling | anchor vocabulary | CFG + noise scaling | action prior + Langevin target | GMN components | Gaussian prior + ODE sampling |
| 安全机制 | classifier guidance | RL reward + cost selector | repulsive loss from unsafe negatives | Q-guided action strengthening | ARM + learned proposals | QP dynamic constraints |
| 工程重点 | guidance 灵活 | 生产部署、2 ms 级 | 安全训练数据 | RL 低延迟动作 | NAVSIM 一步多模态 | 20 Hz 自适应步数 |

## 关键结论

1. **prior 设计是核心差异**。DriveAnchor 用离散但结构化的 anchor prior；MeanFuser 用连续 GMN；Adaptive FM 用标准 Gaussian；BeyondDrive 用围绕专家轨迹的 guided sampling；DACER-F 用动作 prior。

2. **单步化是 2026 工作的主线**。DriveAnchor、MeanFuser、DACER-F 都明确追求 one-step inference，避免 diffusion/ODE 多步推理。Adaptive FM 走另一条路：保留积分，但把 NFE 自适应压到约 5。

3. **安全不再只靠 imitation loss**。Diffusion Planner 用 guidance energy；DriveAnchor 用 RL reward fine-tuning；BeyondDrive 用 hard negatives + repulsive loss；Adaptive FM 用 QP 后处理；DACER-F 用 Q-function 构造高价值目标分布。

4. **输出层级不同，不能简单横向比较**。MeanFuser/DriveAnchor/Adaptive FM 是 trajectory planner；BeyondDrive 是训练框架；DACER-F 是 action-level RL policy；Diffusion Planner 是多步 diffusion planner。

5. **最接近量产部署的是 DriveAnchor**。它显式讨论 Drive Orin、float16、2.06 ms、真实车辆验证；MeanFuser 和 Adaptive FM 更偏 benchmark planner；BeyondDrive 更像可迁移训练策略；DACER-F 更偏 RL 算法。

