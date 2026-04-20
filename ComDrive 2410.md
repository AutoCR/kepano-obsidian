---
categories:
  - "[[Papers]]"
title: "ComDrive: Comfort-Oriented End-to-End Autonomous Driving"
authors:
  - Junming Wang
  - Xingyu Zhang
  - Zebin Xing
  - Songen Gu
  - Xiaoyang Guo
  - Yang Hu
  - Ziying Song
  - Qian Zhang
  - Xiaoxiao Long
  - Wei Yin
venue: arXiv
year: 2024
doi:
url: https://arxiv.org/abs/2410.05051
pdf: https://arxiv.org/pdf/2410.05051v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - comfort-oriented driving
  - diffusion planner
  - trajectory scoring
  - temporal consistency
  - nuScenes
  - multimodal planning
status:
  - read
rating:
dataset:
  - nuScenes
method:
  - conditional diffusion planner
  - dual-stream adaptive trajectory scorer
  - sparse perception
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[SparseDrive 2405]]"
  - "[[Hydra-MDP 2406]]"
---

One-line takeaway: **ComDrive extends the selection-based E2E recipe with diffusion generation and comfort-aware scoring, explicitly targeting smooth, temporally consistent driving rather than only safety.**

# Core takeaways
- Uses a conditional diffusion planner to generate temporally consistent multimodal trajectories.
- Adds a dual-stream adaptive scorer that balances safety and comfort when selecting the final trajectory.
- Treats comfort as a first-class objective instead of only optimizing safety or imitation distance.
- Reports gains in both comfort and collision reduction over prior E2E baselines.

# Method summary
ComDrive first extracts sparse scene structure, then conditions a DDPM-style planner on that representation and trajectory history to generate multiple future ego trajectories. A dual-stream adaptive scorer combines safety and comfort signals to rank those trajectories, so the final output is selected rather than directly regressed.

# Benchmarks / results
On nuScenes validation, the paper reports about 0.58 average L2 and 0.06% collision rate for the larger model, while also claiming a 17% comfort improvement over UniAD and a 25% collision reduction versus SparseDrive.

# Relation to the score-based E2E AD lineage
ComDrive is a hybrid-generative successor to SparseDrive/Hydra-style selection: it keeps a final scoring-and-selection stage, but replaces static candidate heads with diffusion-generated multimodal trajectories.

# Caveats
- Main evidence is still open-loop nuScenes evaluation rather than stronger closed-loop deployment.
- Comfort gains depend on the chosen scorer design and metric definitions.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
