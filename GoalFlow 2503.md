---
categories:
  - "[[Papers]]"
title: "GoalFlow: Goal-Driven Flow Matching for Multimodal Trajectories Generation in End-to-End Autonomous Driving"
authors:
  - Zebin Xing
  - Xingyu Zhang
  - Yang Hu
  - Bo Jiang
  - Tong He
  - Qian Zhang
  - Xiaoxiao Long
  - Wei Yin
affiliation:
  - "School of Artificial Intelligence, University of Chinese Academy of Sciences"
  - "Horizon Robotics"
  - "Nanjing University"
  - "Huazhong University of Science and Technology"
  - "Shanghai AI Laboratory"
venue: arXiv
year: 2025
doi:
url: https://arxiv.org/abs/2503.05689
pdf: https://arxiv.org/pdf/2503.05689v6
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - flow matching
  - rectified flow
  - goal points
  - multimodal trajectories
  - trajectory scoring
  - NAVSIM
  - one-step inference
status:
  - read
rating:
dataset:
  - NAVSIM
  - OpenScene
method:
  - goal point construction
  - goal-conditioned flow matching
  - rectified flow generator
  - trajectory scoring
task: end-to-end autonomous driving planning
created: 2026-04-20
updated: 2026-05-25
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[Flow matching and diffusion models]]"
  - "[[DiffusionDrive 2411]]"
  - "[[GuideFlow 2511]]"
  - "[[CdDrive 2602]]"
  - "[[SparseDrive 2405]]"
  - "[[transfuser]]"
---

One-line takeaway: **GoalFlow uses a selected goal point to guide flow matching, so the model can generate useful multimodal driving trajectories with few denoising steps.**

# Core takeaways
- Driving is multimodal. More than one future can be valid.
- GoalFlow does not let the generator freely create scattered trajectories.
- It first selects a likely and drivable goal point.
- It then generates many goal-conditioned trajectory candidates.
- A trajectory scorer picks the final trajectory.
- The main design is: **goal first, generation second, scoring last**.

# Model structure

![[Attachments/goalflow-2503/figure2-goalflow-architecture.png]]

GoalFlow has three modules.

## 1. Perception module
- The model takes camera images, LiDAR, and ego status.
- It uses a [[transfuser]]-style fusion network.
- The output is a BEV feature.
- The BEV feature describes the road, nearby agents, and scene context.

## 2. Goal point construction module
- The model builds a large vocabulary of candidate goal points.
- Each goal point contains position and heading: `(x, y, θ)`.
- The vocabulary comes from clustered training trajectory endpoints.
- The paper usually uses 4096 or 8192 candidate goal points.

The model scores each goal point with two scores.

| Score | Meaning |
|---|---|
| Distance score | How close the goal point is to the likely future endpoint. |
| DAC score | Whether the goal point is inside the drivable area. |

The final selected goal point should be both likely and drivable.

## 3. Trajectory planning module
- The selected goal point is used as guidance.
- The planner uses rectified flow, a form of [[Flow matching and diffusion models|flow matching]].
- It starts from Gaussian noise.
- It transforms the noise into a realistic trajectory.
- It generates many candidate trajectories.
- The trajectory scorer selects the best one.

# How multimodal generation works

GoalFlow generates multimodal trajectories by sampling many random noises.

Each noise sample becomes one trajectory candidate.

Simple view:

```text
noise sample 1 -> flow model -> trajectory 1
noise sample 2 -> flow model -> trajectory 2
noise sample 3 -> flow model -> trajectory 3
...
```

So one denoising / flow process usually produces one trajectory.

To get many trajectories, the model runs the process for many noise samples. The paper says it generates 128 or 256 trajectories, then scores them.

These runs can be batched on GPU. Conceptually, they are still different samples.

# Denoising steps

Classic diffusion models often need many denoising steps.

GoalFlow uses rectified flow, so it can use fewer steps.

The paper reports:

| Steps | Inference time | PDMS / SPDM |
|---:|---:|---:|
| 20 | 177.8 ms | 89.9 |
| 10 | 92.4 ms | 90.1 |
| 5 | 49.0 ms | 90.3 |
| 1 | 10.4 ms | 88.9 |

This is important. Even one step still works well.

# Trajectory selection

The trajectory scorer trades off two terms.

\[
f(\hat{\tau_i}) = -\lambda_1 \Phi(f_{dis}(\hat{\tau_i})) + \lambda_2 \Phi(f_{pg}(\hat{\tau_i}))
\]

Plain meaning:

- `f_dis` measures distance from the trajectory to the selected goal point.
- Smaller `f_dis` is better.
- `f_pg` means ego progress.
- Larger `f_pg` is better.

Ego progress means how far the ego vehicle moves forward along the route or centerline. It is not just any distance. Sideways movement or backward movement does not count as useful progress.

So the scorer prefers a trajectory that is close to the goal and still moves forward.

# Benchmarks and results

GoalFlow is evaluated on NAVSIM / OpenScene.

Main reported result:

| Method | PDMS / SPDM |
|---|---:|
| TransFuser | 84.0 |
| UniAD | 83.4 |
| PARA-Drive | 84.0 |
| GoalFlow | 90.3 |
| GoalFlow with ground-truth endpoint | 92.1 |
| Human trajectory | 94.8 |

The result suggests that the goal point is very important. When the model gets the ground-truth endpoint as the goal, the score moves closer to human performance.

# Ablation study

| Model | Change | PDMS / SPDM |
|---|---|---:|
| TransFuser | baseline | 84.0 |
| M0 | rectified flow only | 85.6 |
| M1 | + distance goal score | 88.5 |
| M2 | + drivable-area goal score | 89.4 |
| M3 | + trajectory scorer | 90.3 |

The biggest gain comes from goal point guidance.

# Relation to nearby papers

GoalFlow belongs to the generate-and-score family.

- Compared with [[Hydra-MDP 2406]], it does not rely mainly on a fixed trajectory vocabulary. It uses a generative model to create candidates.
- Compared with [[DiffusionDrive 2411]], it uses flow matching and explicit goal guidance for faster sampling.
- Compared with [[SparseDrive 2405]], it still uses goal guidance, but the final trajectories are generated by a flow model.

My current view: GoalFlow is a hybrid method. Generation proposes candidate futures, but scoring still governs the final decision.

# Caveats

- The method depends on goal point quality.
- The goal point vocabulary may miss rare future endpoints.
- The main result is on NAVSIM. More closed-loop testing would make the claim stronger.
- Training is expensive. The paper reports training on 4 nodes, each with 8 RTX 4090 or RTX 3090 GPUs.

# Links
- Paper: https://arxiv.org/abs/2503.05689
- PDF: https://arxiv.org/pdf/2503.05689v6
- Code: https://github.com/YvanYin/GoalFlow
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
