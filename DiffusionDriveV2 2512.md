---
categories:
  - "[[Papers]]"
title: "DiffusionDriveV2: Reinforcement Learning-Constrained Truncated Diffusion Modeling in End-to-End Autonomous Driving"
authors:
  - Jialv Zou
  - Shaoyu Chen
  - Bencheng Liao
  - Zhiyu Zheng
  - Yuehao Song
  - Lefei Zhang
  - Qian Zhang
  - Wenyu Liu
  - Xinggang Wang
affiliation:
  - "School of Electronic Information and Communications, Huazhong University of Science and Technology"
  - "Institute of Artificial Intelligence, Huazhong University of Science and Technology"
  - "Horizon Robotics"
  - "School of Computer Science, Wuhan University"
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2512.07745
pdf: https://arxiv.org/pdf/2512.07745v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - truncated diffusion
  - RL-constrained generation
  - GRPO
  - multimodality
  - NAVSIM
  - anchor intentions
status:
  - read
rating:
dataset:
  - NAVSIM v1
  - NAVSIM v2
method:
  - anchor-based truncated diffusion
  - multiplicative noise
  - intra-anchor GRPO
  - inter-anchor truncated GRPO
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[DiffusionDrive 2411]]"
  - "[[HAD 2604]]"
---

One-line takeaway: **DiffusionDriveV2 uses anchor-aware RL on top of truncated diffusion to preserve multimodality while sharply improving trajectory quality.**

# Core takeaways
- Addresses the diversity-vs-quality tradeoff left by the original DiffusionDrive.
- Uses anchor-aware RL so different maneuvers are optimized without collapsing into each other.
- Pushes NAVSIM performance to new records for the reported setting.

# Method summary
DiffusionDriveV2 retains the anchor-based truncated-diffusion formulation of DiffusionDrive, but adds reinforcement learning with specialized advantage estimation both within anchors and across anchors. This improves quality while preserving maneuver diversity.

# Benchmarks / results
The paper reports 91.2 PDMS on NAVSIM v1 and 85.5 EPDMS on NAVSIM v2 for an aligned ResNet-34 setting, together with diversity/quality analyses supporting the RL design.

# Relation to the score-based E2E AD lineage
This paper is a direct diffusion-based continuation of the score-based E2E AD line: it keeps explicit action-space structure via anchors while improving optimization with RL.

# Caveats
- Evidence is concentrated on NAVSIM.
- Anchor design remains an important structural prior.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
