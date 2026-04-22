---
categories:
  - "[[Papers]]"
title: "GuideFlow: Constraint-Guided Flow Matching for Planning in End-to-End Autonomous Driving"
authors:
  - Lin Liu
  - Caiyan Jia
  - Guanyi Yu
  - Ziying Song
  - JunQiao Li
  - Feiyang Jia
  - Peiliang Wu
  - Xiaoshuai Hao
  - Yadan Luo
affiliation:
  - "School of Computer Science and Technology, Beijing Jiaotong University"
  - "Beijing Key Laboratory of Traffic Data Mining and Embodied Intelligence"
  - "Qcraft"
  - "Yanshan University"
  - "Institute of Information Engineering, Chinese Academy of Sciences"
  - "The University of Queensland"
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2511.18729
pdf: https://arxiv.org/pdf/2511.18729v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - flow matching
  - constraint guidance
  - energy-based model
  - driving style control
  - NavSim
  - Bench2Drive
status:
  - read
rating:
dataset:
  - NavSim
  - Bench2Drive
  - nuScenes
  - ADV-NuScenes
method:
  - constraint-guided flow matching
  - joint EBM training
  - aggressiveness control
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[GoalFlow 2503]]"
  - "[[HAD 2604]]"
---

One-line takeaway: **GuideFlow turns flow matching into a constraint-aware planner, directly enforcing safety and physical rules during generation instead of bolting them on afterward.**

# Core takeaways
- Targets both mode collapse and weak constraint handling.
- Injects constraints directly into the generative process rather than only in post-processing.
- Adds a controllable aggressiveness signal to modulate driving style.

# Method summary
GuideFlow uses conditional flow matching to map noise to feasible trajectories, while adding explicit physical and safety constraints to the vector field. A jointly trained energy-based model further refines generation and helps the planner satisfy constraints more reliably.

# Benchmarks / results
The paper reports 43.0 EPDMS on Navhard with a scorer, plus 75.04 driving score and 50.90% success rate on Bench2Drive.

# Relation to the score-based E2E AD lineage
GuideFlow is part of the broader generate-and-score family. It is relevant to the score-based lineage because it still relies on multimodal candidate quality and often a scorer, but pushes more strongly toward guided generative planning.

# Caveats
- Results should distinguish clearly between no-scorer and with-scorer settings.
- Method complexity is nontrivial because several modules interact.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
