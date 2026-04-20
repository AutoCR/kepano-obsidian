---
categories:
  - "[[Papers]]"
title: "TransDiffuser: Diverse Trajectory Generation with Decorrelated Multi-modal Representation for End-to-end Autonomous Driving"
authors:
  - Xuefeng Jiang
  - Yuan Ma
  - Pengxiang Li
  - Leimeng Xu
  - Xin Wen
  - Kun Zhan
  - Zhongpu Xia
  - Peng Jia
  - Xianpeng Lang
  - Sheng Sun
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2505.09315
pdf: https://arxiv.org/pdf/2505.09315v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - diffusion planning
  - mode collapse
  - decorrelation
  - anchor-free planning
  - NAVSIM
  - multimodal diversity
status:
  - read
rating:
dataset:
  - NAVSIM
method:
  - scene-conditioned diffusion decoder
  - latent decorrelation regularization
  - anchor-free multimodal generation
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[DiffusionDrive 2411]]"
  - "[[GoalFlow 2503]]"
---

One-line takeaway: **TransDiffuser attacks mode collapse in diffusion-based E2E planning by decorrelating multimodal representations, achieving very strong NAVSIM performance without anchor priors.**

# Core takeaways
- Attacks mode collapse in diffusion-based E2E planning.
- Avoids predefined anchors or vocabulary priors.
- Shows that stronger diversity regularization can improve both candidate spread and final planning score.

# Method summary
TransDiffuser uses a scene-conditioned diffusion decoder and adds a simple but effective decorrelation mechanism in latent space so different modes remain distinct during denoising. This improves diversity without relying on predefined anchors.

# Benchmarks / results
The paper reports 94.85 PDMS on NAVSIM and claims state-of-the-art performance without anchor-based priors, while also showing improved diversity metrics in ablations.

# Relation to the score-based E2E AD lineage
TransDiffuser is more of an alternative branch than a Hydra descendant, but it is an important foil to pure scoring papers because it argues strong anchor-free diffusion can outperform static candidate scoring.

# Caveats
- Less interpretable than explicit candidate-scoring pipelines.
- Closed-loop transfer evidence is limited in the paper’s public-facing results.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
