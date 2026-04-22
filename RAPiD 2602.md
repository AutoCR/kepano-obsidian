---
categories:
  - "[[Papers]]"
title: "RAPiD: Real-time Deterministic Trajectory Planning via Diffusion Behavior Priors for Safe and Efficient Autonomous Driving"
authors:
  - Ruturaj Reddy
  - Hrishav Bakul Barua
  - Junn Yong Loo
  - Thanh Thi Nguyen
  - Ganesh Krishnasamy
affiliation:
  - "School of Information Technology, Monash University, Malaysia"
  - "Faculty of Information Technology, Monash University, Australia"
  - "TCS Research, India"
venue: arXiv
year: 2026
doi:
url: https://arxiv.org/abs/2602.07339
pdf: https://arxiv.org/pdf/2602.07339v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - policy distillation
  - diffusion prior
  - deterministic planner
  - nuPlan
  - interPlan
  - real-time inference
status:
  - read
rating:
dataset:
  - nuPlan
  - interPlan
method:
  - diffusion-to-policy distillation
  - score-regularized policy optimization
  - safety critic
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[DiffusionDriveV2 2512]]"
  - "[[HDP 2602]]"
---

One-line takeaway: **RAPiD distills a diffusion planner into a deterministic real-time policy, trading some multimodal flexibility for much faster and safer deployment behavior.**

# Core takeaways
- Targets deployment latency rather than only generation quality.
- Uses a pretrained diffusion planner as a behavior prior for a deterministic policy.
- Shows stronger generalization and much faster inference on closed-loop benchmarks.

# Method summary
RAPiD extracts a deterministic policy from a pretrained diffusion planner using score-regularized policy optimization. A critic trained from planning-quality signals pushes the distilled policy toward safer, smoother behavior while avoiding iterative denoising at runtime.

# Benchmarks / results
On nuPlan and interPlan, RAPiD reports strong closed-loop performance together with about an 8× inference speedup over diffusion baselines, with especially good generalization on interPlan long-tail scenarios.

# Relation to the score-based E2E AD lineage
RAPiD branches off from the score-based lineage by treating diffusion planning as a teacher to be compressed, emphasizing deployable real-time control more than richer multimodal generation.

# Caveats
- Deterministic distillation may lose some multimodal flexibility.
- Not a direct improvement of score-based sampling itself.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
