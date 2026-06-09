---
categories:
  - "[[Papers]]"
title: "DriveAnchor: Progressive Anchor-based Flow Learning for Autonomous Driving Planning"
authors:
  - Limin Yan
  - Haoyun Tang
  - Yutao Qiu
  - Hongqing Liu
  - Haoyu Xu
affiliation:
  - Meituan Autonomous Driving
  - Xi'an Jiaotong University
  - Beijing Institute of Technology
venue: arXiv
year: 2026
doi:
url: https://arxiv.org/abs/2606.00519
pdf: https://arxiv.org/pdf/2606.00519v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - flow matching
  - anchor vocabulary
  - trajectory generation
  - energy field
  - zeroth-order reinforcement learning
  - collision avoidance
  - real-time planning
status:
  - read
rating:
dataset:
  - internal real-world driving data
method:
  - anchor-conditioned flow matching
  - farthest-point-sampled trajectory vocabulary
  - energy-field corridor guidance
  - zeroth-order RL fine-tuning
task: autonomous driving trajectory planning
created: 2026-06-09
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[Flow matching and diffusion models]]"
  - "[[Adaptive Time Step Flow Matching 2602]]"
  - "[[GuideFlow 2511]]"
  - "[[DiffusionDriveV2 2512]]"
  - "[[HAD 2604]]"
  - "[[Generalized Trajectory Scoring 2506]]"
---

One-line takeaway: **DriveAnchor makes flow-matching planning more production-ready by replacing Gaussian noise with a fixed trajectory-anchor vocabulary, then adding corridor control and collision-safety post-training as separate stages.**

# Core takeaways

- The paper targets three deployment gaps in flow / diffusion planners: weak structural diversity, hard run-time controllability, and imitation-only safety limits.
- The main design is a **three-stage pipeline**:
  1. anchor-conditioned flow pretraining for diverse behavior coverage,
  2. Energy Field post-training for corridor / maneuver control,
  3. zeroth-order RL fine-tuning for collision avoidance.
- The anchor vocabulary contains **2,398 80-step trajectories** built by farthest-point sampling over **100M+ real-world frames**.
- The planner evaluates all anchors in one batched forward pass, then passes the top-K candidates to a downstream cost selector.
- The strongest claim is practical: **89% near-range collision reduction**, **32% mean reward gain**, and **2.06 ms/scene** inference on NVIDIA Drive Orin.

# Method details

## Stage 1: Demonstration Flow pretraining

The flow model does not start from Gaussian noise. It starts from a structured anchor prior:

$$
A = \mathrm{FPS}(\{a_k^*\}_{k=1}^{|H|}, M), \quad M=2398.
$$

Each anchor is an 80-step 2D trajectory, so the anchor dimension is \(80 \times 2 = 160\).

Training uses a noise-interpolated input:

$$
x_t = (1 - \alpha) \tilde{x}_0 + \alpha GT, \quad \alpha \sim U(0,1).
$$

The model predicts:

$$
\Delta x = GT - x_t.
$$

A notable detail: the paper says the model is trained **without explicit time input** \(t\) / \(\alpha\). At inference, \(\alpha=0\), so the model maps an anchor directly toward a final trajectory.

## Stage 2: Guided Flow post-training

Stage 2 adds an **Energy Field (EF)** module for controllability.

EF takes:

- scene encoding,
- current anchor,
- corridor polygon query,
- designated exit edge.

It predicts a displacement:

$$
\Delta v_{EF} \in \mathbb{R}^{T \times 2}.
$$

Then the shifted proposal is rematched to the nearest vocabulary anchor:

$$
\tilde{a}_{EF} = a_{anchor} + \Delta v_{EF},
$$

$$
a_{rematch} = \arg\min_{a \in A} \|\tilde{a}_{EF} - a\|_2.
$$

The rematch step matters because the flow decoder is trained on the fixed anchor vocabulary, not arbitrary continuous shifted trajectories.

EF is trained geometrically. A trajectory is good if it enters the target polygon and exits only through the specified edge. Good trajectories get zero correction; bad trajectories are pulled toward the nearest good trajectory in the vocabulary.

Important: EF is **for controllability, not collision safety**. Collision safety is handled in Stage 3.

## Stage 3: Reward-Refined Flow fine-tuning

Stage 3 fine-tunes the flow model with a black-box collision reward:

$$
R(s,a) = \min\{t: \text{collision at step }t\},
$$

or \(T+1=81\) if the trajectory is collision-free.

Because single-step flow inference is deterministic, each anchor uniquely determines one output trajectory. The paper uses this to estimate reward gradients in anchor space rather than using policy gradients.

The RL procedure is:

1. sample one anchor per scene,
2. generate trajectory \(a_i = f_\theta(x_0^{(k)}, s)\),
3. retrieve nearby anchors with KNN plus an \(\epsilon\)-ball filter,
4. query rewards for the generated trajectory and its neighbors,
5. form a reward-weighted finite-difference direction,
6. backpropagate through \(a_i\) while stopping gradient through the estimated reward direction.

The combined objective is:

$$
L(\theta) = L_{FM}(\theta) + \lambda L_{RL}(\theta).
$$

This avoids needing \(\log \pi_\theta\), ODE-to-SDE conversion, or a differentiable reward model.

# Training details

## Data

| Component | Data |
|---|---|
| Stage 1 FM pretraining | 120M real-world driving samples |
| Stage 2 EF post-training | 7.5M balanced unified driving samples |
| Anchor construction | 100M+ real-world frames, temporally disjoint from train/eval |
| Evaluation | ~2M held-out driving scenarios |

Scenarios cover highways, intersections, roundabouts, lane changes, and ramp merges.

## Training pipeline

| Stage | Trainable modules | Frozen modules | Objective |
|---|---|---|---|
| Stage 1 | FM decoder | pre-trained encoder | \(L_{FM}\) |
| Stage 2 | FM decoder + EF decoder | encoder | \(L_{FM} + L_{EF} + L_{kin}\) |
| Stage 3 | FM decoder only | EF + encoder | \(L_{FM} + \lambda L_{RL}\) |

The anchor table remains fixed throughout.

## Hyperparameters

| Item | Value |
|---|---|
| Optimizer | AdamW |
| Learning rate | 1e-5 |
| Weight decay | 0.01 |
| Batch size | 128 / GPU |
| RL loss weight \(\lambda\) | 0.2 |
| ZO neighbors \(N\) | 16 |
| Numerical stabilizer \(\epsilon_0\) | 1e-6 |
| Top-K candidates | 50 |
| Framework | PyTorch 2.3 |
| Deployment precision | float16 |
| Deployment hardware | NVIDIA Drive Orin |

## Training compute

| Stage | GPUs | Wall time | GPU-hours |
|---|---:|---:|---:|
| Stage 1 FM pretraining | 32× A100-80G | 28.2 h | 902 |
| Stage 2 EF post-training | 32× A100-80G | 16.9 h | 541 |
| Stage 3 RL fine-tuning | 64× A100-80G | 3.8 h | 243 |

## Architecture

### Shared encoder

- Inputs: ego, moving/static obstacles, road graph, traffic light.
- 6 transformer layers.
- Token dimension 384.
- 6 attention heads.
- Output projection: MLP 384 → 256.
- Output shape: `[B, 634, 256]`.

### Anchor vocabulary and projector

- Vocabulary size: 2,398 trajectories.
- Anchor dimension: 160.
- Shared trajectory projector: MLP 160 → 256.
- MLP block: Linear → LayerNorm → GELU → Linear → Dropout.

### FM decoder

- Cross-attention heads: 2.
- Model dimension: 256.
- Velocity head: MLP 256 → 256 → MLP 256 → 160.
- Supervision: SmoothL1, target = \(GT - \tilde{x}_0\).
- Inference passes: 2, called FM*2.
- Output: `[B, K, 160]` top-K trajectories.

### EF decoder

- EF input: 33 dimensions = 1 scene type + 16×2 polygon coordinates.
- Scene projector: MLP 1 → 128.
- Polygon projector: MLP 32 → 128.
- Fusion projector: concat(128,128,256) → MLP 512 → 256.
- Cross-attention heads: 2.
- Energy head: MLP 256 → 256 → MLP 256 → 160.
- Supervision: good target = 0; bad target = nearest-good trajectory minus noisy anchor.

## Kinematic losses in Stage 2

Stage 2 adds kinematic losses on the FM output only:

| Constraint | Threshold | Effective weight |
|---|---:|---:|
| Speed | 25.0 m/s | 0.1 |
| Acceleration | 4.5 m/s² | 1e-4 |
| Jerk | 4.5 m/s³ | 1e-4 |
| Curvature | 0.4339 m⁻¹ | 5e-4 |
| Lateral acceleration | 1.0 m/s² | 5e-5 |
| Lateral jerk | 1.0 m/s³ | 1e-4 |

Stage 3 does **not** include these kinematic losses; its reward signal is pure black-box collision detection.

# Results

## Main offline results

Original prior comparison:

| Metric | FM*2 | FMRL*2 |
|---|---:|---:|
| Near-range collision | 27.22% | 2.94% |
| Far-range collision | 56.54% | 7.15% |
| Mean reward | 59.22 | 78.46 |
| gt_ADE@30 | 0.23 | 0.23 |
| gt_ADE@80 | 0.42 | 0.40 |

This gives the headline result: **near-range collisions drop by 89%** and mean reward improves by **32%**, with almost unchanged imitation accuracy.

Best EF-guided setup:

| Metric | EF*1 + FMRL*2 |
|---|---:|
| min_ADE@30 | 0.22 |
| min_FDE@30 | 0.44 |
| min_ADE@80 | 0.95 |
| min_FDE@80 | 1.87 |
| Near-range collision | 1.88% |
| Far-range collision | 8.73% |
| Mean reward | 78.87 |

Top-50 selected trajectories also improve sharply: FMRL*2 reduces near collision from **13.73% to 0.47%**.

## Ablation: number of ZO neighbors

| N | Near-range | Far-range | Mean reward |
|---:|---:|---:|---:|
| 4 | 0.2752 | 0.5675 | 59.10 |
| 8 | 0.0304 | 0.0769 | 78.25 |
| 16 | 0.0294 | 0.0715 | 78.46 |

The paper argues that N=4 gives too little directional coverage in the 160-D trajectory space, while N=16 is the best reported setting.

# Relation to nearby papers

- Compared with [[Adaptive Time Step Flow Matching 2602]], DriveAnchor focuses less on adaptive ODE integration and more on structured anchors plus production post-training.
- Compared with [[GuideFlow 2511]], this paper uses a pre-generation Energy Field and anchor rematching instead of relying on differentiable guidance inside the generation process.
- Compared with [[DiffusionDriveV2 2512]] and [[HAD 2604]], the safety post-training is zeroth-order and anchor-space-based rather than standard policy-gradient-style diffusion / RL fine-tuning.
- Compared with score-based planners like [[Generalized Trajectory Scoring 2506]], DriveAnchor is more generative, but still ends with a candidate set and downstream cost selection.

# Caveats

- All experiments use an internal dataset; there is no public benchmark evaluation on nuPlan, NAVSIM, or Waymo.
- The reward is mainly collision-based. Richer comfort, rule, and interaction rewards are left for future work.
- Rare maneuver coverage is limited by the historical corpus used to build the FPS anchor vocabulary.
- The model omits explicit flow-time input, so the theoretical connection to standard flow matching is less clean.

# My understanding

DriveAnchor is best read as a practical production recipe for flow-matching planners: **use a large, fixed, kinematically plausible anchor set to make generation stable; use a separate geometry module for controllability; then use cheap zeroth-order reward gradients to make the output safer.** The important conceptual move is not just “flow matching for planning,” but the decision to make diversity, controllability, and safety independently updatable stages.

# Related reading

- [[score-based e2e autonomous driving papers 2022-2026]]
- [[Flow matching and diffusion models]]
- [[Adaptive Time Step Flow Matching 2602]]
- [[GuideFlow 2511]]
- [[DiffusionDriveV2 2512]]
- [[HAD 2604]]
- [[Generalized Trajectory Scoring 2506]]
