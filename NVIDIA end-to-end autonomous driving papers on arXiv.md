---
categories:
  - "[[Evergreen]]"
title: "NVIDIA end-to-end autonomous driving papers on arXiv"
created: 2026-04-22
updated: 2026-04-22
tags:
  - 0🌲
  - literature-review
  - autonomous-driving
  - nvidia
  - arxiv
  - end-to-end
  - vla
sources:
  - https://arxiv.org/abs/1604.07316
  - https://arxiv.org/abs/1704.07911
  - https://arxiv.org/abs/2010.08776
  - https://arxiv.org/abs/2306.09001
  - https://arxiv.org/abs/2312.03031
  - https://arxiv.org/abs/2405.01533
  - https://arxiv.org/abs/2406.06978
  - https://arxiv.org/abs/2407.07276
  - https://arxiv.org/abs/2409.16663
  - https://arxiv.org/abs/2506.06664
  - https://arxiv.org/abs/2510.13108
  - https://arxiv.org/abs/2511.00088
  - https://arxiv.org/abs/2601.09708
period:
  start: 2016
  end: 2026
related:
  - "[[Hydra-MDP 2406]]"
  - "[[Generalized Trajectory Scoring 2506]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
---

# NVIDIA end-to-end autonomous driving papers on arXiv

## Executive summary
- This note is a curated map of NVIDIA-linked arXiv papers on end-to-end autonomous driving, plus closely related VLM / VLA driving papers and adjacent stack papers.
- The 2023+ subset has now been expanded into individual paper notes and a dedicated synthesis note: [[NVIDIA end-to-end AV review 2023-now]].
- The clearest modern line runs from **benchmark critique** to **multimodal planning**, then to **reasoning / VLA / evaluation**.

## Scope and definition

### Inclusion rule
- Paper is on arXiv.
- Paper is clearly about autonomous driving, end-to-end driving, multimodal planning, driving-oriented VLM / VLA, or a directly supporting NVIDIA autonomous-driving stack component.
- Paper has a strong NVIDIA-company linkage through authorship or affiliation.

### Exclusion rule
- Pure robotics papers with no clear autonomous-driving relevance.
- Generic VLM papers with no autonomous-driving connection.
- Non-arXiv-only publications.

## Taxonomy

### Family 1 — Core end-to-end driving / planning
- PilotNet-era imitation driving.
- Open-loop end-to-end planning.
- Multimodal planning through trajectory scoring and proposal selection.

### Family 2 — VLA / VLM driving
- Counterfactual reasoning, language-grounded driving, and reasoning-action models.
- Human-aligned evaluation through VLM judges.

### Family 3 — Adjacent autonomous-driving stack papers
- Holistic perception, scene completion, and camera-encoder design that support the main line.

## Timeline
- **2016** — End to End Learning for Self-Driving Cars.
- **2017** — Explaining How a Deep Neural Network Trained with End-to-End Learning Steers a Car.
- **2020** — The NVIDIA PilotNet Experiments.
- **2023** — [[Ego Status All You Need 2312]].
- **2023** — [[SSCBench 2306]].
- **2024** — [[OmniDrive 2405]].
- **2024** — [[Hydra-MDP 2406]].
- **2024** — [[DriveNeXt Camera Encoder 2407]].
- **2024** — [[Latent World Models for Covariate Shift 2409]].
- **2025** — [[Generalized Trajectory Scoring 2506]].
- **2025** — [[DriveCritic 2510]].
- **2025** — [[Alpamayo-R1 2511]].
- **2026** — [[Fast-ThinkAct 2601]].

## Key papers / evidence

### Core end-to-end / planning papers
1. **End to End Learning for Self-Driving Cars** — arXiv:1604.07316.
2. **Explaining How a Deep Neural Network Trained with End-to-End Learning Steers a Car** — arXiv:1704.07911.
3. **The NVIDIA PilotNet Experiments** — arXiv:2010.08776.
4. [[Ego Status All You Need 2312]] — arXiv:2312.03031.
5. [[Hydra-MDP 2406]] — arXiv:2406.06978.
6. [[Latent World Models for Covariate Shift 2409]] — arXiv:2409.16663.
7. [[Generalized Trajectory Scoring 2506]] — arXiv:2506.06664.

### VLA / VLM driving papers
8. [[OmniDrive 2405]] — arXiv:2405.01533.
9. [[DriveCritic 2510]] — arXiv:2510.13108.
10. [[Alpamayo-R1 2511]] — arXiv:2511.00088.
11. [[Fast-ThinkAct 2601]] — arXiv:2601.09708.

### Important adjacent stack papers
12. [[SSCBench 2306]] — arXiv:2306.09001.
13. [[DriveNeXt Camera Encoder 2407]] — arXiv:2407.07276.

## Main takeaways
1. NVIDIA’s public driving line has at least three visible recent phases: **benchmark critique**, **multimodal planning**, and **reasoning / evaluation**.
2. The strongest 2024–2025 planning papers are mostly **score-based** rather than simple direct-regression planners.
3. Recent NVIDIA work is no longer only about “predict the next path”; it is also about **counterfactual reasoning**, **reasoning-action alignment**, and **human-aligned evaluation**.

## Practical reading order
1. [[Ego Status All You Need 2312]]
2. [[Hydra-MDP 2406]]
3. [[Generalized Trajectory Scoring 2506]]
4. [[OmniDrive 2405]]
5. [[DriveCritic 2510]]
6. [[Alpamayo-R1 2511]]
7. [[Fast-ThinkAct 2601]]

## Related notes
- [[NVIDIA end-to-end AV review 2023-now]]
- [[score-based e2e autonomous driving papers 2022-2026]]
- [[score-based e2e autonomous driving review]]
- [[transfuser]]
