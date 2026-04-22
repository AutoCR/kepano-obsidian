---
categories:
  - "[[Papers]]"
title: "Hydra-MDP: End-to-end Multimodal Planning with Multi-target Hydra-Distillation"
authors:
  - Zhenxin Li
  - Kailin Li
  - Shihao Wang
  - Shiyi Lan
  - Zhiding Yu
  - Yishen Ji
  - Zhiqi Li
  - Ziyue Zhu
  - Jan Kautz
  - Zuxuan Wu
  - Yu-Gang Jiang
  - Jose M. Alvarez
affiliation:
  - NVIDIA
  - Fudan University
  - East China Normal University
  - Beijing Institute of Technology
  - Nanjing University
  - Nankai University
venue: arXiv
year: 2024
doi:
url: https://arxiv.org/abs/2406.06978
pdf: https://arxiv.org/pdf/2406.06978v4.pdf
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - multimodal-planning
  - trajectory-vocabulary
  - distillation
  - navsim
  - score-based-planning
status:
  - read
rating:
dataset:
  - Navsim
  - OpenScene
  - nuPlan
method:
  - hydra-distillation
  - trajectory vocabulary scoring
  - teacher-student learning
task: end-to-end multimodal planning
created: 2026-03-13
updated: 2026-04-22
tags:
  - paper
  - nvidia
related:
  - "[[Ego Status All You Need 2312]]"
  - "[[Generalized Trajectory Scoring 2506]]"
  - "[[OmniDrive 2405]]"
  - "[[transfuser]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
---

One-line takeaway: **Hydra-MDP turns closed-loop-like planning signals into differentiable student-predicted sub-scores, making multimodal end-to-end planning more aligned with rule-based driving quality.**

# Core takeaways
- Hydra-MDP is a major NVIDIA step from benchmark critique to a stronger planner.
- The planner is **multimodal** because multiple futures can be valid.
- The planner is also **multi-target** because a single scalar score hides which part of driving quality matters.
- Instead of only imitating the human trajectory, it distills metric-specific signals from rule-based teachers.
- This is one of the key foundations for NVIDIA’s later score-based planning line.

# Model details
## Perception stack
- Perception is based on [[transfuser]].
- Image and LiDAR features are fused through transformer layers.
- The fused output is represented as environment tokens used by the planner.

## Trajectory vocabulary
- About 700k trajectories are sampled from nuPlan-like data.
- Each trajectory spans 4 seconds with 40 timestamps of `(x, y, heading)`.
- K-means cluster centers define a fixed trajectory vocabulary.
- Vocabulary trajectories are embedded as latent planning queries.

## Scoring architecture
- The trajectory queries attend to the environment tokens through transformer decoder layers.
- The student predicts:
  - an imitation score,
  - no-collision score,
  - drivable-area score,
  - time-to-collision score,
  - comfort score,
  - optionally ego-progress score.
- The final trajectory is chosen by combining these predicted sub-scores into a single cost.

## Teacher design
- **Human teacher:** the log-replay / expert trajectory.
- **Rule-based teachers:** offline simulation over all candidate trajectories using privileged environment information.
- Key idea: distill each metric separately rather than distilling one overall scalar.

## Losses
- Distance-based soft imitation loss over the trajectory vocabulary.
- Binary-cross-entropy-style distillation losses for each metric-specific hydra head.
- Final training objective combines imitation and multi-target knowledge distillation.

# Training / inference notes
- Training reported on 8×A100, batch size 256, 20 epochs, learning rate 1e-4.
- Images are arranged as a 256×1024 multi-view input.
- LiDAR uses 4-frame splatting into BEV.
- No heavy data augmentation is emphasized.

# Benchmarks / results
- Hydra-MDP won the Navsim challenge.
- The paper reports overall score improvements over PDM-Closed while staying end-to-end.
- It shows that metric-wise distillation works better than directly distilling a single PDM-like scalar.

# Caveats
- The planner still relies on a **fixed vocabulary**, so multimodality is discretized.
- Score weighting is tuned manually / by grid search.
- Quality depends strongly on the fidelity of the rule-based teachers.

# My understanding
Hydra-MDP is the cleanest expression of NVIDIA’s mid-period idea: planning should be a **trajectory scoring** problem, but the score must be decomposed into interpretable targets. Later work like [[Generalized Trajectory Scoring 2506]] extends this idea beyond fixed vocabulary-only scoring.

# Related reading
- [[Generalized Trajectory Scoring 2506]]
- [[OmniDrive 2405]]
- [[transfuser]]
- [[NVIDIA end-to-end AV review 2023-now]]
