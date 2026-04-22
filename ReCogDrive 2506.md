---
categories:
  - "[[Papers]]"
title: "ReCogDrive: A Reinforced Cognitive Framework for End-to-End Autonomous Driving"
authors:
  - Yongkang Li
  - Kaixin Xiong
  - Xiangyu Guo
  - Fang Li
  - Sixu Yan
  - Gangwei Xu
  - Lijun Zhou
  - Long Chen
  - Haiyang Sun
  - Bing Wang
  - Kun Ma
  - Guang Chen
  - Hangjun Ye
  - Wenyu Liu
  - Xinggang Wang
affiliation:
  - "Huazhong University of Science and Technology"
  - "Xiaomi EV"
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2506.08052
pdf: https://arxiv.org/pdf/2506.08052v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - VLM cognition
  - diffusion planner
  - GRPO
  - NAVSIM
  - Bench2Drive
  - long-tail driving
status:
  - read
rating:
dataset:
  - NAVSIM
  - Bench2Drive
  - DriveBench
  - DriveLM
method:
  - cognitive VLM pretraining
  - diffusion planner
  - DiffGRPO
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[DiffusionDrive 2411]]"
  - "[[FeaXDrive 2604]]"
---

One-line takeaway: **ReCogDrive couples VLM-based driving cognition with a diffusion planner and RL fine-tuning to get strong scene understanding plus SOTA closed-loop planning.**

# Core takeaways
- Uses a VLM to build driving cognition priors rather than outputting textual actions directly.
- Feeds those priors into a diffusion planner for continuous trajectory generation.
- Adds RL post-training to improve safety and comfort.

# Method summary
ReCogDrive first trains a VLM using a staged cognitive supervision pipeline, then injects the learned priors into a diffusion planner that outputs continuous trajectories. A final DiffGRPO stage fine-tunes the planner toward safer and more comfortable behavior.

# Benchmarks / results
The paper reports about 90.8 PDMS on NAVSIM and 71.36 driving score with 45.45% success rate on Bench2Drive, while also showing stronger qualitative scene understanding on DriveBench/DriveLM.

# Relation to the score-based E2E AD lineage
ReCogDrive is a hybridization step relative to the score-based lineage: it is less about better scoring alone and more about using VLM cognition to improve the proposal generator and post-training stage.

# Caveats
- Pipeline complexity is high.
- Gains partly rely on custom cognition data generation and filtering.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
