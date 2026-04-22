---
categories:
  - "[[Papers]]"
title: "Fast-ThinkAct: Efficient Vision-Language-Action Reasoning via Verbalizable Latent Planning"
authors:
  - Chi-Pin Huang
  - Yunze Man
  - Zhiding Yu
  - Min-Hung Chen
  - Jan Kautz
  - Yu-Chiang Frank Wang
  - Fu-En Yang
affiliation:
  - NVIDIA
  - National Taiwan University
  - University of Illinois Urbana-Champaign
venue: arXiv
year: 2026
doi:
url: https://arxiv.org/abs/2601.09708
pdf: https://arxiv.org/pdf/2601.09708v2.pdf
field:
  - embodied-ai
  - vision-language-action
keywords:
  - latent-reasoning
  - verbalizer
  - teacher-student-distillation
  - low-latency-vla
status:
  - read
rating:
dataset:
  - OXE
  - AIST
  - RoboFAC
  - RoboVQA
  - EgoPlan-Bench
method:
  - latent CoT
  - verbalizable latent planning
  - diffusion action model
task: low-latency reasoning for VLA systems
created: 2026-04-22
updated: 2026-04-22
tags:
  - paper
  - nvidia
related:
  - "[[Alpamayo-R1 2511]]"
  - "[[OmniDrive 2405]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
---

One-line takeaway: **Fast-ThinkAct compresses explicit reasoning into compact latent plans, showing how VLA systems can keep planning ability without paying the full latency cost of long verbal chain-of-thought.**

# Core takeaways
- This is not strictly an AV paper, but it is highly relevant to NVIDIA’s broader Physical AI direction.
- The key idea is to keep the benefits of reasoning while moving most of the reasoning into a compact latent space.
- A verbalizer makes the latent plan partly interpretable during training.
- The method then conditions a diffusion action model on the latent visual plan.
- For the NVIDIA driving line, the main relevance is architectural: it hints at how reasoning-heavy control systems can remain real-time.

# Model details
## Teacher-student structure
- A textual teacher VLM produces explicit CoT traces.
- A latent student VLM generates a small number of continuous latent reasoning tokens.
- A verbalizer LLM is trained to decode those latent tokens back into language.
- Spatial tokens represent the compact visual plan.

## Action model
- A diffusion transformer policy consumes the latent plan / KV-cache and outputs executable actions.
- This splits reasoning from low-level action generation, similar in spirit to AR1’s fast action expert.

## Losses
- Preference-style verbalizer loss.
- Hidden-state distillation loss from teacher to student.
- Waypoint / plan supervision loss.
- Separate imitation / denoising loss for the action policy.

# Training / inference notes
- Training proceeds in multiple stages: CoT supervision, teacher optimization, latent-student distillation, and action-policy learning.
- Inference uses only the student plus the action model, not the full training-time verbalizer pipeline.
- The paper reports large latency reductions versus explicit reasoning VLAs.

# Benchmarks / results
- The paper reports strong gains on embodied reasoning and robot-control benchmarks.
- The most important systems result is the very large inference-latency reduction versus explicit-reasoning VLA baselines.

# Caveats
- This is not a driving benchmark paper, so direct transfer to autonomous driving remains suggestive rather than proven.
- Compressing reasoning improves speed but makes internal reasoning less directly inspectable unless verbalized.

# My understanding
Fast-ThinkAct is best viewed as a forward-looking systems paper for NVIDIA’s reasoning-and-action agenda. It helps explain how papers like [[Alpamayo-R1 2511]] might evolve if real-time deployment pressure becomes the main bottleneck.

# Related reading
- [[Alpamayo-R1 2511]]
- [[OmniDrive 2405]]
- [[NVIDIA end-to-end AV review 2023-now]]
