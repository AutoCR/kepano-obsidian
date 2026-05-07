---
categories:
  - "[[Papers]]"
title: "Sparse4D: Multi-view 3D Object Detection with Sparse Spatial-Temporal Fusion"
authors:
  - Xuewu Lin
  - Tianwei Lin
  - Zixiang Pei
  - Lichao Huang
  - Zhizhong Su
affiliation:
  - "Horizon Robotics"
venue: arXiv
year: 2022
doi:
url: https://arxiv.org/abs/2211.10581
pdf: https://arxiv.org/pdf/2211.10581v2
field:
  - autonomous-driving
  - 3d-object-detection
keywords:
  - sparse-3d-detection
  - multi-view-camera
  - temporal-fusion
  - deformable-4d-aggregation
  - nuscenes
status:
  - read
rating:
dataset:
  - nuScenes
method:
  - sparse 3D anchors
  - deformable 4D aggregation
  - depth reweighting
  - iterative box refinement
task: camera-only multi-view 3D object detection
created: 2026-05-07
updated:
tags:
  - paper
related:
  - "[[Deformable DETR 2010]]"
  - "[[SparseDrive 2405]]"
  - "[[SparseDriveV2 2603]]"
  - "[[mAP]]"
---

One sentence: **Sparse4D is a camera-only 3D detector. It avoids dense BEV construction. It refines sparse 3D anchors by sampling image features from multiple views, scales, and timestamps.**

# Core takeaways

- The paper wants sparse 3D detection to match strong BEV-based methods.
- BEV methods are accurate, but dense BEV features can be expensive.
- Sparse4D keeps only a small set of object anchors.
- Each anchor samples features from image feature maps.
- The key idea is **deformable 4D aggregation**.
- “4D” means **3D space + time**.
- The model also adds a **depth reweight module** to reduce depth ambiguity.

# Model structure

![[Attachments/sparse4d-2211/figure2-model-structure.png]]

Sparse4D has two main parts.

1. **Image encoder**
   - Input is multi-view camera images.
   - The paper uses 6 nuScenes cameras.
   - A backbone and FPN produce multi-scale image features.
   - The model keeps features from several timestamps.

2. **Sparse decoder**
   - The decoder starts from 3D anchors and instance features.
   - It has several refinement modules.
   - Each module updates the anchor box and the instance feature.
   - The paper uses 6 refinement modules by default.

Each 3D anchor stores:

- center: `x, y, z`
- size: `w, h, l`
- yaw angle: `sin(yaw), cos(yaw)`
- velocity: `vx, vy, vz`

Default settings:

| Item | Value |
|---|---:|
| 3D anchors | 900 |
| refinement modules | 6 |
| feature scales | 4 |
| fixed keypoints | 7 |
| learnable keypoints | 6 |
| total keypoints per anchor | 13 |
| image size | 640 × 1600 |
| default backbone | ResNet101 |

# Deformable 4D aggregation

![[Attachments/sparse4d-2211/figure3-deformable-4d-aggregation.png]]

This is the main module of Sparse4D.

It turns one anchor into a better instance feature.

The process has three steps.

## 1. Generate 4D keypoints

For each 3D anchor, the model creates keypoints.

There are two kinds of keypoints.

**Fixed keypoints**:

- one point at the box center
- six points at the centers of the six box faces

So there are 7 fixed keypoints.

**Learnable keypoints**:

- The model predicts them from the instance feature.
- They can move to useful object parts.
- The paper uses 6 learnable keypoints.

Then the model extends these keypoints to past frames.

It uses:

- object velocity, to estimate where the object was before
- ego motion, to align old frames with the current frame

This gives keypoints in both space and time.

## 2. Sample image features

Each 4D keypoint is projected into each camera image.

Then the model samples image features by bilinear interpolation.

For one anchor, the sampled features cover:

- multiple keypoints
- multiple timestamps
- multiple camera views
- multiple feature scales

This is sparse. The model does not build a full BEV feature map.

## 3. Fuse the sampled features

The fusion is hierarchical.

First, it fuses features across views and scales.

Then, it fuses features across time.

Finally, it fuses features across keypoints.

The output is a stronger instance feature for the anchor.

This feature is used to refine the 3D box.

# Depth reweight module

Camera-only 3D detection has a depth problem.

Different 3D points can project to the same 2D image area.

So two anchors may sample similar image features, even if their depths are different.

Sparse4D adds a small depth module.

It predicts a depth confidence for the anchor.

If the anchor has a wrong depth, its feature is downweighted.

This helps the model reject boxes that look right in 2D but are wrong in 3D.

The depth loss uses the labeled 3D box center depth.

It does not need dense LiDAR depth supervision.

# Training

Sparse4D is trained end to end.

It uses Hungarian matching, similar to DETR.

The loss has three parts:

- classification loss: focal loss
- box regression loss: L1 loss
- depth loss: binary cross entropy

Training details:

| Item | Setting |
|---|---|
| optimizer | AdamW |
| backbone learning rate | 2e-5 |
| other learning rate | 2e-4 |
| schedule | cosine annealing |
| initialization | FCOS3D pretraining |
| validation experiments | 24 epochs |
| test-set model | 48 epochs |

# Results

On nuScenes validation with ResNet101:

| Method | Temporal | mAP | NDS |
|---|---:|---:|---:|
| DETR3D | no | 0.349 | 0.434 |
| BEVFormer-S | no | 0.375 | 0.448 |
| Sparse4D T=1 | no | 0.382 | 0.451 |
| BEVFormer | yes | 0.416 | 0.517 |
| BEVDepth | yes | 0.412 | 0.535 |
| Sparse4D T=4 | yes | 0.436 | 0.541 |
| Sparse4D T=9 | yes | 0.445 | 0.547 |

On nuScenes test with VoVNet-99:

| Method | mAP | NDS |
|---|---:|---:|
| DETR3D | 0.412 | 0.479 |
| BEVFormer | 0.481 | 0.569 |
| PETRv2 | 0.490 | 0.582 |
| BEVDistill | 0.496 | 0.594 |
| Sparse4D | 0.511 | 0.595 |

Sparse4D is much stronger than DETR3D.

It is also competitive with BEV methods.

# Ablation notes

## Learnable keypoints help

They give the model freedom to choose useful object locations.

The gain is small but consistent.

## Depth reweighting helps

It reduces false anchors with wrong depth.

The gain is also small but consistent.

## Motion compensation is important

Temporal features help.

Ego-motion compensation gives a large boost.

Object-motion compensation improves velocity estimation.

## More history helps

More past frames improve the result.

The paper still sees gains up to around 9 or 10 frames.

# My understanding

Sparse4D is like a stronger DETR3D.

DETR3D samples one sparse 3D reference point.

Sparse4D samples many keypoints for each anchor.

It also samples across time.

This gives the anchor much richer evidence.

The model avoids dense BEV features, so it is more deployment-friendly.

The most important idea is simple:

> Do not build a full BEV map. Keep sparse anchors. Let each anchor look at the right image locations across cameras and time.

# Relation to later sparse driving papers

Sparse4D is mainly a perception paper.

It is related to [[SparseDrive 2405]] because both use a sparse scene representation style.

But the focus is different.

- Sparse4D focuses on camera-only 3D detection.
- SparseDrive extends sparse representation to detection, tracking, mapping, prediction, and planning.
- SparseDriveV2 focuses more on planning and trajectory scoring.

# Useful links

- Paper: https://arxiv.org/abs/2211.10581
- PDF: https://arxiv.org/pdf/2211.10581v2
- Code: https://github.com/linxuewu/Sparse4D
