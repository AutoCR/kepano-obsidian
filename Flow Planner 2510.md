---
categories:
  - "[[Papers]]"
title: "Flow Matching-Based Autonomous Driving Planning with Advanced Interactive Behavior Modeling"
authors:
  - Tianyi Tan
  - Yinan Zheng
  - Ruiming Liang
  - Zexu Wang
  - Kexin Zheng
  - Jinliang Zheng
  - Jianxiong Li
  - Xianyuan Zhan
  - Jingjing Liu
affiliation:
  - "Institute for AI Industry Research (AIR), Tsinghua University"
  - "Institute of Automation, Chinese Academy of Sciences"
  - "The Chinese University of Hong Kong"
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2510.11083
pdf: https://arxiv.org/pdf/2510.11083v1
field:
  - autonomous-driving
  - learning-based-planning
  - generative-planning
keywords:
  - flow matching
  - classifier-free guidance
  - trajectory tokenization
  - interaction modeling
  - nuPlan
  - interPlan
status:
  - read
rating:
dataset:
  - nuPlan
  - interPlan
method:
  - flow matching planner
  - fine-grained trajectory tokenization
  - scale-adaptive attention
  - classifier-free guidance
task: autonomous driving motion planning
created: 2026-05-26
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[ComDrive 2410]]"
  - "[[GoalFlow 2503]]"
  - "[[GuideFlow 2511]]"
  - "[[VAE and Diffusion]]"
---

One-line takeaway: **Flow Planner replaces one-shot imitation with a flow-matching trajectory generator whose tokens explicitly interact with nearby scene tokens, making it stronger on closed-loop interactive planning.**

# Core takeaways
- The paper targets the weak point of learning-based planners: interactive behavior in dense, unusual, or high-conflict traffic.
- It splits the future ego trajectory into overlapping segment tokens instead of compressing the whole plan into one token.
- It fuses ego trajectory tokens with lane and agent tokens through distance-aware spatiotemporal attention.
- It trains a flow-matching generator and uses classifier-free guidance (CFG) to amplify scene-conditioned behavior during inference.
- The method is evaluated on nuPlan and the more interaction-heavy interPlan benchmark.

# Model structure

![[Attachments/flow-planner-2510/figure1-model-structure.png]]

The model has three important layers of design:

1. **Fine-grained trajectory tokenization** decomposes the noisy ego future into overlapping chunks.
2. **Interaction-enhanced spatiotemporal fusion** lets ego, lane, and neighbor tokens exchange information with distance-aware attention.
3. **Flow matching with CFG** samples from noise toward a clean ego trajectory, while increasing the influence of scene conditions at inference.

# Method details

## Flow-matching formulation
The planning problem is written as conditional generation:

$$
\tau_0 \sim \mathcal{N}(0, I), \quad \tau_1 \sim q(\tau_1 \mid C)
$$

where `C` is the scene condition and `τ₁` is the target ego future trajectory. The model learns the vector field along a probability path between noise and data:

$$
\mathcal{L} = \mathbb{E}_{t, \tau_0, \tau_1}\left\|v_\theta(\tau_t, t \mid C) - v_t(\tau_t, t)\right\|^2
$$

In the released implementation, the model predicts `x_start` / clean trajectory segments, then converts that prediction into velocity for ODE sampling.

## Trajectory tokenization
Official config:

```yaml
future_len: 80
action_len: 20
action_overlap: 10
state_dim: 4
```

So the 80-step future plan is represented as 7 overlapping tokens:

```text
token 0: steps  0-19
token 1: steps 10-29
token 2: steps 20-39
...
token 6: steps 60-79
```

Each token predicts a `20 × 4` segment, where the 4 state dimensions are normalized `(x, y, cos(heading), sin(heading))`.

## Interaction fusion
Scene tokens and ego tokens are concatenated and fused. The attention score is adjusted by spatial distance:

$$
\text{Attention}=\operatorname{Softmax}\left(\frac{QK^T}{\sqrt{d}} - \lambda D\right)V
$$

`D` is the pairwise Euclidean distance between tokens. Intuitively, far-away agents and lanes receive lower attention unless the model learns they matter.

## Classifier-free guidance
The model is trained with condition masking. At inference, it combines conditioned and unconditioned velocity fields:

$$
\tilde{v}_t(\tau_t,t\mid C) = (1-\omega)v_t(\tau_t,t) + \omega v_t(\tau_t,t\mid C)
$$

The released config uses `cfg_weight: 1.8`, 4 midpoint ODE steps, and masks neighbor information for the unconditioned branch.

# How the final layer outputs the ego plan
The final layer does **not** output ego planning tokens. It receives the fused ego planning tokens and turns each token into one trajectory segment.

Implementation path:

```text
flow_planner/model/flow_planner_model/decoder.py
flow_planner/model/modules/decoder_modules.py
```

Shape flow:

```text
x_token:    (B, 7, 256)   # fused ego planning tokens
y:          (B, 7, 256)   # time + route + action position + CFG embedding
prediction: (B, 7, 80)    # flat 20-step segment
reshape:    (B, 7, 20, 4)
assemble:   (B, 1, 80, 4)
```

The final layer applies adaptive LayerNorm conditioning:

```python
shift, scale = adaLN_modulation(y).chunk(2, dim=-1)
x = LayerNorm(x_token)
x = x * (1 + scale) + shift
prediction = MLP(x)
```

Then overlapping segments are averaged / assembled into the full future trajectory.

# Training / inference notes
- Training uses the nuPlan 1M split.
- The appendix reports 8×NVIDIA A6000 GPUs, batch size 2048, AdamW, learning rate `5e-4`, and EMA.
- Inference uses a midpoint ODE solver with 4 simulation steps.
- The planner outputs normalized ego future states and then postprocesses them back to physical coordinates.

# Results and significance
- Flow Planner reports strong closed-loop performance on nuPlan Val14 / Test14-style settings.
- The more interesting result is on interPlan, where the paper emphasizes high-interaction scenes such as nudging around crashed vehicles, high traffic density, and jaywalking.
- Ablations suggest that trajectory tokenization, distance-aware attention, separate adaLN/FFN for heterogeneous modalities, and CFG each contribute to the final performance.

# Caveats
- The method still depends on processed perception / vectorized scene inputs rather than raw sensor end-to-end learning.
- CFG improves interaction modeling but adds an extra conditioned/unconditioned evaluation path during inference.
- The paper is strong on closed-loop benchmarks, but real vehicle deployment latency and robustness are not the main focus.

# My understanding
Flow Planner is best read as a **generative counterpart to score-based / candidate planning**: it does not score a fixed vocabulary, but it keeps the same concern with multimodal futures and scene-conditioned quality. Its most practical idea is the overlapping trajectory-token representation: each token is small enough to interact with scene context, while overlap preserves smoothness across the full ego plan.

# Related reading
- [[score-based e2e autonomous driving papers 2022-2026]]
- [[ComDrive 2410]]
- [[GoalFlow 2503]]
- [[GuideFlow 2511]]
- [[VAE and Diffusion]]
