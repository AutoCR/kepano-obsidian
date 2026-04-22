---
categories:
  - "[[Papers]]"
title: "DriveCritic: Towards Context-Aware, Human-Aligned Evaluation for Autonomous Driving with Vision-Language Models"
authors:
  - Jingyu Song
  - Zhenxin Li
  - Shiyi Lan
  - Xinglong Sun
  - Nadine Chang
  - Maying Shen
  - Joshua Chen
  - Katherine A. Skinner
  - Jose M. Alvarez
affiliation:
  - NVIDIA
  - University of Michigan
  - Fudan University
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2510.13108
pdf: https://arxiv.org/pdf/2510.13108v2.pdf
field:
  - autonomous-driving
  - evaluation
keywords:
  - vlm-judge
  - pairwise-preference
  - human-aligned-evaluation
  - navsim
  - rlvr
status:
  - read
rating:
dataset:
  - DriveCritic dataset
  - Navsim
method:
  - Qwen2.5-VL evaluator
  - SFT
  - RLVR
  - DAPO
task: human-aligned pairwise evaluation of driving trajectories
created: 2026-04-22
updated: 2026-04-22
tags:
  - paper
  - nvidia
related:
  - "[[OmniDrive 2405]]"
  - "[[Generalized Trajectory Scoring 2506]]"
  - "[[Alpamayo-R1 2511]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
---

One-line takeaway: **DriveCritic replaces brittle hand-crafted open-loop metrics with a VLM judge trained on pairwise driving preferences in ambiguous scenes.**

# Core takeaways
- The paper is about **evaluation**, not trajectory generation.
- It targets cases where existing metrics do not match human judgment.
- The dataset is deliberately built around ambiguous lane-keeping / progress trade-offs.
- The evaluator combines visual context, BEV context, ego status, and symbolic metric hints.
- This is NVIDIA’s strongest sign that “better planners” also require “better judges.”

# Model details
## Input representation
- A stitched multi-camera front-view image.
- A BEV map with overlaid trajectory candidates.
- Ego-state signals such as speed and acceleration.
- Selected symbolic sub-scores like ego progress and lane-keeping context.

## Model
- Backbone: **Qwen2.5-VL-7B**.
- Output: pairwise preference judgment between trajectory A and B, plus structured reasoning.
- This is an evaluator / judge, not a planner.

## Training pipeline
- SFT warm start on a subset with teacher-generated reasoning traces.
- RLVR-style reinforcement learning with DAPO.
- Rewards include format correctness and matching the preference label.

# Training / inference notes
- The DriveCritic dataset is built from Navsim-derived trajectory pairs.
- The train / test split focuses on difficult cases rather than average easy cases.
- GPT-5 is used to help produce chain-of-thought style supervision and pseudo-label some data.

# Benchmarks / results
- DriveCritic substantially outperforms EPDMS and zero-shot frontier VLM baselines on the paper’s test set.
- The final model is also more robust under A/B flip consistency checks.
- The paper reports clear gains from SFT, then further gains from RLVR / DAPO.

# Caveats
- Preference data are still relatively small and rely heavily on expert curation.
- The benchmark is a pilot benchmark rather than a complete replacement for all AV evaluation.
- A VLM judge brings its own risks: prompt sensitivity, hallucination, and extra compute.

# My understanding
This paper is a conceptual sequel to [[Ego Status All You Need 2312]]. The earlier paper says “the metric is not enough.” DriveCritic says “then let us build a better judge.”

# Related reading
- [[Ego Status All You Need 2312]]
- [[OmniDrive 2405]]
- [[Alpamayo-R1 2511]]
- [[NVIDIA end-to-end AV review 2023-now]]
