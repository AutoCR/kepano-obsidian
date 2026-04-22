---
categories:
  - "[[Papers]]"
title: "OmniDrive: A Holistic Vision-Language Dataset for Autonomous Driving with Counterfactual Reasoning"
authors:
  - Shihao Wang
  - Zhiding Yu
  - Xiaohui Jiang
  - Shiyi Lan
  - Min Shi
  - Nadine Chang
  - Jan Kautz
  - Ying Li
  - Jose M. Alvarez
affiliation:
  - NVIDIA
  - The Hong Kong Polytechnic University
  - Beijing Institute of Technology
venue: arXiv
year: 2024
doi:
url: https://arxiv.org/abs/2405.01533
pdf: https://arxiv.org/pdf/2405.01533v2.pdf
field:
  - autonomous-driving
  - vision-language-models
keywords:
  - counterfactual-reasoning
  - driving-qa
  - llm-agent
  - open-loop-planning
  - drivelm
status:
  - read
rating:
dataset:
  - OmniDrive
  - nuScenes
  - DriveLM
method:
  - counterfactual data generation
  - Omni-L
  - Omni-Q
task: language-grounded reasoning and planning for autonomous driving
created: 2026-04-22
updated: 2026-04-22
tags:
  - paper
  - nvidia
related:
  - "[[Ego Status All You Need 2312]]"
  - "[[Hydra-MDP 2406]]"
  - "[[DriveCritic 2510]]"
  - "[[Alpamayo-R1 2511]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
---

One-line takeaway: **OmniDrive turns driving into a counterfactual vision-language problem, showing that stronger language alignment plus 3D grounding can improve both reasoning and planning.**

# Core takeaways
- The paper builds a **counterfactual driving QA / planning dataset** instead of only predicting expert trajectories.
- It compares two design philosophies: start from a language-native VLM (**Omni-L**) versus start from a 3D-perception-heavy model (**Omni-Q**).
- Omni-L generally works better, suggesting that strong language alignment is an important starting point.
- Counterfactual language supervision improves both reasoning quality and open-loop planning.
- This paper is a bridge from pure trajectory prediction to reasoning-augmented driving models.

# Model details
## Data generation
- Start from nuScenes scenes.
- Generate simulated alternative trajectories by clustering representative future motions.
- Use rule-based checks to measure collisions, road-boundary violations, red-light failures, and other consequences.
- Combine scene summaries, object context, and simulated consequences into GPT-4 prompts.
- Produce several QA types: scene understanding, 3D grounding, counterfactual reasoning, and planning dialogue.

## Omni-Q
- Built in the spirit of StreamPETR + Q-Former.
- Uses carrier queries and detection queries.
- Queries attend to multi-view image features with 3D positional encoding.
- Detection queries support explicit 3D perception supervision.
- Carrier queries are projected into the language model token space.

## Omni-L
- More LLaVA-like.
- Multi-view image features are flattened and sent through an MLP projector into the LLM.
- 3D positional encoding is added on top of image patches.
- This simpler language-first route performs better overall in the paper.

## Training objective
- Stage 1: 2D pretraining.
- Stage 2: 3D driving finetuning.
- Main training loss is text-generation supervision rather than a complicated mixed contrastive objective.

# Training / inference notes
- The paper uses EVA-02-L as the shared visual backbone.
- Finetuning uses AdamW with separate learning rates for the projector and backbone / LLM.
- Planning is framed as language-grounded prediction rather than a classical separate trajectory decoder.

# Benchmarks / results
- OmniDrive pretraining improves counterfactual reasoning on DriveLM-style evaluation.
- It also improves nuScenes open-loop planning, especially when ego-status shortcuts are reduced.
- Omni-L beats Omni-Q on several reasoning-oriented metrics, while Omni-Q remains useful for testing how much explicit 3D perception helps.

# Caveats
- The paper still lives partly inside the nuScenes open-loop world criticized by [[Ego Status All You Need 2312]].
- QA quality depends on prompt design, rule-check quality, and generated labels.
- The work is more about data and alignment than about a fully deployable planner.

# My understanding
OmniDrive is where NVIDIA’s line visibly widens from “predict trajectories” to “reason about alternatives.” It is an important precursor to both [[Alpamayo-R1 2511]] and [[DriveCritic 2510]].

# Related reading
- [[Ego Status All You Need 2312]]
- [[Hydra-MDP 2406]]
- [[DriveCritic 2510]]
- [[Alpamayo-R1 2511]]
