---
categories:
  - "[[Papers]]"
title: "BEVFusion: Multi-Task Multi-Sensor Fusion with Unified Bird's-Eye View Representation"
authors:
  - Zhijian Liu
  - Haotian Tang
  - Alexander Amini
  - Xinyu Yang
  - Huizi Mao
  - Daniela L. Rus
  - Song Han
affiliation:
  - Massachusetts Institute of Technology
  - OmniML
venue: ICRA / arXiv
year: 2024
doi:
url: https://arxiv.org/abs/2205.13542
pdf: https://arxiv.org/pdf/2205.13542v3
field:
  - autonomous-driving
  - 3d-perception
keywords:
  - BEV
  - multi-sensor fusion
  - camera-LiDAR fusion
  - 3D object detection
  - BEV map segmentation
status:
  - read
rating:
dataset:
  - nuScenes
  - Waymo Open Dataset
method:
  - unified BEV representation
  - camera-to-BEV view transformation
  - efficient BEV pooling
  - fully convolutional BEV fusion
task: multi-task camera-LiDAR 3D perception
created: 2026-06-08
updated:
tags:
  - paper
  - autonomous-driving
  - bev
  - sensor-fusion
related:
  - "[[2026-04-02 1945 BEV 调研]]"
  - "[[bev perception 2022-2026 deep-research-report]]"
---

# BEVFusion 2205

## One-line takeaway

BEVFusion converts camera and LiDAR features into the same bird's-eye-view coordinate system, fuses them in BEV space, and uses the fused BEV feature for tasks such as 3D object detection and BEV map segmentation.

## Core motivation

Earlier camera-LiDAR fusion methods often used **point-level fusion**:

```text
LiDAR point → project to image → sample camera feature → decorate LiDAR point
```

This keeps LiDAR geometry, but it throws away most dense camera semantics because camera features are only sampled at sparse LiDAR point locations. The paper argues this is especially harmful for semantic-oriented tasks such as BEV map segmentation.

BEVFusion changes the fusion target:

```text
Camera image features → BEV feature map
LiDAR point cloud     → BEV feature map
                              ↓
                      fuse in BEV space
```

So both dense camera semantics and accurate LiDAR geometry are preserved in a metric top-down grid.

## Model structure

![[Attachments/bevfusion-2205/figure2-model-structure.png]]

The architecture has three main parts:

1. **Camera encoder** extracts multi-view image features.
2. **Camera-to-BEV view transform** lifts camera features into BEV using depth-aware projection and BEV pooling.
3. **LiDAR encoder** voxelizes the point cloud, extracts sparse 3D features, and flattens the height dimension into BEV.
4. **BEV fusion + BEV encoder** combines camera BEV and LiDAR BEV features, then uses convolutional BEV processing to reduce small spatial misalignment.
5. **Task-specific heads** predict 3D boxes and BEV semantic maps.

Simple mental model:

```text
camera: rich semantics, weak direct depth
LiDAR:  accurate geometry, sparse semantics
BEV:    shared metric space where both can align cell-by-cell
```

## Camera-to-BEV transformation

The camera branch follows the Lift-Splat-Shoot style idea:

```text
image feature + predicted depth distribution
→ frustum feature points along camera rays
→ quantize to BEV grid
→ pool features inside each BEV cell
```

The bottleneck is BEV pooling. A naive implementation generates millions of camera feature points per frame and scatters them into BEV cells, which is slow.

BEVFusion optimizes this with:

- **Precomputation**: camera intrinsics/extrinsics are fixed after calibration, so the 3D coordinates and BEV grid indices for camera feature points can be cached.
- **Interval reduction**: after sorting points by BEV grid index, aggregate each contiguous interval directly instead of repeatedly writing prefix-sum intermediates to DRAM.

Reported effect: more than **40× speedup** for the camera-to-BEV transform bottleneck.

## LiDAR-to-BEV transformation

The LiDAR branch is more direct because LiDAR points already have metric 3D coordinates.

Typical flow in the released configs:

```text
LiDAR points
→ voxelization
→ sparse 3D encoder
→ flatten / collapse z-axis
→ LiDAR BEV feature
```

In the nuScenes fusion detector config, the point cloud range is roughly around the ego vehicle, and voxel sizes such as `0.075m × 0.075m × 0.2m` or `0.1m × 0.1m × 0.2m` are used depending on the variant.

## Fusion design

Once both branches are represented as BEV feature maps, fusion is simple. The released implementation uses a convolutional fuser, e.g. `ConvFuser`, that combines camera BEV features and LiDAR BEV features into a 256-channel fused BEV representation.

The important idea is not a complicated attention module. The important idea is the **choice of fusion space**:

```text
not image space
not sparse LiDAR point space
but dense shared BEV space
```

This makes the framework task-agnostic: detection, map segmentation, tracking, motion forecasting, and other BEV-output tasks can share the same fused representation.

## Training details

The paper itself focuses more on the framework and efficiency; the released repository gives the most concrete reproducibility details.

### Reproduction commands from the official repo

Detection model:

```bash
torchpack dist-run -np 8 python tools/train.py \
  configs/nuscenes/det/transfusion/secfpn/camera+lidar/swint_v0p075/convfuser.yaml \
  --model.encoders.camera.backbone.init_cfg.checkpoint pretrained/swint-nuimages-pretrained.pth \
  --load_from pretrained/lidar-only-det.pth
```

Segmentation model:

```bash
torchpack dist-run -np 8 python tools/train.py \
  configs/nuscenes/seg/fusion-bev256d2-lss.yaml \
  --model.encoders.camera.backbone.init_cfg.checkpoint pretrained/swint-nuimages-pretrained.pth
```

### Common nuScenes training setup

- Distributed training: examples use **8 GPUs** via `torchpack dist-run -np 8`.
- Per-GPU samples: `samples_per_gpu: 4`, so the effective batch is about **32** in the default config.
- Workers: `workers_per_gpu: 4`.
- Optimizer: **AdamW**.
- Weight decay: `0.01`.
- Gradient clipping: max norm `35`.
- Mixed precision: config enables `fp16` with dynamic loss scaling.
- Camera input size: `256 × 704` in the default nuScenes setup.
- LiDAR sweeps: current frame plus **9 multi-sweeps** are loaded in the nuScenes pipeline.
- 2D augmentation: resize, rotation, random flip, normalization, optional GridMask.
- 3D augmentation: random scale, rotation, translation, random 3D flip, point shuffle.
- Detection classes: 10 nuScenes foreground classes.
- Map classes: drivable area, pedestrian crossing, walkway, stop line, carpark area, divider.

### Detection-specific details

- Camera backbone: Swin-Tiny-style backbone in the high-performing config, initialized from Swin / nuImages pretrained weights.
- LiDAR branch: sparse voxel encoder.
- Fusion detector config: fine-tunes from a pretrained LiDAR-only detector checkpoint.
- Fusion detection config sets `max_epochs: 6` and learning rate around `2e-4` for the camera+LiDAR fine-tuning stage.
- Detection head: TransFusion-style head with Hungarian assignment and focal/L1 losses.

### Segmentation-specific details

- Config: `configs/nuscenes/seg/fusion-bev256d2-lss.yaml`.
- Camera view transform: LSS-style transform with BEV bounds around `[-51.2m, 51.2m]` and BEV cell size `0.4m` before the segmentation grid transform.
- Optimizer: AdamW with learning rate `1e-4`.
- Schedule: cyclic learning-rate/momentum policy in the released config.
- Map head: BEV segmentation head with focal loss.
- Segmentation evaluation region: `[-50m, 50m] × [-50m, 50m]` around the ego car.

## Results reported in the paper

On nuScenes, BEVFusion reports:

- **3D object detection**: about `+1.3%` higher mAP and NDS compared with strong prior multi-sensor methods.
- **BEV map segmentation**: about `+13.6%` mIoU over LiDAR-only and about `+6%` over camera-only baselines.
- **Efficiency**: about `1.9×` lower computation cost compared with TransFusion-style baselines, and over `40×` acceleration for BEV pooling.

The segmentation gain is the clearest evidence for the paper's thesis: dense camera semantics should not be reduced to sparse LiDAR point decoration.

## My understanding

BEVFusion's main contribution is not just “add camera and LiDAR together.” Its contribution is choosing the right common representation.

Point-level fusion asks:

```text
What image feature belongs to this LiDAR point?
```

BEVFusion asks:

```text
What do all sensors say about this physical BEV cell?
```

That second question is better aligned with autonomous driving perception because objects, lanes, freespace, and maps naturally live in top-down metric space.

## References

- Paper: https://arxiv.org/abs/2205.13542
- PDF: https://arxiv.org/pdf/2205.13542v3
- Code: https://github.com/mit-han-lab/bevfusion
