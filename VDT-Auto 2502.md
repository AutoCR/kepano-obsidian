---
categories:
  - "[[Papers]]"
title: "VDT-Auto: End-to-end Autonomous Driving with VLM-Guided Diffusion Transformers"
authors:
  - Ziang Guo
  - Konstantin Gubernatorov
  - Selamawit Asfaw
  - Zakhar Yagudin
  - Dzmitry Tsetserukou
affiliation:
  - "Intelligent Space Robotics Laboratory, Center for Digital Engineering, Skolkovo Institute of Science and Technology, Moscow, Russia"
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2502.20108
pdf: https://arxiv.org/pdf/2502.20108v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - VLM guidance
  - diffusion transformer
  - BEV encoder
  - nuScenes
  - language-conditioned planning
status:
  - read
rating:
dataset:
  - nuScenes
method:
  - BEV encoder
  - fine-tuned VLM guidance
  - diffusion transformer planner
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[GoalFlow 2503]]"
---

One-line takeaway: **VDT-Auto injects explicit language-model scene understanding into a diffusion planner, making it more of a VLM-conditioned generative planner than a classic score-based scorer.**

# Core takeaways
- Injects a VLM as a conditioning signal for trajectory generation.
- Separates geometric BEV conditioning from contextual language-style conditioning.
- Represents a neighboring VLM + diffusion route rather than a central Hydra-style scorer.

# Method summary
VDT-Auto builds BEV features from multi-camera input and uses a fine-tuned vision-language model to produce contextual embeddings and path priors. These condition a diffusion transformer that outputs the final trajectory.

# Benchmarks / results
The paper reports 0.52 m average L2 error and about 21% collision rate on nuScenes open-loop planning, plus a qualitative real-world demonstration.

# Relation to the score-based E2E AD lineage
VDT-Auto is adjacent to the score-based lineage: it is relevant as a conditioning alternative, but not a direct descendant of Hydra-MDP-style trajectory scoring.

# Caveats
- Benchmarking is on nuScenes rather than NAVSIM/Bench2Drive.
- Real-world evidence is mostly qualitative.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
