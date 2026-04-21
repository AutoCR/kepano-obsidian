---
categories:
  - "[[Evergreen]]"
title: "Review of score-based end-to-end autonomous driving"
created: 2026-04-20
updated: 2026-04-21
tags:
  - 0🌲
  - literature-review
  - autonomous-driving
  - score-based
  - e2e
  - planning
sources: []
period:
  start: 2022
  end: 2026
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[Hydra-MDP 2406]]"
  - "[[SparseDriveV2 2603]]"
  - "[[DiffusionDrive 2411]]"
  - "[[DriveSuprim 2506]]"
  - "[[FeaXDrive 2604]]"
---

# Review of score-based end-to-end autonomous driving

This note synthesizes the papers collected in [[score-based e2e autonomous driving papers 2022-2026]].

## Executive summary

Score-based end-to-end autonomous driving has become one of the most convincing planning paradigms because it matches the **multimodal nature of driving**. Instead of forcing the model to output a single future, these methods compare multiple candidate futures and use a learned objective to decide which one is best.

The field now contains three connected families:
1. **Pure scoring / selection** over a static or structured candidate set.
2. **Proposal + scoring hybrids** that generate better local candidates but still depend on an explicit scorer.
3. **Diffusion / flow-matching planners** that broaden candidate generation while retaining guidance, ranking, or RL-based quality control.

My current view is simple: the long-term winner will probably be **hybrid**. The literature is converging toward systems where **generation proposes and scoring governs**.

## 1. Scope and definition

In this review, **score-based end-to-end autonomous driving** means:
- the planner represents **multiple candidate trajectories**,
- then uses a learned mechanism to **score, rank, select, refine, or optimize** them,
- rather than directly regressing one future trajectory.

This broad definition is useful because it captures the real design debate in the field: not regression versus generation, but **how candidate futures are represented, improved, and judged**.

## 2. Historical development

### 2.1 Before the score-based wave
Earlier end-to-end planners often regressed a single future trajectory. That approach was simple, but it did not fit the fact that driving is inherently multimodal: at many moments, more than one future is valid.

### 2.2 Hydra-MDP as the conceptual turning point
[[Hydra-MDP 2406]] is the clearest early milestone. Its real contribution is not just a better planner head. It reframes planning as **candidate generation plus learned multi-target scoring**, with simulator-derived objectives for safety, progress, and comfort.

This shift is important because it turns driving quality from an implicit imitation target into an explicit ranking problem.

### 2.3 Expansion into stronger scoring and stronger generation
After Hydra-MDP, the literature splits in two directions.

**Direction A: scoring is the main bottleneck.**  
This line includes [[Hydra-MDP++ 2503]], [[Hydra-NeXt 2503]], [[HMAD 2505]], [[DriveSuprim 2506]], [[Generalized Trajectory Scoring 2506]], [[ZTRS 2510]], [[Navigation Compliance Gap 2512]], and [[SparseDriveV2 2603]]. These papers argue that better supervision, better candidate structure, and better ranking are enough to keep improving performance.

**Direction B: candidate generation is still too weak.**  
This line includes [[DiffusionDrive 2411]], [[GoalFlow 2503]], [[CdDrive 2602]], [[DiffusionDriveV2 2512]], [[HAD 2604]], [[HDP 2602]], and [[FeaXDrive 2604]]. These papers argue that good scoring is not enough if the right future is never proposed in the first place.

### 2.4 The current phase
The field no longer looks like a clean contest between score-based planners and generative planners. It increasingly looks like a contest between different ways of doing **quality control over multimodal futures**.

## 3. Taxonomy of methods

### 3.1 Pure scoring over static or structured candidate sets
Representative papers:
- [[Hydra-MDP 2406]]
- [[Hydra-MDP++ 2503]]
- [[DriveSuprim 2506]]
- [[ZTRS 2510]]
- [[SparseDriveV2 2603]]

Common pattern:
- define a fixed or factorized candidate space,
- evaluate each candidate with learned metrics,
- select the highest-quality one.

Why this works:
- it aligns naturally with multimodality,
- it makes safety and comfort objectives easy to attach,
- it is often easier to debug than free-form generation.

Main weakness:
- it cannot recover a maneuver that is missing from the candidate set.

### 3.2 Proposal + scoring hybrids
Representative papers:
- [[HMAD 2505]]
- [[Generalized Trajectory Scoring 2506]]
- [[CdDrive 2602]]
- [[ComDrive 2410]]

Common pattern:
- generate or refine candidates with anchors, offsets, or diffusion,
- preserve a learned scorer as the final decision-maker.

Why this matters:
- it improves local adaptivity,
- but keeps the interpretability of explicit trajectory selection.

Main weakness:
- the pipeline becomes harder to analyze because gains may come from the generator, the scorer, or the interaction between them.

### 3.3 Diffusion / flow-matching dominant planners
Representative papers:
- [[DiffusionDrive 2411]]
- [[GoalFlow 2503]]
- [[GuideFlow 2511]]
- [[DiffusionDriveV2 2512]]
- [[HAD 2604]]
- [[FeaXDrive 2604]]
- [[HDP 2602]]

Common pattern:
- use diffusion or flow matching to generate multimodal trajectories,
- then improve quality with scoring, guidance, hierarchy, constraints, or RL.

Why this matters:
- these methods relax the coverage limit of static vocabularies,
- especially in interaction-heavy or unusual scenes.

Main weakness:
- inference is usually harder to control, optimize, and deploy.

## 4. The main design axes

### 4.1 Candidate source: static coverage vs adaptive generation
A central question is whether good planning mostly requires:
- a **better candidate vocabulary**, or
- a **more adaptive candidate generator**.

[[SparseDriveV2 2603]] represents the strongest answer from the first camp: if scoring and coverage are strong enough, scoring alone can scale. [[DiffusionDrive 2411]], [[GoalFlow 2503]], and [[CdDrive 2602]] represent the second camp: some scenes require adaptive generation beyond static vocabularies.

My synthesis: routine scenes often favor structured scoring, while difficult interactive scenes increasingly favor adaptive generation.

### 4.2 What is being scored?
The field has moved from scoring “expert-likeness” to scoring richer notions of trajectory quality:
- safety,
- progress,
- comfort,
- route compliance,
- feasibility,
- or decomposed multi-objective reward terms.

[[Hydra-MDP 2406]] made simulator metrics central. [[Navigation Compliance Gap 2512]] showed that benchmark scores can still miss route obedience. [[FeaXDrive 2604]] argued that **trajectory-space feasibility** deserves its own modeling.

### 4.3 Open-loop optimization vs closed-loop reality
This remains one of the largest unresolved gaps.

- [[Hydra-NeXt 2503]] directly targets the open-loop / closed-loop gap.
- [[DriveSuprim 2506]] and [[SparseDriveV2 2603]] show that strong offline score-based methods can be competitive, but they still depend heavily on offline supervision.
- [[ZTRS 2510]], [[DiffusionDriveV2 2512]], [[ReCogDrive 2506]], [[HAD 2604]], and [[FeaXDrive 2604]] all move toward RL or post-training.

The trend is clear: **pure imitation is no longer enough**.

### 4.4 Representation of planning uncertainty
Different papers represent uncertainty in very different ways:
- discrete trajectory vocabularies,
- factorized path / velocity vocabularies,
- anchor-conditioned denoising,
- goal-conditioned generation,
- flow matching with constraints,
- or feasibility-aware trajectory diffusion.

This matters because the representation determines which failures the planner can express and correct.

## 5. What the strongest papers are really saying

### 5.1 Hydra-MDP and descendants
The Hydra family says planning should be treated as **learned ranking**, not direct regression.

### 5.2 DriveSuprim and SparseDriveV2
These papers say the field may have underestimated how much performance is still available from **better selection precision** and **better candidate coverage** inside a score-based formulation.

### 5.3 DiffusionDrive and descendants
These papers say that some failures are not ranking failures at all. They are **proposal failures**: the right future was never in the set.

### 5.4 Hybrid papers
Hybrid papers imply that the real engineering answer may not be ideological purity. It may be a system with:
- strong structured priors,
- adaptive candidate refinement,
- and a robust learned selector or critic.

## 6. Strengths of the score-based paradigm

### 6.1 Better alignment with multimodal driving
Driving is naturally multimodal, so comparing candidate futures is more realistic than regressing one trajectory.

### 6.2 Natural interface for safety-aware supervision
Candidate trajectories are easy to score with safety, comfort, progress, route, and feasibility metrics.

### 6.3 Interpretability and controllability
A scorer-based system often makes it clearer why one candidate wins and another loses.

### 6.4 Compatibility with RL and teacher supervision
The same action space can support imitation, simulator teachers, and RL fine-tuning.

## 7. Weaknesses and unresolved problems

### 7.1 Candidate coverage remains a hard ceiling
If the right maneuver is absent, even a perfect scorer cannot recover it.

### 7.2 Scores are often benchmark-shaped
Many objectives are tightly coupled to benchmark metrics, which risks overfitting to what is measured.

### 7.3 Closed-loop generalization remains fragile
Open-loop success does not guarantee reliable closed-loop behavior.

### 7.4 System complexity is rising quickly
Recent systems often combine scoring, refinement, diffusion, RL, route conditioning, and guidance, which makes them harder to analyze and reproduce.

## 8. My synthesis

I think the literature now supports five conclusions:

1. **The real conceptual shift is from regression to comparison.**
2. **Pure scoring is still highly competitive.**
3. **Adaptive generation matters more in hard interactions.**
4. **Quality control matters as much as proposal quality.**
5. **The long-term winner will probably be hybrid.**

In short, the field may converge on a planner where **generation proposes and scoring governs**.

## 9. Practical reading order

1. [[Hydra-MDP 2406]]
2. [[SparseDrive 2405]]
3. [[Hydra-MDP++ 2503]]
4. [[DriveSuprim 2506]]
5. [[SparseDriveV2 2603]]
6. [[DiffusionDrive 2411]]
7. [[CdDrive 2602]]
8. [[DiffusionDriveV2 2512]]
9. [[FeaXDrive 2604]]
10. [[Navigation Compliance Gap 2512]]

## 10. Bottom line

**Score-based end-to-end autonomous driving is compelling because it turns planning into structured comparison over many plausible futures, aligns naturally with safety-aware supervision, and scales from static ranking to modern hybrid generative systems.**

The main open question is no longer whether score-based planning works. The real question is **which combination of candidate structure, generation mechanism, and quality-control objective gives the best trade-off between safety, controllability, robustness, and efficiency**.

## Related notes

- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Core papers: [[Hydra-MDP 2406]], [[DriveSuprim 2506]], [[SparseDriveV2 2603]], [[DiffusionDrive 2411]], [[FeaXDrive 2604]]
