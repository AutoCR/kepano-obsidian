---
categories:
  - "[[Papers]]"
title: "DiffusionDrive: Truncated Diffusion Model for End-to-End Autonomous Driving"
authors:
  - Bencheng Liao
  - Shaoyu Chen
  - Haoran Yin
  - Bo Jiang
  - Cheng Wang
  - Sixu Yan
  - Xinbang Zhang
  - Xiangyu Li
  - Ying Zhang
  - Qian Zhang
  - Xinggang Wang
affiliation:
  - "Institute of Artificial Intelligence, Huazhong University of Science and Technology"
  - "School of EIC, Huazhong University of Science and Technology"
  - "Horizon Robotics"
venue: arXiv
year: 2024
doi:
url: https://arxiv.org/abs/2411.15139
pdf: https://arxiv.org/pdf/2411.15139v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - truncated diffusion
  - anchor-based planning
  - multimodal planning
  - NAVSIM
  - real-time inference
  - diffusion policy
status:
  - read
rating:
dataset:
  - NAVSIM
  - nuScenes
method:
  - truncated diffusion policy
  - anchor-conditioned denoising
  - cascade diffusion decoder
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[Hydra-MDP 2406]]"
  - "[[HMAD 2505]]"
  - "[[Diffusion]]"
---

One-line takeaway: **DiffusionDrive makes diffusion practical for E2E driving by anchoring and truncating the denoising process, yielding strong NAVSIM performance at real-time speed.**

# Core takeaways
- Makes diffusion practical for E2E driving by truncating the denoising schedule.
- Uses prior multimodal anchors so denoising starts from structured proposals rather than pure noise.
- Achieves strong NAVSIM performance with only two denoising steps and real-time speed.

# Method summary
DiffusionDrive perturbs anchor trajectories with Gaussian noise and learns a truncated denoising process conditioned on scene features. A cascade decoder progressively injects scene context, allowing the model to retain multimodal generation while greatly reducing the number of sampling steps.

# Benchmarks / results
On NAVSIM with an aligned ResNet-34 backbone, the paper reports 88.1 PDMS at 45 FPS with only two denoising steps, framing itself as a real-time diffusion planner.

# Relation to the score-based E2E AD lineage
DiffusionDrive is a major hybrid-generative counterpart to Hydra-MDP-style scoring. It keeps explicit multimodality, but moves from vocabulary scoring toward anchor-based diffusion generation.

# Caveats
- Primary evidence is concentrated on NAVSIM/open-loop evaluation.
- Performance still depends on anchor quality and truncated-schedule design.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
