---
categories:
  - "[[Papers]]"
title: "Generalized Trajectory Scoring for End-to-end Multimodal Planning"
authors:
  - Zhenxin Li
  - Wenhao Yao
  - Zi Wang
  - Xinglong Sun
  - Joshua Chen
  - Nadine Chang
  - Maying Shen
  - Zuxuan Wu
  - Shiyi Lan
  - Jose M. Alvarez
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2506.06664
pdf: https://arxiv.org/pdf/2506.06664v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - trajectory scoring
  - diffusion proposals
  - static vocabulary
  - vocabulary generalization
  - NAVSIM v2
  - robustness
status:
  - read
rating:
dataset:
  - NAVSIM
  - Navhard
method:
  - hybrid scorer over static and dynamic candidates
  - vocabulary generalization
  - sensor augmentation
  - refinement decoder
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[SparseDriveV2 2603]]"
  - "[[CdDrive 2602]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
---

One-line takeaway: **GTRS argues that robust E2E planning comes from jointly scoring coarse static anchors and fine diffusion proposals, yielding the winning NAVSIM v2 challenge solution.**

# Core takeaways
- Unifies large static-vocabulary scoring with dynamic proposal scoring.
- Trains the scorer to generalize across variable candidate subsets.
- Targets NAVSIM v2 / Navhard where robustness matters more than easy leaderboard gains.

# Method summary
The method combines a diffusion-based proposal generator with a scorer trained over a very dense static trajectory vocabulary. Training includes candidate subsampling and refinement so the scorer can handle both static and dynamic candidate pools at inference.

# Benchmarks / results
On Navhard, the challenge-winning ensemble reaches about 49.4 EPDMS, close to privileged PDM-Closed. Single-model variants also report strong EPDMS in the mid-40s.

# Relation to the score-based E2E AD lineage
This paper bridges Hydra-style vocabulary scoring and later hybrid systems like CdDrive by showing the value of scoring both static and dynamic candidates in a common framework.

# Caveats
- Top performance partly depends on ensembling.
- Pipeline complexity is higher than pure scoring-only approaches.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
