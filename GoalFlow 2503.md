---
categories:
  - "[[Papers]]"
title: "GoalFlow: Goal-Driven Flow Matching for Multimodal Trajectories Generation in End-to-End Autonomous Driving"
authors:
  - Zebin Xing
  - Xingyu Zhang
  - Yang Hu
  - Bo Jiang
  - Tong He
  - Qian Zhang
  - Xiaoxiao Long
  - Wei Yin
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2503.05689
pdf: https://arxiv.org/pdf/2503.05689v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - flow matching
  - goal points
  - multimodal trajectories
  - trajectory scoring
  - NAVSIM
  - one-step inference
status:
  - read
rating:
dataset:
  - NAVSIM
method:
  - goal point construction
  - flow matching generator
  - trajectory scoring
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[GuideFlow 2511]]"
  - "[[CdDrive 2602]]"
---

One-line takeaway: **GoalFlow replaces diffusion with goal-driven flow matching, using explicit goal points to reduce mode ambiguity and make multimodal generation practical for E2E planning.**

# Core takeaways
- Uses explicit goal points to constrain multimodal generation.
- Replaces diffusion with flow matching for more efficient generation.
- Keeps a scoring stage to select among generated candidates.

# Method summary
GoalFlow first constructs goal points from scene information, then uses flow matching to generate multimodal trajectories that satisfy both the scene context and those goals. A refined scoring mechanism ranks the generated candidates and picks the final plan.

# Benchmarks / results
On NavSim test, GoalFlow reports 90.3 PDMS, and the paper emphasizes that it remains competitive even with very few denoising/integration steps.

# Relation to the score-based E2E AD lineage
GoalFlow belongs to the broader generate-and-score family: less pure scoring than Hydra/SparseDriveV2, but still relevant because final decision making is done by learned scoring over multimodal proposals.

# Caveats
- Quality depends on the quality of constructed goal points.
- Comparison with static-vocabulary approaches mixes guidance and generation differences.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
