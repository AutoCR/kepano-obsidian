---
categories:
  - "[[Papers]]"
title: "MapTRv2: An End-to-End Framework for Online Vectorized HD Map Construction"
authors:
  - Bencheng Liao
  - Shaoyu Chen
  - Yunchi Zhang
  - Bo Jiang
  - Qian Zhang
  - Wenyu Liu
  - Chang Huang
  - Xinggang Wang
affiliation:
  - Institute of Artificial Intelligence, Huazhong University of Science and Technology
  - School of Electronic Information and Communications, Huazhong University of Science and Technology
  - Horizon Robotics
venue: arXiv
year: 2023
doi:
url: https://arxiv.org/abs/2308.05736
pdf: https://arxiv.org/pdf/2308.05736v2
field:
  - autonomous-driving
  - online-mapping
  - computer-vision
keywords:
  - vectorized-hd-map
  - online-map-construction
  - DETR
  - hierarchical-matching
  - decoupled-self-attention
  - one-to-many-training
  - dense-supervision
status:
  - read
rating:
dataset:
  - nuScenes
  - Argoverse2
method:
  - permutation-equivalent point-set modeling
  - hierarchical query embedding
  - hierarchical bipartite matching
  - decoupled self-attention
  - auxiliary one-to-many set prediction
  - auxiliary dense supervision
task: online vectorized HD map construction
created: 2026-05-14
updated: 2026-05-14
tags:
  - paper
  - autonomous-driving
  - online-mapping
related:
  - "[[MapTR 2208]]"
  - "[[Deformable DETR 2010]]"
  - "[[SparseDrive 2405]]"
  - "[[SparseDriveV2 2603]]"
---

One-line takeaway: **MapTRv2 is MapTR plus centerlines, decoupled self-attention, dense auxiliary supervision, and auxiliary one-to-many training, making vectorized online HD map construction faster to train and stronger on both 2D and 3D map benchmarks.**

# Core takeaways

- MapTRv2 is a journal extension of [[MapTR 2208]].
- It keeps MapTR's key idea: model each map element as a point set with equivalent valid point permutations.
- It adds **directed lane centerline modeling**, which is more useful for planning than only boundaries/dividers/crossings.
- It adds **decoupled self-attention** to reduce memory and computation in the map decoder.
- It adds **auxiliary dense supervision**: depth prediction, PV segmentation, and BEV segmentation.
- It adds **auxiliary one-to-many set prediction** to create more positive samples during training.
- The paper reports real-time inference speed and stronger performance than MapTR v1.

# Difference from MapTR

Short version:

```text
MapTRv2 = MapTR v1
        + lane centerlines
        + decoupled self-attention
        + dense auxiliary losses
        + one-to-many training
        + 3D map construction on Argoverse2
```

| Aspect | MapTR | MapTRv2 |
|---|---|---|
| Main paper type | ICLR 2023 conference paper | journal/arXiv extension |
| Map elements | crossings, dividers, boundaries | also directed lane centerlines |
| Decoder self-attention | vanilla attention over flattened instance-point queries | decoupled inter-instance and intra-instance attention |
| Training assignment | one-to-one DETR-style matching | one-to-one main branch + auxiliary one-to-many branch |
| Extra supervision | mainly vector set prediction losses | depth + PV segmentation + BEV segmentation |
| Evaluation | mainly nuScenes 2D vector maps | nuScenes plus Argoverse2 2D/3D vector maps |

# Decoupled self-attention

MapTR has hierarchical queries. If there are:

- $N$ map-instance queries,
- $N_v$ point queries per instance,

then the decoder can flatten them into $N \times N_v$ tokens.

Vanilla self-attention attends over all flattened tokens:

```text
sequence length = N × Nv
cost ≈ O((N × Nv)^2)
```

MapTRv2 instead separates two kinds of interaction:

1. **Inter-instance attention**: different map elements communicate with each other.
2. **Intra-instance / point attention**: points inside the same map element communicate with each other.

So the simplified complexity becomes:

```text
O(N^2 + Nv^2)
```

instead of:

```text
O((N × Nv)^2)
```

## Simple example

Assume:

```text
N = 50 map elements
Nv = 20 points per element
```

Vanilla attention uses:

```text
50 × 20 = 1000 tokens
1000^2 = 1,000,000 pairwise interactions
```

Decoupled attention roughly uses:

```text
inter-instance: 50^2 = 2,500
intra-instance: 20^2 = 400
```

The exact implementation still has projection and batching details, but the intuition is simple:

> Do not let every point of every map element attend to every point of every other map element. First model map-element interaction, then model point interaction inside each element.

## Why this helps

- Inter-instance attention lets nearby lanes, dividers, crossings, and boundaries exchange context.
- Intra-instance attention lets the points of one polyline/polygon keep a coherent shape.
- It saves memory while preserving the two interactions that matter most.

The paper's ablation shows that only inter-instance attention saves memory but loses a little accuracy; decoupled attention recovers point-level interaction and performs better.

# Auxiliary one-to-many training

DETR-style training usually uses one-to-one matching:

```text
one GT object -> one positive prediction
```

This is clean for inference, but training can converge slowly because there are few positive samples.

MapTRv2 adds an auxiliary one-to-many branch during training. The main branch still uses normal one-to-one matching. The auxiliary branch repeats the GT map elements $K$ times, so each GT can supervise multiple predictions.

The paper describes the auxiliary loss as:

$$
L_{one2many} = L_{Hungarian}(\hat{Y}', Y')
$$

where $Y'$ is the repeated-and-padded GT set and $\hat{Y}'$ is the auxiliary prediction set.

## Simple example

Suppose one scene has three GT map elements:

```text
GT1 = lane divider
GT2 = road boundary
GT3 = pedestrian crossing
```

Standard one-to-one training might match:

```text
GT1 -> pred_7
GT2 -> pred_13
GT3 -> pred_2
```

Only 3 predictions are positive; the rest are background.

With one-to-many training and $K = 3$, the GT set is repeated:

```text
GT1, GT2, GT3,
GT1, GT2, GT3,
GT1, GT2, GT3
```

Now each GT can supervise multiple predictions:

```text
GT1 -> pred_7,  pred_21, pred_34
GT2 -> pred_13, pred_8,  pred_40
GT3 -> pred_2,  pred_19, pred_27
```

So there are 9 positive supervised predictions instead of 3.

Important detail: **the auxiliary one-to-many branch is for training only**. At inference time, the model can still use the normal one-to-one prediction branch.

# Auxiliary dense supervision

MapTRv2 also adds dense auxiliary losses:

$$
L_{dense} = \alpha_d L_{depth} + \alpha_b L_{BEVSeg} + \alpha_p L_{PVSeg}
$$

- **Depth loss**: uses LiDAR points to render GT depth maps for perspective-view features.
- **PV segmentation**: supervises map-related foreground in perspective-view images.
- **BEV segmentation**: rasterizes map GT onto a BEV canvas and supervises BEV foreground.

My understanding: these losses do not replace vectorized map prediction. They provide extra training signals so the image/BEV features learn geometry and map foreground faster.

# Results discussed

On nuScenes:

| Method | Backbone / setting | Epochs | mAP | FPS |
|---|---|---:|---:|---:|
| MapTR | R50 | 24 | 50.3 | 15.1 |
| MapTR | R50 | 110 | 58.7 | 15.1 |
| MapTRv2 | R50 | 24 | 61.5 | 14.1 |
| MapTRv2 | R50 | 110 | 68.7 | 14.1 |
| MapTRv2 | VoVNetV2-99 | 110 | 73.4 | 9.9 |

Key point:

> MapTRv2 trained for 24 epochs already beats MapTR trained for 110 epochs: 61.5 mAP vs. 58.7 mAP.

On Argoverse2, MapTRv2 also reports strong 2D and 3D vectorized map construction results, substantially outperforming VectorMapNet in the paper's comparison.

# My understanding

MapTR's main contribution is the representation and matching: map elements are point sets with valid equivalent point orders.

MapTRv2 keeps that idea but improves the training and decoder efficiency:

- **Decoupled attention** makes the query interaction cheaper and more structured.
- **One-to-many training** gives the decoder more positive samples, so training converges faster.
- **Dense supervision** helps the visual features learn map geometry before the final vector prediction loss has to do everything.

So MapTRv2 is not a completely different method. It is a stronger and more practical version of MapTR.

# Related reading

- [[MapTR 2208]]
- [[Deformable DETR 2010]]
- [[SparseDrive 2405]]
- [[SparseDriveV2 2603]]
