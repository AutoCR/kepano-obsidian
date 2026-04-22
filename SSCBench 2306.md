---
categories:
  - "[[Papers]]"
title: "SSCBench: A Large-Scale 3D Semantic Scene Completion Benchmark for Autonomous Driving"
authors:
  - Yiming Li
  - Sihang Li
  - Xinhao Liu
  - Moonjun Gong
  - Kenan Li
  - Nuo Chen
  - Zijun Wang
  - Zhiheng Li
  - Tao Jiang
  - Fisher Yu
  - Yue Wang
  - Hang Zhao
  - Zhiding Yu
  - Chen Feng
affiliation:
  - NVIDIA
  - New York University
  - Tsinghua University
  - ETH Zurich
venue: arXiv
year: 2023
doi:
url: https://arxiv.org/abs/2306.09001
pdf: https://arxiv.org/pdf/2306.09001v3.pdf
field:
  - autonomous-driving
  - 3d-perception
keywords:
  - semantic-scene-completion
  - occupancy
  - benchmark
  - voxel-prediction
status:
  - read
rating:
dataset:
  - SSCBench-KITTI-360
  - SSCBench-nuScenes
  - SSCBench-Waymo
method:
  - multi-dataset benchmark construction
  - unified voxel taxonomy
  - modality comparison
task: 3D semantic scene completion for autonomous driving
created: 2026-04-22
updated: 2026-04-22
tags:
  - paper
  - nvidia
related:
  - "[[DriveNeXt Camera Encoder 2407]]"
  - "[[OmniDrive 2405]]"
  - "[[NVIDIA end-to-end autonomous driving papers on arXiv]]"
  - "[[NVIDIA end-to-end AV review 2023-now]]"
---

One-line takeaway: **SSCBench is a large multi-dataset SSC benchmark that makes holistic 3D scene completion more measurable for autonomous driving, exposing large modality and domain gaps.**

# Core takeaways
- This is a **benchmark / dataset paper**, not a new planner.
- It unifies KITTI-360, nuScenes, and Waymo under a common semantic scene completion setup.
- It explicitly compares monocular, trinocular, and LiDAR SSC.
- It highlights how much performance depends on sensor coverage and domain shift.
- It is adjacent to NVIDIA’s E2E line because it strengthens the 3D scene understanding substrate around later planning and VLM work.

# Model details
## Dataset construction pipeline
- Aggregate multiple LiDAR sweeps into denser semantic point clouds.
- Align dynamic objects before accumulation to reduce ghost artifacts.
- Voxelize the final scene into a SemanticKITTI-style 3D grid.
- Map heterogeneous labels from different datasets into a unified semantic taxonomy.
- Mask unknown / unobserved voxels through visibility checks.

## Benchmark design
- Inputs: monocular RGB, trinocular RGB, or point clouds.
- Outputs: voxel occupancy plus semantic class predictions.
- Metrics: geometry IoU and semantic mIoU.
- Benchmarked methods include MonoScene, VoxFormer, TPVFormer, OccFormer, SSCNet, and LMSCNet.

# Training / inference notes
- Total size: about 66k+ frames, around 7.7× the scale of SemanticKITTI.
- The standard voxel volume is 256×256×32 with 0.2 m voxel size.
- The benchmark is explicitly designed to support cross-domain evaluation rather than only in-domain leaderboard tracking.

# Benchmarks / results
- Camera-only SSC performance varies strongly across datasets.
- LiDAR methods dominate especially when sensor coverage is rich, as on Waymo.
- Trinocular inputs improve over monocular inputs.
- Cross-domain evaluation reveals substantial generalization gaps.

# Caveats
- Outdoor SSC labels are still incomplete even after sweep aggregation.
- Label unification simplifies evaluation but compresses some semantic distinctions.
- Since this is a benchmark paper, its “model details” are really benchmark-pipeline details rather than a novel AV policy architecture.

# My understanding
SSCBench is not itself an end-to-end driving paper, but it matters for NVIDIA’s 2023+ stack because it reflects the push toward richer 3D world understanding. In the review timeline, it belongs to the supporting perception layer rather than the planner layer.

# Related reading
- [[DriveNeXt Camera Encoder 2407]]
- [[OmniDrive 2405]]
- [[NVIDIA end-to-end AV review 2023-now]]
