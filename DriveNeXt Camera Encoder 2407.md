---
categories:
  - "[[Papers]]"
title: "Exploring Camera Encoder Designs for Autonomous Driving Perception"
authors:
  - Barath Lakshmanan
  - Joshua Chen
  - Shiyi Lan
  - Maying Shen
  - Zhiding Yu
  - Jose M. Alvarez
affiliation:
  - NVIDIA
venue: arXiv
year: 2024
doi:
url: https://arxiv.org/abs/2407.07276
pdf: https://arxiv.org/pdf/2407.07276v1.pdf
field:
  - autonomous-driving
  - perception
keywords:
  - camera-encoder
  - drive-next
  - 3d-detection
  - convnext
  - hybrid-attention
status:
  - read
rating:
dataset:
  - internal NVIDIA AV perception dataset
method:
  - DriveNeXt
  - DriveNeXt-Hybrid
task: camera feature encoding for autonomous driving perception
created: 2026-04-22
updated: 2026-04-22
tags:
  - paper
  - nvidia
related:
  - "[[SSCBench 2306]]"
  - "[[Latent World Models for Covariate Shift 2409]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
---

One-line takeaway: **This paper shows that autonomous-driving camera encoders should be designed for AV-specific detection geometry rather than inherited unchanged from generic image-classification backbones.**

# Core takeaways
- The work is a systematic encoder-design paper, not a planner paper.
- Early-stage compute matters more than just making deeper layers larger.
- High-resolution inputs help long-range detection a lot.
- A small amount of attention helps, but a mostly convolutional encoder remains efficient and strong.
- This paper reflects NVIDIA’s push to improve the perception substrate behind later planning systems.

# Model details
## DriveNeXt
- Starts from a ConvNeXt-like hierarchical encoder.
- Replaces some generic design choices with AV-oriented and hardware-friendly ones.
- Uses large-kernel convolutions and channel bottlenecks.
- Explores stage depth, width, compute ratio, and resolution trade-offs.

## DriveNeXt-Hybrid
- Adds multi-head self-attention in stage 3.
- Attention is used selectively rather than turning the encoder into a pure ViT.
- Best results come from inserting a small attention block stack near the end of the stage.

## Key architectural conclusions
- Four stages work well for this setting.
- The best stage-compute ratio is roughly 1:7:4:1.
- Capacity added to earlier stages helps more than capacity added late.
- Native / less-downsampled resolution strongly benefits far-range perception.

# Training / inference notes
- Trained on an internal large-scale industrial AV dataset.
- The paper reports tiny / small / base / large variants for different deployment budgets.
- Evaluation is framed through downstream 3D detection mAP rather than image-classification transfer.

# Benchmarks / results
- The optimized design improves the baseline from about 72.8 mAP to about 79.2 mAP.
- Hybrid attention adds a further modest gain over the pure-conv version.
- High-resolution input boosts performance but costs much more compute.

# Caveats
- Results are on a proprietary internal dataset, so direct public-benchmark comparison is limited.
- The contribution is narrowly about the camera encoder, not end-to-end planning itself.

# My understanding
This paper belongs in the “adjacent stack” bucket. It matters because NVIDIA’s later end-to-end and reasoning-heavy work still depends on strong camera representations. The paper is best read as infrastructure for the planner line, not as the planner line itself.

# Related reading
- [[SSCBench 2306]]
- [[Latent World Models for Covariate Shift 2409]]
- [[NVIDIA end-to-end AV review 2023-now]]
