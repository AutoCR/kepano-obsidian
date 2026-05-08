---
categories:
  - "[[Papers]]"
title: "Sparse4D v2: Recurrent Temporal Fusion with Sparse Model"
authors:
  - Xuewu Lin
  - Tianwei Lin
  - Zixiang Pei
  - Lichao Huang
  - Zhizhong Su
affiliation:
  - "Horizon Robotics"
venue: arXiv
year: 2023
doi:
url: https://arxiv.org/abs/2305.14018
pdf: https://arxiv.org/pdf/2305.14018v2
field:
  - autonomous-driving
  - 3d-object-detection
keywords:
  - sparse-3d-detection
  - multi-view-camera
  - temporal-fusion
  - recurrent-fusion
  - efficient-deformable-aggregation
  - nuscenes
status:
  - read
rating:
dataset:
  - nuScenes
method:
  - recurrent temporal fusion
  - sparse object instances
  - efficient deformable aggregation
  - camera parameter encoding
  - dense depth supervision
task: camera-only multi-view temporal 3D object detection
created: 2026-05-08
updated: 2026-05-08
tags:
  - paper
related:
  - "[[Sparse4D 2211]]"
  - "[[Deformable DETR 2010]]"
  - "[[SparseDrive 2405]]"
  - "[[SparseDriveV2 2603]]"
---

One sentence: **Sparse4D v2 makes Sparse4D faster and stronger by propagating sparse object instances through time, instead of sampling many historical image frames again.**

# Core takeaways

- Sparse4D v2 is an improved version of [[Sparse4D 2211]].
- The main change is the temporal fusion module.
- Sparse4D v1 samples features from many historical frames.
- Sparse4D v2 carries object instances forward frame by frame.
- This makes temporal fusion cheaper and more suitable for long history.
- The paper also adds Efficient Deformable Aggregation, camera parameter encoding, and dense depth supervision.

# Temporal fusion

![[Attachments/sparse4dv2-2305/figure1-temporal-fusion-comparison.png]]

## Sparse4D v1

Sparse4D v1 uses **multi-frame sampling**.

At the current frame, it projects the current anchors into historical frames.
Then it samples image features from each historical frame.
Then it fuses these sampled features.

Simple example:

```python
# at frame t
f_t   = sample(frame_t, anchor_t)
f_t1  = sample(frame_t_minus_1, projected_anchor)
f_t2  = sample(frame_t_minus_2, projected_anchor)

fused = fuse([f_t, f_t1, f_t2])
```

If the model uses more history frames, it does more sampling.
So the cost grows with the number of frames.

```text
more frames -> more image sampling -> more memory and slower inference
```

## Sparse4D v2

Sparse4D v2 uses **recurrent temporal fusion**.

It keeps sparse object instances from the previous frame.
Each instance has:

```text
anchor/state: 3D box, yaw, velocity
instance feature: semantic feature vector
anchor embedding: encoded anchor state
```

At the next frame, the model projects the previous anchor to the current frame.
The instance feature is carried forward.

```python
# from frame t-1 to frame t
anchor_t = project(anchor_t_minus_1, ego_motion, velocity)
feature_t = feature_t_minus_1
```

Then the current decoder refines this propagated instance with current image features.

This avoids re-sampling all old frames.
The cost is closer to constant per frame.

# Model structure

![[Attachments/sparse4dv2-2305/figure2-sparse4dv2-framework.png]]

Sparse4D v2 has an encoder-decoder structure.

1. Multi-view images go into a backbone and neck.
2. The model gets current image features.
3. Previous-frame instances are projected by ego motion.
4. A single-frame layer handles new objects.
5. Multi-frame layers refine temporal objects.
6. High-confidence instances are used as output and passed to the next frame.

The paper uses 900 instance anchors:

| Type | Count |
|---|---:|
| temporal instances from previous output | 600 |
| current-frame instances from single-frame layer | 300 |
| total | 900 |

# Efficient Deformable Aggregation

EDA means **Efficient Deformable Aggregation**.

It computes the same mathematical result as normal deformable aggregation.
The difference is the implementation.

## Normal deformable aggregation

Normal DA usually does separate steps:

```python
sampled = sample_features(image_features, points)
weighted = sampled * weights
output = sum(weighted)
```

This creates large intermediate tensors.
Those tensors must be written to and read from GPU memory.
That is slow and memory-heavy.

## EDA

EDA fuses the steps into one CUDA operation.

```python
output = 0
for point in points:
    value = sample(image_features, point)
    output += value * weight
```

It samples, multiplies, and accumulates directly.
It does not store large intermediate sampled features.

Simple example:

```text
features = [10, 20, 30, 40]
weights  = [0.1, 0.2, 0.3, 0.4]

output = 10*0.1 + 20*0.2 + 30*0.3 + 40*0.4
       = 30
```

DA and EDA both output `30`.
EDA is faster because it avoids extra memory writes and reads.

Reported benefit in the paper:

| Setting | Without EDA | With EDA |
|---|---:|---:|
| training memory | 6328 MB | 3100 MB |
| max batch size | 3 | 8 |
| training time | 23.5 h | 14.5 h |
| inference memory | 925 MB | 432 MB |
| inference FPS | 13.7 | 20.3 |

# Camera parameter encoding

Sparse4D v2 explicitly gives camera parameters to the network.

This helps the model understand:

- camera intrinsics
- camera extrinsics
- object position relative to each camera

This is useful for orientation estimation.
The paper reports that removing camera parameter encoding hurts mAP and orientation error.

# Dense depth supervision

Sparse methods can be hard to train at the beginning.
Sparse4D v2 adds dense depth supervision using LiDAR point clouds.

This depth head is only used during training.
It is removed during inference.

The goal is simple:

```text
better depth learning during training -> better 3D detection
```

# Relation to StreamPETR

Sparse4D v2 is similar to StreamPETR at the idea level.

Both methods propagate sparse object-level representations through time.
Both avoid repeatedly sampling many historical image frames.

But they are not the same implementation.

| Item | StreamPETR | Sparse4D v2 |
|---|---|---|
| temporal unit | object query | object instance |
| memory | FIFO memory queue | propagated instances from previous output |
| motion handling | ego transform + motion-aware layer normalization | explicit 3D anchor projection |
| fusion module | propagation transformer | Sparse4D decoder with temporal cross-attention |
| new objects | current initialized queries | single-frame layer selects new objects |

Simple mental model:

```text
StreamPETR:
Keep a queue of past object queries.
Let current queries attend to them.

Sparse4D v2:
Move previous detected 3D object instances into the current frame.
Then refine them.
```

# Results

On nuScenes validation with ResNet50 and 256 × 704 input:

| Method | mAP | NDS | FPS |
|---|---:|---:|---:|
| StreamPETR | 0.432 | 0.537 | 26.7 |
| Sparse4D v2 | 0.439 | 0.539 | 20.3 |

On nuScenes validation with ResNet101 and 512 × 1408 input:

| Method | mAP | NDS | FPS |
|---|---:|---:|---:|
| Sparse4D v1 | 0.444 | 0.550 | 2.9 |
| StreamPETR | 0.504 | 0.592 | 6.4 |
| Sparse4D v2 | 0.505 | 0.594 | 8.4 |

Sparse4D v2 is especially strong at higher image resolution.
The sparse decoder cost is less tied to image resolution.

# My understanding

Sparse4D v2 is mainly about making sparse temporal perception practical.

The important idea is not only better accuracy.
The important idea is **how to keep temporal memory cheaply**.

Sparse4D v1 looks back into old images.
Sparse4D v2 carries object memory forward.

That is why v2 can use temporal information with much lower cost.
