---
categories:
  - "[[Papers]]"
title: "Adaptive Time Step Flow Matching for Autonomous Driving Motion Planning"
authors:
  - Ananya Trivedi
  - Anjian Li
  - Mohamed Elnoor
  - Yusuf Umut Ciftci
  - Avinash Singh
  - Jovin D'sa
  - Sangjae Bae
  - David Isele
  - Taskin Padir
  - Faizan M. Tariq
affiliation:
  - "Honda Research Institute, San Jose, CA 95134, USA"
  - "Northeastern University, Boston, MA 02115, USA"
  - "Princeton University, Princeton, NJ 08544, USA"
  - "University of Maryland, College Park, MD 20742, USA"
  - "Stanford University, Stanford, CA 94305, USA"
venue: arXiv
year: 2026
doi:
url: https://arxiv.org/abs/2602.10285
pdf: https://arxiv.org/pdf/2602.10285v1
field:
  - autonomous-driving
  - end-to-end-planning
keywords:
  - flow matching
  - adaptive inference steps
  - WOMD
  - real-time planning
  - convex QP
  - joint prediction-planning
status:
  - read
rating:
dataset:
  - Waymo Open Motion Dataset
method:
  - conditional flow matching
  - adaptive step selection
  - variance estimator
  - QP post-processing
task: end-to-end autonomous driving planning
created: 2026-04-20
updated:
tags:
  - paper
related:
  - "[[score-based e2e autonomous driving papers 2022-2026]]"
  - "[[score-based e2e autonomous driving review]]"
  - "[[GuideFlow 2511]]"
  - "[[GoalFlow 2503]]"
---

One-line takeaway: **This paper makes flow-matching planning practical online by adapting the number of inference steps per scene and adding a cheap constraint-satisfying QP smoother.**

# Core takeaways
- Adapts the number of inference steps online depending on scene difficulty.
- Jointly predicts other agents and the ego plan.
- Uses a lightweight QP to improve smoothness and dynamic feasibility.

# Method summary
The planner uses conditional flow matching to jointly model surrounding-agent motion and ego planning, then chooses the number of integration steps online with a variance estimator. A convex quadratic-program refinement stage enforces smoothness and dynamic constraints at low extra cost.

# Benchmarks / results
On WOMD, the paper reports improved smoothness, goal-reaching, and dynamic-constraint adherence relative to transformer, diffusion, and consistency baselines, while maintaining 20 Hz on an RTX 3070.

# Relation to the score-based E2E AD lineage
This paper is close to the score-based trend but sits more in motion-planning literature than the NAVSIM/Bench2Drive score-based benchmark ecosystem.

# Caveats
- Evaluated on WOMD rather than the main E2E AD benchmark stack used by many other papers here.
- Post-processing means behavior is not purely the learned generator.

# Links
- Paper list: [[score-based e2e autonomous driving papers 2022-2026]]
- Review: [[score-based e2e autonomous driving review]]
