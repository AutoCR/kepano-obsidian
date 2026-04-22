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
  - https://arxiv.org/abs/2312.03031
  - https://arxiv.org/abs/2406.06978
  - https://arxiv.org/abs/2409.16663
  - https://arxiv.org/abs/2506.06664
  - https://arxiv.org/abs/2511.00088
period:
  start: 2016
  end: 2026
related:
  - "[[Hydra-MDP 2406]]"
  - "[[Generalized Trajectory Scoring 2506]]"
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
---

# NVIDIA end-to-end autonomous driving papers on arXiv

## Executive summary

- This note is a best-effort list of **NVIDIA company-related arXiv papers** on **end-to-end autonomous driving**, plus closely related **VLA / VLM driving** papers and a few important adjacent stack papers.
- The list was assembled by searching across **paper titles**, **arXiv IDs**, **NVIDIA-linked author clusters** (for example Mariusz Bojarski, Urs Müller, Zhiding Yu, Shiyi Lan, José M. Alvarez, Stan Birchfield, Nikolai Smolyanskiy, Jan Kautz), and multiple keyword variants.
- The clearest lineage runs from **PilotNet-era imitation driving** to **open-loop end-to-end planning**, then to **multimodal planning**, and more recently to **reasoning-action / VLA-style driving**.

## Scope and definition

### Inclusion rule
- Paper is on arXiv.
- Paper is clearly about autonomous driving, end-to-end driving, multimodal planning, driving-oriented VLA/VLM, or a directly supporting NVIDIA autonomous-driving stack component.
- Paper has strong NVIDIA-company linkage through title, author cluster, or indexed affiliation.

### Exclusion rule
- Pure robotics papers not focused on autonomous driving.
- Generic vision-language or perception papers with no clear autonomous-driving scope.
- Non-arXiv-only publications.

## Taxonomy

### Family 1 — Core end-to-end driving / planning
- Early imitation-learning and PilotNet papers.
- Open-loop end-to-end autonomous driving papers.
- Multimodal planning papers that directly score or generate driving trajectories.

### Family 2 — VLA / VLM driving
- Driving papers that use vision-language reasoning, action prediction, long-tail reasoning, or multimodal language-grounded evaluation.

### Family 3 — Adjacent autonomous-driving stack papers
- Perception and traffic-prediction papers that are not themselves the driving policy, but are directly relevant to the NVIDIA autonomous-driving stack.

## Timeline

- **2016** — **End to End Learning for Self-Driving Cars** establishes the classic NVIDIA end-to-end imitation-driving line.
- **2016–2017** — Visual explanation work clarifies what the end-to-end controller attends to.
- **2020** — **The NVIDIA PilotNet Experiments** revisits and systematizes the PilotNet line.
- **2023** — **Is Ego Status All You Need for Open-Loop End-to-End Autonomous Driving?** marks the newer NVIDIA open-loop E2E line.
- **2024** — [[Hydra-MDP 2406]] pushes NVIDIA’s multimodal end-to-end planning direction.
- **2024** — World-model-based imitation learning appears in **Mitigating Covariate Shift in Imitation Learning for Autonomous Vehicles Using Latent Space Generative World Models**.
- **2025** — [[Generalized Trajectory Scoring 2506]] extends score-based multimodal planning.
- **2025** — **Alpamayo-R1** and **DriveCritic** bring reasoning / VLA / VLM ideas into the NVIDIA driving line.

## Key papers / evidence

### Core end-to-end / planning papers
1. **End to End Learning for Self-Driving Cars**  
   - arXiv: 1604.07316  
   - Link: https://arxiv.org/abs/1604.07316  
   - Category: core-e2e

2. **VisualBackProp: efficient visualization of CNNs**  
   - arXiv: 1611.05418  
   - Link: https://arxiv.org/abs/1611.05418  
   - Category: core-e2e-support

3. **Explaining How a Deep Neural Network Trained with End-to-End Learning Steers a Car**  
   - arXiv: 1704.07911  
   - Link: https://arxiv.org/abs/1704.07911  
   - Category: core-e2e-support

4. **The NVIDIA PilotNet Experiments**  
   - arXiv: 2010.08776  
   - Link: https://arxiv.org/abs/2010.08776  
   - Category: core-e2e

5. **Is Ego Status All You Need for Open-Loop End-to-End Autonomous Driving?**  
   - arXiv: 2312.03031  
   - Link: https://arxiv.org/abs/2312.03031  
   - Category: core-e2e

6. [[Hydra-MDP 2406]]  
   - Full title: **Hydra-MDP: End-to-end Multimodal Planning with Multi-target Hydra-Distillation**  
   - arXiv: 2406.06978  
   - Link: https://arxiv.org/abs/2406.06978  
   - Category: core-e2e

7. **Mitigating Covariate Shift in Imitation Learning for Autonomous Vehicles Using Latent Space Generative World Models**  
   - arXiv: 2409.16663  
   - Link: https://arxiv.org/abs/2409.16663  
   - Category: core-e2e

8. [[Generalized Trajectory Scoring 2506]]  
   - Full title: **Generalized Trajectory Scoring for End-to-end Multimodal Planning**  
   - arXiv: 2506.06664  
   - Link: https://arxiv.org/abs/2506.06664  
   - Category: core-e2e

### VLA / VLM driving papers
9. **Alpamayo-R1: Bridging Reasoning and Action Prediction for Generalizable Autonomous Driving in the Long Tail**  
   - arXiv: 2511.00088  
   - Link: https://arxiv.org/abs/2511.00088  
   - Category: vla-driving

10. **DriveCritic: Towards Context-Aware, Human-Aligned Evaluation for Autonomous Driving with Vision-Language Models**  
   - arXiv: 2510.13108  
   - Link: https://arxiv.org/abs/2510.13108  
   - Category: vlm/vla-adjacent

11. **OmniDrive: A Holistic Vision-Language Dataset for Autonomous Driving with Counterfactual Reasoning**  
   - arXiv: 2405.01533  
   - Also indexed later as: 2504.04348  
   - Link: https://arxiv.org/abs/2405.01533  
   - Category: vlm/vla-adjacent

12. **Fast-ThinkAct: Efficient Vision-Language-Action Reasoning via Verbalizable Latent Planning**  
   - arXiv: 2601.09708  
   - Link: https://arxiv.org/abs/2601.09708  
   - Category: vla-driving-adjacent

### Important adjacent autonomous-driving stack papers
13. **PredictionNet: Real-Time Joint Probabilistic Traffic Prediction for Planning, Control, and Simulation**  
   - arXiv: 2109.11094  
   - Link: https://arxiv.org/abs/2109.11094  
   - Category: planning-adjacent

14. **MVLidarNet: Real-Time Multi-Class Scene Understanding for Autonomous Driving Using Multiple Views**  
   - arXiv: 2006.05518  
   - Link: https://arxiv.org/abs/2006.05518  
   - Category: perception-adjacent

15. **NVRadarNet: Real-Time Radar Obstacle and Free Space Detection for Autonomous Driving**  
   - arXiv: 2209.14499  
   - Link: https://arxiv.org/abs/2209.14499  
   - Category: perception-adjacent

16. **SSCBench: A Large-Scale 3D Semantic Scene Completion Benchmark for Autonomous Driving**  
   - arXiv: 2306.09001  
   - Link: https://arxiv.org/abs/2306.09001  
   - Category: perception-adjacent

17. **Exploring Camera Encoder Designs for Autonomous Driving Perception**  
   - arXiv: 2407.07276  
   - Link: https://arxiv.org/abs/2407.07276  
   - Category: perception-adjacent

## Main takeaways

1. NVIDIA’s public arXiv driving line has at least two visible waves: the **PilotNet imitation-driving era** and the newer **open-loop / multimodal planning** era.
2. The newer line increasingly mixes **trajectory scoring**, **world-model supervision**, and **multimodal or reasoning-based components**.
3. Recent NVIDIA driving work is no longer just “end-to-end control”; it also includes **VLA/VLM-flavored evaluation and reasoning-action papers** such as **Alpamayo-R1**, **DriveCritic**, and **OmniDrive**.

## Open questions

1. Which of the recent VLA / reasoning papers become part of NVIDIA’s main production planning stack versus remaining evaluation or data papers?
2. How should OmniDrive’s two arXiv IDs be treated in downstream bibliographies?
3. Which missing paper notes in this vault should be created next to complete the NVIDIA driving lineage?

## Practical reading order

1. **End to End Learning for Self-Driving Cars**
2. **The NVIDIA PilotNet Experiments**
3. **Is Ego Status All You Need for Open-Loop End-to-End Autonomous Driving?**
4. [[Hydra-MDP 2406]]
5. [[Generalized Trajectory Scoring 2506]]
6. **Alpamayo-R1: Bridging Reasoning and Action Prediction for Generalizable Autonomous Driving in the Long Tail**
7. **DriveCritic: Towards Context-Aware, Human-Aligned Evaluation for Autonomous Driving with Vision-Language Models**

## Related notes

- [[Hydra-MDP 2406]]
- [[Generalized Trajectory Scoring 2506]]
- [[score-based e2e autonomous driving papers 2022-2026]]
- [[score-based e2e autonomous driving review]]
- [[transfuser]]
