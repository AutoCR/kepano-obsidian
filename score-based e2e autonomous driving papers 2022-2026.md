     1|---
     2|categories:
     3|  - "[[Papers]]"
     4|title: "Score-based end-to-end autonomous driving papers (2022-2026)"
     5|authors: []
     6|venue: literature note
     7|year: 2026
     8|doi:
     9|url:
    10|pdf:
    11|field:
    12|  - autonomous-driving
    13|  - end-to-end-planning
    14|keywords:
    15|  - score-based
    16|  - trajectory-scoring
    17|  - diffusion-planner
    18|  - flow-matching
    19|status:
    20|  - collected
    21|rating:
    22|dataset:
    23|  - NAVSIM
    24|  - Bench2Drive
    25|  - nuScenes
    26|method:
    27|  - scoring-based planning
    28|  - trajectory selection
    29|  - diffusion planning
    30|  - flow matching
    31|task: end-to-end autonomous driving planning
    32|created: 2026-04-20
    33|updated:
    34|tags:
    35|  - paper
    36|  - survey
    37|related:
    38|  - "[[Hydra-MDP 2406]]"
    39|  - "[[SparseDrive 2405]]"
    40|  - "[[SparseDriveV2 2603]]"
    41|  - "[[FeaXDrive 2604]]"
    42|  - "[[navsim评估方法]]"
    43|---
    44|
    45|# Score-based E2E AD paper list
    46|
    47|这份清单以 **[[Hydra-MDP 2406]]** 为参照，重点收集 arXiv 上 **candidate trajectory generation + learned scoring / selection** 这一类端到端自动驾驶方法；同时也把和这条路线紧密相关的 **diffusion / flow matching + scoring** 混合方法放进来，便于横向比较。
    48|
    49|## Quick take
    50|
    51|- **2022-2023**：几乎没有明确成型的 Hydra-MDP 风格 E2E trajectory-scoring 论文。
    52|- **2024**：这条路线开始成型，代表作包括 [[Hydra-MDP 2406]]、[[SparseDrive 2405]]、ComDrive、DiffusionDrive。
    53|- **2025-2026**：快速扩展成三类：
    54|  1. 纯 **scoring / selection**
    55|  2. **proposal + scoring** 混合方法
    56|  3. **diffusion / flow matching + scoring-selection**
    57|
    58|## A. Strict score-based / trajectory-scoring line
    59|
    60|### 2024
    61|- **[[Hydra-MDP 2406]]**  
    62|  *Hydra-MDP: End-to-end Multimodal Planning with Multi-target Hydra-Distillation*  
    63|  arXiv: [2406.06978](https://arxiv.org/abs/2406.06978)  
    64|  关键词：multi-teacher distillation, trajectory vocabulary, trajectory scoring
    65|
    66|- **[[SparseDrive 2405]]**  
    67|  *SparseDrive: End-to-End Autonomous Driving via Sparse Scene Representation*  
    68|  arXiv: [2405.19620](https://arxiv.org/abs/2405.19620)  
    69|  关键词：sparse representation, multimodal planning, planning-oriented end-to-end AD
    70|
    71|- **[[ComDrive 2410]]**  
    72|  *ComDrive: Comfort-Oriented End-to-End Autonomous Driving*  
    73|  arXiv: [2410.05051](https://arxiv.org/abs/2410.05051)  
    74|  关键词：DDPM planner + adaptive trajectory scorer, comfort-oriented planning
    75|
    76|### 2025
    77|- **[[Hydra-NeXt 2503]]**  
    78|  *Hydra-NeXt: Robust Closed-Loop Driving with Open-Loop Training*  
    79|  arXiv: [2503.12030](https://arxiv.org/abs/2503.12030)  
    80|  关键词：Hydra family follow-up, closed-loop robustness
    81|
    82|- **[[Hydra-MDP++ 2503]]**  
    83|  *Hydra-MDP++: Advancing End-to-End Driving via Expert-Guided Hydra-Distillation*  
    84|  arXiv: [2503.12820](https://arxiv.org/abs/2503.12820)  
    85|  关键词：expert-guided Hydra distillation, multi-head decoder
    86|
    87|- **[[HMAD 2505]]**  
    88|  *HMAD: Advancing E2E Driving with Anchored Offset Proposals and Simulation-Supervised Multi-target Scoring*  
    89|  arXiv: [2505.23129](https://arxiv.org/abs/2505.23129)  
    90|  关键词：anchored proposals, multi-target scoring
    91|
    92|- **[[DriveSuprim 2506]]**  
    93|  *DriveSuprim: Towards Precise Trajectory Selection for End-to-End Planning*  
    94|  arXiv: [2506.06659](https://arxiv.org/abs/2506.06659)  
    95|  关键词：selection-based planning, precise trajectory selection
    96|
    97|- **[[Generalized Trajectory Scoring 2506]]**  
    98|  *Generalized Trajectory Scoring for End-to-end Multimodal Planning*  
    99|  arXiv: [2506.06664](https://arxiv.org/abs/2506.06664)  
   100|  关键词：general trajectory scorer, static vs dynamic candidate sets
   101|
   102|- **[[ZTRS 2510]]**  
   103|  *ZTRS: Zero-Imitation End-to-end Autonomous Driving with Trajectory Scoring*  
   104|  arXiv: [2510.24108](https://arxiv.org/abs/2510.24108)  
   105|  关键词：trajectory scoring, zero-imitation
   106|
   107|- **[[Navigation Compliance Gap 2512]]**  
   108|  *Closing the Navigation Compliance Gap in End-to-end Autonomous Driving*  
   109|  arXiv: [2512.10660](https://arxiv.org/abs/2512.10660)  
   110|  关键词：trajectory-scoring planners, command following, navigation compliance
   111|
   112|### 2026
   113|- **[[CdDrive 2602]]**  
   114|  *A Unified Candidate Set with Scene-Adaptive Refinement via Diffusion for End-to-End Autonomous Driving*  
   115|  arXiv: [2602.03112](https://arxiv.org/abs/2602.03112)  
   116|  关键词：static vocabulary + adaptive diffusion refinement + joint scoring
   117|
   118|- **[[SparseDriveV2 2603]]**  
   119|  *SparseDriveV2: Scoring is All You Need for End-to-End Autonomous Driving*  
   120|  arXiv: [2603.29163](https://arxiv.org/abs/2603.29163)  
   121|  关键词：factorized vocabulary, coarse-to-fine scoring, scaling of scoring-based planning
   122|
   123|## B. Closely related diffusion / flow-matching / hybrid methods
   124|
   125|### 2024
   126|- **[[DiffusionDrive 2411]]**  
   127|  *DiffusionDrive: Truncated Diffusion Model for End-to-End Autonomous Driving*  
   128|  arXiv: [2411.15139](https://arxiv.org/abs/2411.15139)  
   129|  说明：虽然不是纯 scoring-only，但对后续 generative planner 路线影响很大。
   130|
   131|### 2025
   132|- **[[VDT-Auto 2502]]**  
   133|  *VDT-Auto: End-to-end Autonomous Driving with VLM-Guided Diffusion Transformers*  
   134|  arXiv: [2502.20108](https://arxiv.org/abs/2502.20108)
   135|
   136|- **[[GoalFlow 2503]]**  
   137|  *GoalFlow: Goal-Driven Flow Matching for Multimodal Trajectories Generation in End-to-End Autonomous Driving*  
   138|  arXiv: [2503.05689](https://arxiv.org/abs/2503.05689)  
   139|  说明：goal scoring + flow matching generation。
   140|
   141|- **[[TransDiffuser 2505]]**  
   142|  *TransDiffuser: Diverse Trajectory Generation with Decorrelated Multi-modal Representation for End-to-end Autonomous Driving*  
   143|  arXiv: [2505.09315](https://arxiv.org/abs/2505.09315)
   144|
   145|- **[[ReCogDrive 2506]]**  
   146|  *ReCogDrive: A Reinforced Cognitive Framework for End-to-End Autonomous Driving*  
   147|  arXiv: [2506.08052](https://arxiv.org/abs/2506.08052)
   148|
   149|- **[[GuideFlow 2511]]**  
   150|  *GuideFlow: Constraint-Guided Flow Matching for Planning in End-to-End Autonomous Driving*  
   151|  arXiv: [2511.18729](https://arxiv.org/abs/2511.18729)
   152|
   153|- **[[DiffusionDriveV2 2512]]**  
   154|  *DiffusionDriveV2: Reinforcement Learning-Constrained Truncated Diffusion Modeling in End-to-End Autonomous Driving*  
   155|  arXiv: [2512.07745](https://arxiv.org/abs/2512.07745)
   156|
   157|### 2026
   158|- **[[RAPiD 2602]]**  
   159|  *RAPiD: Real-time Deterministic Trajectory Planning via Diffusion Behavior Priors for Safe and Efficient Autonomous Driving*  
   160|  arXiv: [2602.07339](https://arxiv.org/abs/2602.07339)  
   161|  说明：用 score-regularized policy optimization 从 diffusion planner 中蒸馏高效策略。
   162|
   163|- **[[Adaptive Time Step Flow Matching 2602]]**  
   164|  *Adaptive Time Step Flow Matching for Autonomous Driving Motion Planning*  
   165|  arXiv: [2602.10285](https://arxiv.org/abs/2602.10285)
   166|
   167|- **[[HDP 2602]]**  
   168|  *Unleashing the Potential of Diffusion Models for End-to-End Autonomous Driving*  
   169|  arXiv: [2602.22801](https://arxiv.org/abs/2602.22801)
   170|
   171|- **[[HAD 2604]]**  
   172|  *HAD: Combining Hierarchical Diffusion with Metric-Decoupled RL for End-to-End Driving*  
   173|  arXiv: [2604.03581](https://arxiv.org/abs/2604.03581)
   174|
   175|- **[[FeaXDrive 2604]]**  
   176|  *FeaXDrive: Feasibility-aware Trajectory-Centric Diffusion Planning for End-to-End Autonomous Driving*  
   177|  arXiv: [2604.12656](https://arxiv.org/abs/2604.12656)
   178|
   179|## C. Timeline
   180|
   181|- **2022**: no clear Hydra-style score-based E2E planning paper found on arXiv.
   182|- **2023**: still no strong Hydra-style representative paper found.
   183|- **2024**: the line becomes visible with Hydra-MDP / SparseDrive / ComDrive / DiffusionDrive.
   184|- **2025**: expansion into anchored proposals, stronger scorers, navigation-aware scoring, and flow / diffusion hybrids.
   185|- **2026**: scaling phase — denser vocabularies, hybrid static+dynamic candidates, and the explicit claim that **scoring can scale to SOTA**.
   186|
   187|## D. My shortlist if I only want the core lineage
   188|
   189|1. [[Hydra-MDP 2406]]
   190|2. [[SparseDrive 2405]]
   191|3. [[ComDrive 2410]]
   192|4. [[Hydra-NeXt 2503]]
   193|5. [[Hydra-MDP++ 2503]]
   194|6. [[HMAD 2505]]
   195|7. [[DriveSuprim 2506]]
   196|8. [[Generalized Trajectory Scoring 2506]]
   197|9. [[ZTRS 2510]]
   198|10. [[CdDrive 2602]]
   199|11. [[SparseDriveV2 2603]]
   200|
   201|## E. Notes
   202|
   203|- 这里的 **score-based** 采用的是较宽口径：以 **candidate trajectories + learned scoring / selection** 为主。
   204|- 如果按更严格定义，只保留不依赖 diffusion / flow matching 生成器的工作，则应主要看 A 部分。
   205|- 如果之后要继续扩展，可以新增一张表：
   206|  - `paper | year | score-only / hybrid / generative | candidate source | scoring method | benchmark`
   207|

## F. Review note

- [[score-based e2e autonomous driving review]]
