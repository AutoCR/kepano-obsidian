---
categories:
  - "[[Papers]]"
title: "Hydra-NeXt: Robust Closed-Loop Driving with Open-Loop Training"
authors:
  - Zhenxin Li
  - Shihao Wang
  - Shiyi Lan
  - Zhiding Yu
  - Zuxuan Wu
  - Jose M. Alvarez
affiliation:
  - "Fudan University"
  - "The Hong Kong Polytechnic University"
  - "NVIDIA"
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2503.12030
pdf: https://arxiv.org/pdf/2503.12030v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - closed-loop robustness
  - open-loop training
  - trajectory refinement
  - control decoder
  - Bench2Drive
  - Hydra lineage
status:
  - read
rating:
dataset:
  - Bench2Drive
  - NAVSIM
method:
  - multi-branch planning
  - control decoder
  - trajectory refinement module
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

One-line takeaway: **Hydra-NeXt extends the Hydra line from open-loop score-based planning toward closed-loop robustness by adding short-horizon control prediction and kinematic trajectory refinement.**

# Core takeaways
- Targets the open-loop / closed-loop gap directly.
- Combines trajectory prediction, control prediction, and trajectory refinement in a unified planner.
- Reports large gains on Bench2Drive while remaining strong on NAVSIM.

# Method summary
Hydra-NeXt starts from a Hydra-MDP-style multimodal planner and adds a control decoder for short-horizon actions plus a refinement module that adjusts plans for better kinematic feasibility and reactivity in closed-loop execution.

# Benchmarks / results
The paper reports 65.89 Driving Score and 48.20% Success Rate on Bench2Drive, plus 88.6 PDMS on NAVSIM, positioning it as a stronger closed-loop-oriented successor to Hydra-MDP.

# Relation to the score-based E2E AD lineage
This is a direct descendant of Hydra-MDP: same candidate-based planning DNA, but focused on surviving closed-loop deployment rather than only maximizing open-loop planning scores.

# Caveats
- Still primarily trained from open-loop data.
- Comfort and traffic-sign behavior remain weaker than some specialized baselines.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
