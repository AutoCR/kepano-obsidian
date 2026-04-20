---
categories:
  - "[[Papers]]"
title: "FeaXDrive: Feasibility-aware Trajectory-Centric Diffusion Planning for End-to-End Autonomous Driving"
authors:
  - Baoyun Wang
  - Zhuoren Li
  - Ming Liu
  - Xinrui Zhang
  - Bo Leng
  - Lu Xiong
venue: arXiv
year: 2026
doi:
url: https://arxiv.org/abs/2604.12656
pdf: https://arxiv.org/pdf/2604.12656v1
field:
  - autonomous-driving
  - end-to-end-planning
  - diffusion-planning
keywords:
  - feasibility
  - trajectory-centric diffusion
  - curvature constraint
  - drivable-area guidance
  - GRPO
  - NAVSIM
status:
  - read
rating:
dataset:
  - NAVSIM
method:
  - trajectory-centric diffusion planning
  - adaptive curvature-constrained training
  - drivable-area guidance
  - feasibility-aware GRPO
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[SparseDriveV2 2603]]"
  - "[[Hydra-MDP 2406]]"
  - "[[SparseDrive 2405]]"
  - "[[TransFuser 2205]]"
  - "[[transfuser]]"
  - "[[navsim评估方法]]"
---

One-line takeaway: **FeaXDrive argues that diffusion-based end-to-end driving should optimize not only benchmark score but also trajectory-space feasibility, and it does so by making the clean trajectory the central object for training, sampling, and post-training optimization.**

# Core takeaways

- The paper targets a real weakness of diffusion planners: they can generate semantically plausible trajectories that are still **hard to execute physically**.
- The authors define this issue as **trajectory-space feasibility**, covering:
  - local geometric smoothness,
  - trajectory-level kinematic validity,
  - consistency with the drivable area.
- Their main design change is to move from **noise-centric diffusion** to **trajectory-centric diffusion**, i.e. predict the clean trajectory directly at each denoising step.
- On top of that, they add three feasibility-oriented components:
  1. **adaptive curvature-constrained training**,
  2. **drivable-area guidance inside reverse diffusion sampling**,
  3. **feasibility-aware GRPO fine-tuning**.
- The main empirical message is strong: FeaXDrive keeps competitive NAVSIM closed-loop performance while **substantially reducing curvature violations** and improving **drivable-area compliance**.

# What the paper is really contributing

I think the real contribution is not just “another diffusion planner,” but a **reframing of diffusion planning around feasibility**.

Many prior end-to-end planners optimize imitation or benchmark-oriented score, and diffusion models are often praised for multimodality. FeaXDrive points out that this is not enough: a planner can still output trajectories with geometric spikes, kinematic inconsistency, or off-road behavior. The paper’s answer is to inject feasibility at three levels:

- **representation level**: predict trajectories directly instead of noise,
- **training level**: penalize curvature-related infeasibility,
- **sampling level**: correct trajectories with drivable-area geometry online,
- **post-training level**: make RL reward feasibility-aware.

So the paper is best read as a **feasibility-oriented redesign of diffusion planning**, rather than a backbone scaling paper.

# Method details

## 1. Trajectory-centric diffusion planning

The planner predicts the clean future trajectory `x0` directly from the noisy state, diffusion step, and scene condition.

Why this matters:
- constraints become easier to express in trajectory space than in noise space;
- the same object can be used for both training losses and inference-time guidance;
- it gives a cleaner interface for feasibility modeling.

Compared with a standard epsilon/noise-prediction formulation, this is the conceptual foundation of the whole paper.

## 2. Adaptive curvature-constrained training

This module aims to improve **intrinsic trajectory feasibility**.

The rough pipeline is:
- smooth the predicted trajectory with a lightweight differentiable operator;
- estimate curvature in **arc-length space** rather than raw discrete time;
- apply a **speed-adaptive curvature bound**;
- penalize violations during training.

The point is not just “make trajectories smoother,” but to better align the output with **vehicle kinematics** and suppress local geometric artifacts.

This looks like the strongest single component for reducing curvature-violation rate.

## 3. Drivable-area guidance during inference

During reverse diffusion, the model does not simply denoise and stop. It also uses a local road prior:
- construct a local **signed distance field (SDF)** from the drivable area in an HD map,
- correct the predicted clean trajectory at each reverse step,
- continue sampling with the corrected trajectory.

An important detail: this is **inside the diffusion loop**, not a post-processing patch after generation. That means the guidance changes the future evolution of the denoising chain.

This is the main mechanism for improving **DAC / off-road behavior**.

## 4. Feasibility-aware GRPO fine-tuning

After imitation learning, the paper applies GRPO-based RL fine-tuning.

The reward has two parts:
- task / benchmark quality,
- feasibility reward.

The feasibility term is mainly based on the same curvature-feasibility criterion introduced earlier. This is important because a common failure mode in RL fine-tuning is that score improves while physical plausibility gets worse. FeaXDrive explicitly tries to avoid that trade-off.

# Results

## Closed-loop NAVSIM performance

Main reported results:

- **FeaXDrive-IL**:
  - **PDMS 88.7**
  - best among the IL methods listed in the table
  - **DAC 97.5**
- **FeaXDrive (with RLFT)**:
  - **PDMS 90.0**
  - slightly below ReCogDrive w/GRPO (**90.5**)
  - but **DAC 98.3 vs 96.7**, showing better drivable-area compliance

So if the benchmark lens is only final PDMS, FeaXDrive is competitive but not outright dominant. If the lens includes **physical feasibility and road compliance**, the method looks much stronger.

## Feasibility metrics

This is where the paper is most convincing.

Curvature violation rate:
- **DiffusionDrive**: **8.59%**
- **ReCogDrive-IL**: **8.05%**
- **FeaXDrive-IL**: **0.88%**

After RL fine-tuning:
- **ReCogDrive w/GRPO**: **15.5%**
- **FeaXDrive**: **2.40%**

This strongly supports the paper’s core claim: **benchmark-oriented post-training can damage feasibility unless feasibility is explicitly modeled in the reward/design**.

## Ablation

The ablation is clean and useful:
- switching from noise-centric to trajectory-centric already helps both planning score and feasibility;
- adding curvature-constrained training dramatically reduces curvature violations;
- adding drivable-area guidance significantly improves drivable-area compliance and final IL-stage performance.

This makes the method decomposition believable.

# Inference and practicality

The paper reports total latency of about **348.73 ms**.

Breakdown:
- VLM backbone: **245.33 ms**
- planner: **82.96 ms**
- local SDF construction: **16.03 ms**
- drivable-area guidance overhead: **4.41 ms**

My reading: the added feasibility machinery is **not** the runtime bottleneck. The visual-language backbone dominates latency. So the paper’s extra guidance appears practical from a systems perspective.

# How it relates to nearby papers

## vs [[Hydra-MDP 2406]]

Hydra-MDP is fundamentally a **scoring / vocabulary-based** planner with distillation and rule-based supervision. FeaXDrive is very different in planning form: it is **generative and diffusion-based**. The main similarity is that both care about planner quality beyond plain imitation, but FeaXDrive focuses much more explicitly on **trajectory feasibility during generation**.

## vs [[SparseDriveV2 2603]]

SparseDriveV2 argues that dense candidate spaces plus efficient scoring can still beat trajectory generation. FeaXDrive pushes in the opposite direction: if we do use generation, then the key missing piece is **feasibility-aware generation**, not just stronger multimodality.

So these two papers are useful conceptual opposites:
- SparseDriveV2: scaling **candidate scoring** can be enough;
- FeaXDrive: if you use diffusion generation, it should be grounded in **feasible trajectory space**.

## vs [[SparseDrive 2405]]

SparseDrive is more of a full sparse scene-centric end-to-end driving system. FeaXDrive is narrower and more planner-centric. SparseDrive’s safety flavor comes from planner design and collision-aware rescoring, while FeaXDrive’s safety/feasibility flavor comes from **trajectory-space constraints and guidance**.

## vs [[TransFuser 2205]] / [[transfuser]]

TransFuser is an earlier end-to-end planning backbone built around sensor fusion and imitation-style planning. FeaXDrive sits much later in the literature and reflects the shift from direct regression to **distribution modeling / generative planning / benchmark-aware optimization**.

# Affiliations and metadata

Authors:
- Baoyun Wang
- Zhuoren Li
- Ming Liu
- Xinrui Zhang
- Bo Leng
- Lu Xiong

Affiliation listed in the arXiv HTML metadata:
- **College of Automotive and Energy Engineering, Tongji University, Shanghai 201804, China**

Baoyun Wang and Zhuoren Li are marked as equal contributors.

# My understanding

My main takeaway is:

**FeaXDrive is important because it makes “feasibility” a first-class design target in diffusion planning.**

The paper is not claiming the largest absolute leaderboard gain. Instead, it argues that modern end-to-end planners should stop treating physical executability as an afterthought. In that sense, its contribution is conceptual as much as empirical.

If I compare it to neighboring planning papers, I would summarize the difference like this:
- some papers ask how to score candidate trajectories better;
- some ask how to generate richer trajectories;
- FeaXDrive asks how to make generated trajectories **actually feasible to execute**.

That makes it a useful reference paper whenever I want to think about the gap between **benchmark score** and **real motion validity** in diffusion-based driving planners.

# Related reading

- [[SparseDriveV2 2603]]
- [[Hydra-MDP 2406]]
- [[SparseDrive 2405]]
- [[TransFuser 2205]]
- [[navsim评估方法]]
