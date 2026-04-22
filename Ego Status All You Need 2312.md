---
categories:
  - "[[Papers]]"
title: "Is Ego Status All You Need for Open-Loop End-to-End Autonomous Driving?"
authors:
  - Zhiqi Li
  - Zhiding Yu
  - Shiyi Lan
  - Jiahan Li
  - Jan Kautz
  - Tong Lu
  - Jose M. Alvarez
affiliation:
  - NVIDIA
  - Nanjing University
venue: arXiv
year: 2023
doi:
url: https://arxiv.org/abs/2312.03031
pdf: https://arxiv.org/pdf/2312.03031v2.pdf
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - nuscenes
  - open-loop-planning
  - ego-status
  - benchmark-analysis
  - evaluation-metrics
status:
  - read
rating:
dataset:
  - nuScenes
method:
  - Ego-MLP
  - BEV-Planner
  - curb collision rate
  - benchmark analysis
task: open-loop end-to-end autonomous driving planning
created: 2026-04-22
updated: 2026-04-22
tags:
  - paper
  - nvidia
related:
  - "[[Hydra-MDP 2406]]"
  - "[[OmniDrive 2405]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
---

One-line takeaway: **This paper argues that nuScenes open-loop planning is biased enough that ego-state shortcuts can rival perception-heavy planners, so benchmark conclusions should be treated cautiously.**

# Core takeaways
- The paper is primarily a **benchmark critique** rather than a new production-grade planner.
- On nuScenes open-loop planning, simple ego-state baselines remain surprisingly competitive with perception-heavy methods.
- The authors show that common metrics under-penalize unsafe road-boundary violations.
- They introduce **curb collision rate (CCR)** to better measure road compliance.
- The analysis suggests that some published “end-to-end” gains are driven more by ego dynamics priors than by genuine scene understanding.

# Model details
## Ego-MLP
- Input: ego velocity, acceleration, yaw, and driving command.
- Architecture: a lightweight MLP that directly predicts the future ego trajectory.
- Key point: it removes historical ego trajectory inputs used by prior MLP baselines to avoid possible label leakage.

## BEV-Planner
- Image backbone: ResNet-50.
- Multi-camera images are converted into BEV features.
- The current BEV feature is concatenated with the previous 4 BEV features without explicit alignment.
- A BEV encoder compresses the features to 256 channels.
- A learnable **ego query** cross-attends to BEV tokens.
- The refined ego query is passed through an MLP trajectory head.
- Main supervision is a simple **L1 trajectory loss**.

## Evaluation contribution
- The main methodological contribution is **CCR**, which rasterizes road boundaries and checks whether predicted trajectories intersect the curb / drivable boundary.
- The paper also perturbs image inputs and ego status to test whether planning outputs actually depend on perception.

# Training / inference notes
- Input resolution: 256×704.
- BEV grid: 128×128 over roughly a 50 m range.
- Temporal input: current frame plus 4 past BEV frames.
- Training setup reported in the paper: 12 epochs on 8×V100, batch size 32, learning rate 1e-4.
- The authors explicitly note that BEV-Planner is a simple analysis baseline, not a deployment-ready AV stack.

# Benchmarks / results
- Ego-only planning can remain near the level of stronger published methods on standard nuScenes open-loop metrics.
- The paper shows that some post-processing improves average L2 while worsening road compliance, which CCR reveals clearly.
- Robustness analysis shows some planners react much more strongly to ego-speed perturbations than to major image corruption.

# Caveats
- The conclusions are tightly tied to **nuScenes open-loop evaluation**.
- CCR is useful but still imperfect; near-stationary trajectories can game it.
- The paper does not solve the benchmark issue completely; it mainly demonstrates the issue convincingly.

# My understanding
This paper is the “reset” point of NVIDIA’s recent line. Before proposing stronger planners, it asks whether the benchmark is actually measuring scene understanding. That critique helps explain why later NVIDIA papers move toward better metrics, closed-loop-aligned objectives, and richer supervision.

# Related reading
- [[Hydra-MDP 2406]]
- [[OmniDrive 2405]]
- [[NVIDIA end-to-end AV review 2023-now]]
