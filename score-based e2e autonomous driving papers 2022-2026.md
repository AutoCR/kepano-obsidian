---
categories:
  - "[[Evergreen]]"
title: "Score-based end-to-end autonomous driving papers (2022-2026)"
created: 2026-04-20
updated: 2026-04-21
tags:
  - 0🌲
  - literature-map
  - autonomous-driving
  - score-based
  - e2e
  - planning
sources: []
period:
  start: 2022
  end: 2026
related:
  - "[[score-based e2e autonomous driving review]]"
  - "[[Hydra-MDP 2406]]"
  - "[[SparseDrive 2405]]"
  - "[[SparseDriveV2 2603]]"
  - "[[DiffusionDrive 2411]]"
  - "[[FeaXDrive 2604]]"
---

# Score-based end-to-end autonomous driving papers (2022-2026)

## Executive summary

This note is a **reading map** for the score-based end-to-end autonomous driving literature from 2022 to 2026. I use **score-based** in the broad planning sense: the system evaluates multiple candidate trajectories and then **scores, ranks, selects, or refines** them, instead of directly regressing a single future.

The literature now clusters into three practical families:
1. **Pure scoring / selection** over a fixed or structured trajectory set.
2. **Proposal + scoring hybrids** that generate candidates with anchors or refinement and still rely on a final scorer.
3. **Diffusion / flow-matching planners with explicit quality control**, where generation is central but scoring, guidance, or RL still governs the final behavior.

If I only want the core lineage, I would start with [[Hydra-MDP 2406]], [[SparseDrive 2405]], [[Hydra-MDP++ 2503]], [[DriveSuprim 2506]], [[SparseDriveV2 2603]], and then read [[DiffusionDrive 2411]] plus a few hybrid follow-ups.

## Scope

### Inclusion rule
A paper belongs here if it does at least one of the following:
- constructs **multiple candidate trajectories**,
- learns a **trajectory scorer / selector / critic**,
- or combines **trajectory generation** with an explicit **ranking, guidance, refinement, or RL quality-control step**.

### Exclusion rule
I do **not** treat every end-to-end planner or every diffusion planner as part of this line. A method is only included if it is clearly connected to the broader **candidate generation + scoring / selection** viewpoint.

## Taxonomy

### A. Core score-based / trajectory-scoring line
These papers make **trajectory scoring or selection** the center of the planner.

#### 2024
- **[[Hydra-MDP 2406]]**  
  *Hydra-MDP: End-to-end Multimodal Planning with Multi-target Hydra-Distillation*  
  arXiv: [2406.06978](https://arxiv.org/abs/2406.06978)  
  Why it matters: the clearest early formulation of planning as **candidate trajectories + learned multi-target scoring**.

- **[[SparseDrive 2405]]**  
  *SparseDrive: End-to-End Autonomous Driving via Sparse Scene Representation*  
  arXiv: [2405.19620](https://arxiv.org/abs/2405.19620)  
  Why it matters: efficient multimodal planning on sparse scene representations; important adjacent root of the later scoring-heavy line.

- **[[ComDrive 2410]]**  
  *ComDrive: Comfort-Oriented End-to-End Autonomous Driving*  
  arXiv: [2410.05051](https://arxiv.org/abs/2410.05051)  
  Why it matters: adds comfort-aware candidate generation and scoring.

#### 2025
- **[[Hydra-NeXt 2503]]**  
  *Hydra-NeXt: Robust Closed-Loop Driving with Open-Loop Training*  
  arXiv: [2503.12030](https://arxiv.org/abs/2503.12030)  
  Focus: push the Hydra family toward better closed-loop robustness.

- **[[Hydra-MDP++ 2503]]**  
  *Hydra-MDP++: Advancing End-to-End Driving via Expert-Guided Hydra-Distillation*  
  arXiv: [2503.12820](https://arxiv.org/abs/2503.12820)  
  Focus: stronger teacher targets and improved Hydra distillation.

- **[[HMAD 2505]]**  
  *HMAD: Advancing E2E Driving with Anchored Offset Proposals and Simulation-Supervised Multi-target Scoring*  
  arXiv: [2505.23129](https://arxiv.org/abs/2505.23129)  
  Focus: anchored proposals plus simulator-supervised scoring.

- **[[DriveSuprim 2506]]**  
  *DriveSuprim: Towards Precise Trajectory Selection for End-to-End Planning*  
  arXiv: [2506.06659](https://arxiv.org/abs/2506.06659)  
  Focus: argues that **selection precision** is still a bottleneck.

- **[[Generalized Trajectory Scoring 2506]]**  
  *Generalized Trajectory Scoring for End-to-end Multimodal Planning*  
  arXiv: [2506.06664](https://arxiv.org/abs/2506.06664)  
  Focus: scores both static and dynamically generated candidate sets.

- **[[ZTRS 2510]]**  
  *ZTRS: Zero-Imitation End-to-end Autonomous Driving with Trajectory Scoring*  
  arXiv: [2510.24108](https://arxiv.org/abs/2510.24108)  
  Focus: replaces imitation learning with offline RL in a score-based planner.

- **[[Navigation Compliance Gap 2512]]**  
  *Closing the Navigation Compliance Gap in End-to-end Autonomous Driving*  
  arXiv: [2512.10660](https://arxiv.org/abs/2512.10660)  
  Focus: shows that strong score-based planners can still ignore route commands unless compliance is trained explicitly.

#### 2026
- **[[CdDrive 2602]]**  
  *A Unified Candidate Set with Scene-Adaptive Refinement via Diffusion for End-to-End Autonomous Driving*  
  arXiv: [2602.03112](https://arxiv.org/abs/2602.03112)  
  Focus: static vocabulary plus scene-adaptive diffusion refinement.

- **[[SparseDriveV2 2603]]**  
  *SparseDriveV2: Scoring is All You Need for End-to-End Autonomous Driving*  
  arXiv: [2603.29163](https://arxiv.org/abs/2603.29163)  
  Focus: the strongest recent statement that **scoring itself can scale**.

### B. Closely related diffusion / flow-matching / hybrid planners
These papers are not always pure score-based methods, but they matter because they keep the same **multimodal candidate-quality-control** perspective.

#### 2024
- **[[DiffusionDrive 2411]]**  
  *DiffusionDrive: Truncated Diffusion Model for End-to-End Autonomous Driving*  
  arXiv: [2411.15139](https://arxiv.org/abs/2411.15139)  
  Role in the lineage: a key generative counterpart to the strict scoring family.

#### 2025
- **[[VDT-Auto 2502]]** — VLM-guided diffusion planning.  
  arXiv: [2502.20108](https://arxiv.org/abs/2502.20108)

- **[[GoalFlow 2503]]** — goal-driven flow matching with explicit planning structure.  
  arXiv: [2503.05689](https://arxiv.org/abs/2503.05689)

- **[[TransDiffuser 2505]]** — diffusion-based multimodal trajectory generation.  
  arXiv: [2505.09315](https://arxiv.org/abs/2505.09315)

- **[[ReCogDrive 2506]]** — reinforced cognitive planning framework.  
  arXiv: [2506.08052](https://arxiv.org/abs/2506.08052)

- **[[GuideFlow 2511]]** — constraint-guided flow matching for planning.  
  arXiv: [2511.18729](https://arxiv.org/abs/2511.18729)

- **[[DiffusionDriveV2 2512]]** — RL-constrained truncated diffusion planning.  
  arXiv: [2512.07745](https://arxiv.org/abs/2512.07745)

#### 2026
- **[[RAPiD 2602]]** — distills a diffusion planner into a deterministic real-time policy.  
  arXiv: [2602.07339](https://arxiv.org/abs/2602.07339)

- **[[Adaptive Time Step Flow Matching 2602]]** — adaptive flow-matching planning.  
  arXiv: [2602.10285](https://arxiv.org/abs/2602.10285)

- **[[HDP 2602]]** — stronger diffusion planning system design.  
  arXiv: [2602.22801](https://arxiv.org/abs/2602.22801)

- **[[HAD 2604]]** — hierarchical diffusion plus RL.  
  arXiv: [2604.03581](https://arxiv.org/abs/2604.03581)

- **[[FeaXDrive 2604]]** — feasibility-aware trajectory-centric diffusion planning.  
  arXiv: [2604.12656](https://arxiv.org/abs/2604.12656)

## Timeline at a glance

- **2022** — no clear Hydra-style score-based E2E planner is visible yet.
- **2023** — the field is still mostly pre-formation from the score-based perspective.
- **2024** — the line becomes recognizable with [[Hydra-MDP 2406]], [[SparseDrive 2405]], [[ComDrive 2410]], and [[DiffusionDrive 2411]].
- **2025** — the literature expands into stronger scorers, anchored proposals, navigation-aware scoring, and diffusion / flow hybrids.
- **2026** — the field enters a scaling phase: denser vocabularies, better refinement, more RL, and stronger claims that scoring can reach SOTA.

## How I would read the field

### Minimal core reading path
1. [[Hydra-MDP 2406]]
2. [[SparseDrive 2405]]
3. [[Hydra-MDP++ 2503]]
4. [[DriveSuprim 2506]]
5. [[SparseDriveV2 2603]]

### If I also want the generative counterpoint
6. [[DiffusionDrive 2411]]
7. [[CdDrive 2602]]
8. [[FeaXDrive 2604]]

### If I care about RL / post-training
9. [[ZTRS 2510]]
10. [[DiffusionDriveV2 2512]]
11. [[HAD 2604]]
12. [[RAPiD 2602]]

## Main takeaways

- The key conceptual shift is from **predicting one trajectory** to **comparing many plausible futures**.
- The strongest disagreement in the literature is no longer whether multimodality matters; it is whether **better scoring** or **better candidate generation** matters more.
- Recent papers increasingly suggest a hybrid answer: **generation proposes, scoring governs**.

## Related notes

- Review note: [[score-based e2e autonomous driving review]]
- Core paper anchors: [[Hydra-MDP 2406]], [[SparseDriveV2 2603]], [[DiffusionDrive 2411]], [[FeaXDrive 2604]]
