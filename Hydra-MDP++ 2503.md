---
categories:
  - "[[Papers]]"
title: "Hydra-MDP++: Advancing End-to-End Driving via Expert-Guided Hydra-Distillation"
authors:
  - Kailin Li
  - Zhenxin Li
  - Shiyi Lan
  - Yuan Xie
  - Zhizhong Zhang
  - Jiayi Liu
  - Zuxuan Wu
  - Zhiding Yu
  - Jose M. Alvarez
affiliation:
  - "East China Normal University"
  - "Fudan University"
  - "NVIDIA"
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2503.12820
pdf: https://arxiv.org/pdf/2503.12820v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - Hydra distillation
  - expert-guided scoring
  - traffic-light compliance
  - lane keeping
  - EPDMS
  - NAVSIM
status:
  - read
rating:
dataset:
  - NAVSIM
method:
  - teacher-student Hydra distillation
  - multi-head decoder
  - extended-metric supervision
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[Hydra-MDP 2406]]"
  - "[[DriveSuprim 2506]]"
---

One-line takeaway: **Hydra-MDP++ strengthens Hydra-MDP by expanding simulator-teacher supervision beyond basic safety/progress metrics to cover traffic-rule and comfort failures missed by earlier NAVSIM teachers.**

# Core takeaways
- Preserves the original Hydra-MDP score-based paradigm while improving the teachers.
- Adds supervision for traffic-light compliance, lane keeping, and extended comfort.
- Shows that richer rule-based teachers improve both PDMS and extended metrics.

# Method summary
Hydra-MDP++ keeps the Hydra teacher-student architecture and multi-head planning formulation, but broadens the supervision space with extra rule-based teachers for traffic-light compliance, lane keeping, and comfort. This improves the learned scorer without fundamentally changing the planning architecture.

# Benchmarks / results
On NAVSIM, the paper reports up to 91.0 PDMS in a scaled configuration and improved extended metrics such as EPDMS when richer teacher targets are included.

# Relation to the score-based E2E AD lineage
Hydra-MDP++ is the clearest refinement of the original Hydra-MDP formulation and a core strict-score-based paper in this lineage.

# Caveats
- Still depends on hand-designed simulator teachers and benchmark-specific targets.
- Candidate quality remains limited by the proposal set.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
