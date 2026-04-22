---
categories:
  - "[[Papers]]"
title: "Mitigating Covariate Shift in Imitation Learning for Autonomous Vehicles Using Latent Space Generative World Models"
authors:
  - Alexander Popov
  - Alperen Degirmenci
  - David Wehr
  - Shashank Hegde
  - Ryan Oldja
  - Alexey Kamenev
  - Bertrand Douillard
  - David Nistér
  - Urs Muller
  - Ruchi Bhargava
  - Stan Birchfield
  - Nikolai Smolyanskiy
affiliation:
  - NVIDIA
venue: arXiv
year: 2024
doi:
url: https://arxiv.org/abs/2409.16663
pdf: https://arxiv.org/pdf/2409.16663v4.pdf
field:
  - autonomous-driving
  - imitation-learning
keywords:
  - covariate-shift
  - world-model
  - latent-dynamics
  - carla
  - drive-sim
status:
  - read
rating:
dataset:
  - CARLA expert dataset
  - NVIDIA internal real driving dataset
method:
  - latent-space generative world model
  - end-to-end imitation learning
task: end-to-end autonomous driving under covariate shift
created: 2026-04-22
updated: 2026-04-22
tags:
  - paper
  - nvidia
related:
  - "[[DriveNeXt Camera Encoder 2407]]"
  - "[[Generalized Trajectory Scoring 2506]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
---

One-line takeaway: **This paper uses a latent generative world model during training so an imitation-learned driving policy can learn recovery behavior instead of collapsing under covariate shift.**

# Core takeaways
- The world model is used mainly at **training time**, not at deployment time.
- The policy learns from imagined latent rollouts, so it can recover from slightly off-demonstration states.
- The perception encoder is stronger than a basic imitation-learning stack: it uses DINOv2 features, multi-view / temporal attention, and a learned scene query.
- The method is validated in CARLA and qualitatively in DRIVE Sim.
- It is a different branch from vocabulary-scoring planners like [[Hydra-MDP 2406]], but it addresses the same central issue: robustness beyond clean demonstrations.

# Model details
## Perception encoder
- Inputs: sequential RGB frames, past ego motion, navigation path, and ego speed.
- Each image is encoded with **DINOv2**.
- Cross-attention between current and previous frames injects motion information.
- Camera geometry embeddings are added before attention.
- A learned **scene query** pools the scene into a compact latent observation vector.

## Latent world-model stack
- History network: GRU-like recurrent state update.
- Posterior / state estimator: predicts the latent posterior from current observation + history + expert action.
- Prior / world model: predicts future latent state given history and action.
- Policy: a 4-layer MLP that maps latent state to steering and acceleration / brake.
- Decoders reconstruct RGB and BEV semantics during training.

## Losses
- ELBO-style training objective.
- Reconstruction terms for RGB, BEV semantics, and actions.
- KL term that aligns the posterior latent state with the action-conditioned prior.
- The training signal encourages the policy to align with expert-like states even after perturbation.

# Training / inference notes
- CARLA setup: about 50k expert episodes, 12 frames each at 10 Hz.
- Real-data setup: roughly 450 hours of internal data and 1.3M episodes.
- Training compute is heavy: multi-day runs on large A100 clusters.
- Deployment runs near real time at 10 Hz on Orin AGX.
- The world model is not required at runtime.

# Benchmarks / results
- The method outperforms a matched behavior-cloning baseline under perturbation.
- It shows stronger route completion and crash-free behavior in CARLA robustness tests.
- The paper also demonstrates recovery behavior in DRIVE Sim under control lag.

# Caveats
- A lot of the most impressive evidence is in simulation or internal infrastructure.
- The world model is short-horizon and mostly a training-time helper, not a full planning world model in the broad sense.
- Public-benchmark comparability is weaker than for Navsim / nuScenes papers.

# My understanding
This paper is a “robustness via latent dynamics” branch inside NVIDIA’s E2E work. Compared with the score-based planning branch, it focuses less on candidate scoring and more on making imitation learning recover from compounding errors.

# Related reading
- [[Hydra-MDP 2406]]
- [[Generalized Trajectory Scoring 2506]]
- [[NVIDIA end-to-end AV review 2023-now]]
