---
categories:
  - "[[Papers]]"
title: "Closing the Navigation Compliance Gap in End-to-end Autonomous Driving"
authors:
  - Hanfeng Wu
  - Marlon Steiner
  - Michael Schmidt
  - Alvaro Marcos-Ramiro
  - Christoph Stiller
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2512.10660
pdf: https://arxiv.org/pdf/2512.10660v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - navigation compliance
  - command following
  - trajectory scoring
  - Hydra distillation
  - controllability
  - NAVSIM
status:
  - read
rating:
dataset:
  - NAVSIM navtest
  - NAVSIM navcontrol
method:
  - command-conditioned score-based planner
  - Hydra distillation
  - navigation-compliance loss
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[Hydra-MDP 2406]]"
  - "[[Hydra-MDP++ 2503]]"
---

One-line takeaway: **This paper shows that score-based planners can score well yet ignore alternate commands, and fixes that with navigation-aware Hydra distillation plus explicit compliance training.**

# Core takeaways
- Identifies that score-based planners can score well while still ignoring route commands.
- Adds explicit command conditioning and navigation-compliance supervision.
- Improves both standard NAVSIM metrics and command-following behavior.

# Method summary
The paper proposes NaviHydra, a command-conditioned planner in the Hydra family. It gathers route-consistent candidates, distills from a rule-based simulator, and adds a dedicated navigation-compliance loss so the scorer learns to distinguish on-route from off-route but otherwise feasible candidates.

# Benchmarks / results
On NAVSIM navtest, the paper reports 92.7 PDM Score; on the controllability-focused navcontrol split, it reports 91.4 PDM Score and 36.2 CM, clearly ahead of prior Hydra baselines.

# Relation to the score-based E2E AD lineage
This is a direct descendant of Hydra-MDP-style scoring, focused on controllability rather than multimodal generation. It highlights a blind spot of optimizing only conventional planning metrics.

# Caveats
- Tightly tied to route-command availability and simulator distillation.
- Strongest variant uses richer sensing, complicating some comparisons.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
