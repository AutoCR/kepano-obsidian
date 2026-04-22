---
categories:
  - "[[Papers]]"
title: "Generalized Trajectory Scoring for End-to-end Multimodal Planning"
authors:
  - Zhenxin Li
  - Wenhao Yao
  - Zi Wang
  - Xinglong Sun
  - Joshua Chen
  - Nadine Chang
  - Maying Shen
  - Zuxuan Wu
  - Shiyi Lan
  - Jose M. Alvarez
affiliation:
  - NVIDIA
  - Fudan University
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2506.06664
pdf: https://arxiv.org/pdf/2506.06664v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - trajectory-scoring
  - diffusion-proposals
  - navsim
  - multimodal-planning
  - robustness
status:
  - read
rating:
dataset:
  - Navsim
  - Navhard
method:
  - GTRS
  - diffusion proposals
  - vocabulary dropout
  - sensor augmentation
task: end-to-end multimodal planning
created: 2026-04-20
updated: 2026-04-22
tags:
  - paper
  - nvidia
related:
  - "[[Hydra-MDP 2406]]"
  - "[[Latent World Models for Covariate Shift 2409]]"
  - "[[DriveCritic 2510]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
---

One-line takeaway: **GTRS wins Navsim v2 by training a scorer that can judge both dense static trajectory vocabularies and dynamic diffusion proposals under harder sensor conditions.**

# Core takeaways
- This paper extends the score-based planning line beyond fixed-vocabulary-only scoring.
- It separates planning into **candidate generation** and **candidate scoring**.
- The scorer is trained to generalize across different candidate subsets rather than overfit to a single proposal pool.
- Sensor augmentation and refinement improve robustness on the harder Navhard regime.
- GTRS is best read as the next step after [[Hydra-MDP 2406]].

# Model details
## Overall structure
- **DP:** diffusion-based trajectory generator.
- **GTRS-Dense:** dense-vocabulary scorer.
- **GTRS-Aug:** robustness-oriented scorer with sensor augmentation and refinement.

## Diffusion proposal generator
- Uses image features and BEV features.
- A diffusion transformer generates dynamic trajectory proposals.
- BEV segmentation supervision helps stabilize the feature representation.
- DDPM-style denoising is used for sampling.

## Dense scorer
- A trajectory tokenizer encodes candidate trajectories.
- A transformer decoder models interactions between image tokens and trajectory tokens.
- Training uses a **super-dense static vocabulary** and **trajectory dropout**.
- The crucial trick is that the scorer learns to handle variable candidate sets and can therefore score unseen diffusion proposals at inference.

## Augmented scorer
- Applies sensor perturbations / view rotations during training.
- Adds a training-only refinement decoder for top-k candidates.
- Uses EMA self-distillation to refine high-value scores.

# Training / inference notes
- Trained on Navtrain; synthetic / Navhard test data are not directly used for training.
- Reported training uses 24×A100, large batch size, and 20 epochs for scorers.
- The diffusion proposal generator is trained longer than the scorer.
- Inference appends dynamic proposals to the static vocabulary and chooses the highest-scoring candidate.

# Benchmarks / results
- The paper reports a step-by-step gain from dense scoring, then candidate-set generalization, then augmentation/refinement.
- Best single-model and ensemble results substantially improve over earlier Navsim baselines.
- The top ensemble approaches the privileged PDM-Closed reference under Navhard.

# Caveats
- Benchmark dependence is strong: the paper is tightly optimized for Navsim / EPDMS-style evaluation.
- Proposal quality still matters a lot; the scorer cannot rescue completely bad candidate sets.
- Dynamic proposals are added mainly at inference, so full joint generator-scorer optimization is limited.

# My understanding
Hydra-MDP says “predict decomposed scores over a vocabulary.” GTRS says “that is not enough — the scorer must generalize across both static and dynamic proposal spaces.” It is a more flexible and more challenge-robust version of the same overall planning worldview.

# Related reading
- [[Hydra-MDP 2406]]
- [[DriveCritic 2510]]
- [[NVIDIA end-to-end AV review 2023-now]]
