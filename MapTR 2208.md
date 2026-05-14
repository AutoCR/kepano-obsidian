---
categories:
  - "[[Papers]]"
title: "MapTR: Structured Modeling and Learning for Online Vectorized HD Map Construction"
authors:
  - Bencheng Liao
  - Shaoyu Chen
  - Xinggang Wang
  - Tianheng Cheng
  - Qian Zhang
  - Wenyu Liu
  - Chang Huang
affiliation:
  - Huazhong University of Science & Technology
  - Horizon Robotics
venue: ICLR
year: 2023
doi:
url: https://arxiv.org/abs/2208.14437
pdf: https://arxiv.org/pdf/2208.14437v2
field:
  - autonomous-driving
  - online-mapping
  - computer-vision
keywords:
  - vectorized-hd-map
  - online-map-construction
  - DETR
  - hierarchical-matching
  - permutation-equivalent-modeling
  - BEV
status:
  - read
rating:
dataset:
  - nuScenes
method:
  - permutation-equivalent point-set modeling
  - hierarchical query embedding
  - hierarchical bipartite matching
  - point-to-point loss
  - edge direction loss
task: online vectorized HD map construction
created: 2026-05-13
updated: 2026-05-13
tags:
  - paper
  - autonomous-driving
  - online-mapping
related:
  - "[[MapTRv2 2308]]"
  - "[[Deformable DETR 2010]]"
  - "[[SparseDrive 2405]]"
  - "[[SparseDriveV2 2603]]"
---

One-line takeaway: **MapTR turns online vectorized HD map construction into a DETR-like set prediction problem, and its key trick is to treat multiple valid point orders of the same map element as equivalent.**

# Core takeaways

- HD map elements are predicted as **vectorized instances**, not rasterized BEV segmentation masks.
- MapTR models each map element as a point set plus a group of equivalent permutations: $(V, \Gamma)$.
- This solves the ambiguity of point order for polylines and polygons.
- The model predicts all map instances and all points in parallel, instead of using slow autoregressive point decoding.
- MapTR-nano reaches real-time speed: **25.1 FPS** on RTX 3090.
- MapTR-tiny with camera-only input outperforms prior camera+LiDAR vectorized map methods reported in the paper.

# Problem

Online HD map construction needs to predict local static road elements around the ego vehicle, such as:

- pedestrian crossings,
- lane dividers,
- road boundaries.

Earlier methods have two main issues:

1. **Rasterized map prediction** loses instance-level vector structure.
2. **Vectorized map prediction** often needs post-processing or sequential point generation, which is slow.

MapTR asks whether a DETR-like parallel set prediction framework can directly output vectorized HD map elements.

# Permutation-equivalent modeling

![[Attachments/maptr-2208/figure3-permutation-equivalent-modeling.png]]

Map elements do not have a unique point order.

For a **polyline**, either endpoint can be the start point:

```text
[v0, v1, v2]
[v2, v1, v0]
```

For a **polygon**, any vertex can be the start point, and the polygon can be traversed clockwise or counter-clockwise.

So MapTR represents a map element as:

$$
\mathcal{V} = (V, \Gamma)
$$

where:

- $V = \{v_j\}_{j=0}^{N_v-1}$ is the unordered point set.
- $\Gamma$ is the group of equivalent valid permutations.
- $N_v$ is the number of sampled points for one map element.

For a polyline, $\Gamma$ contains two permutations: forward and reverse.

For a polygon, $\Gamma$ contains $2N_v$ permutations: all cyclic shifts in both directions.

This is the real contribution: **the model does not force an arbitrary fixed order as supervision.**

# Model structure

![[Attachments/maptr-2208/figure4-maptr-architecture.png]]

MapTR has an encoder-decoder structure.

## Map encoder

Input is surround-view camera images. A backbone extracts image features, then a 2D-to-BEV module converts them into BEV features.

The paper uses GKT by default, but also tests IPM, LSS, and Deformable Attention.

## Map decoder

The decoder uses hierarchical queries:

```text
instance query + point query = hierarchical query
```

Formally:

$$
q_{ij}^{hie} = q_i^{ins} + q_j^{pt}
$$

where:

- $q_i^{ins}$ represents the $i$-th map instance.
- $q_j^{pt}$ represents the $j$-th point slot shared across instances.
- $q_{ij}^{hie}$ represents point $j$ of instance $i$.

This lets the Transformer exchange information both:

- across map instances,
- across points within the same map instance.

# Hierarchical matching

MapTR training has two assignment levels.

## 1. Instance-level matching

This decides which predicted map element matches which ground-truth map element.

The pairwise matching cost is:

$$
L_{ins\_match}(\hat{y}_{\pi(i)}, y_i)
=
L_{Focal}(\hat{p}_{\pi(i)}, c_i)
+
L_{position}(\hat{V}_{\pi(i)}, V_i)
$$

where:

- $\hat{p}_{\pi(i)}$ is the predicted class score.
- $c_i$ is the GT class label.
- $\hat{V}_{\pi(i)}$ is the predicted point set.
- $V_i$ is the GT point set.

Then Hungarian matching finds the global best assignment between predictions and GT instances.

## Position matching cost

The position cost measures geometric compatibility between a predicted point set and a GT point set.

The paper compares Chamfer distance cost and point-to-point cost. The default/better version uses point-to-point cost:

$$
L_{position}(\hat{V}, V)
=
\min_{\gamma \in \Gamma}
\sum_{j=0}^{N_v-1}
D_{Manhattan}(\hat{v}_j, v_{\gamma(j)})
$$

That means:

1. Try all valid equivalent GT point orders $\gamma$.
2. Compute total Manhattan distance between predicted points and the permuted GT points.
3. Use the smallest distance as the instance-level position cost.

This cost is used inside Hungarian matching to answer:

> Which predicted map instance should match this GT map instance?

## 2. Point-level matching

After instance-level matching, the predicted instance and GT instance are already paired.

Point-level matching decides the best point order inside that matched pair:

$$
\hat{\gamma}
=
\arg\min_{\gamma \in \Gamma}
\sum_{j=0}^{N_v-1}
D_{Manhattan}(\hat{v}_j, v_{\gamma(j)})
$$

This selected $\hat{\gamma}$ is then used for point-to-point loss and edge direction loss.

## Difference between instance matching and point-level matching

| Aspect | Instance-level matching | Point-level matching |
|---|---|---|
| Question | Which predicted instance matches which GT instance? | Which GT point order supervises this matched instance? |
| Scope | Across all predicted/GT instances | Inside one matched instance pair |
| Uses class score? | Yes | No |
| Uses Hungarian algorithm? | Yes | No |
| Uses min over $\Gamma$? | Yes, inside the position cost | Yes, directly |
| Output | Instance assignment $\hat{\pi}$ | Point-order assignment $\hat{\gamma}$ |

So they share the same inner point-order search, but the overall algorithms are different:

- **Instance-level matching** = Hungarian matching over instances, with classification cost plus position cost.
- **Point-level matching** = direct minimum over equivalent point permutations for one matched instance pair.

# Losses

The training loss is:

$$
L = \lambda L_{cls} + \alpha L_{p2p} + \beta L_{dir}
$$

## Classification loss

Focal loss over matched predictions and target classes.

## Point-to-point loss

After choosing the best point permutation $\hat{\gamma}$:

$$
L_{p2p}
=
\sum_i 1_{\{c_i \ne \varnothing\}}
\sum_{j=0}^{N_v-1}
D_{Manhattan}(\hat{v}_{\hat{\pi}(i),j}, v_{i,\hat{\gamma}_i(j)})
$$

## Edge direction loss

Point loss only supervises vertices. Edge direction loss additionally supervises the geometry of the segment between adjacent points.

This helps represent shapes more accurately.

# Results

On nuScenes val set:

| Method | Input | mAP | FPS |
|---|---:|---:|---:|
| HDMapNet | Camera | 23.0 | 0.8 |
| HDMapNet | Camera + LiDAR | 31.0 | 0.5 |
| VectorMapNet | Camera | 40.9 | 2.9 |
| VectorMapNet | Camera + LiDAR | 45.2 | <2.9 |
| MapTR-nano | Camera | 45.9 | 25.1 |
| MapTR-tiny | Camera | 58.7 | 11.2 |

# Ablations

## Permutation-equivalent modeling

| Modeling | mAP |
|---|---:|
| Fixed-order point sequence | 44.4 |
| Permutation-equivalent modeling | 50.3 |

This is the most important ablation. The gain is especially large for pedestrian crossings because polygons have more valid point orders.

## Edge direction loss

| Setting | mAP |
|---|---:|
| without edge direction loss | 48.2 |
| with edge direction loss | 50.3 |

## 2D-to-BEV method

| BEV transform | mAP |
|---|---:|
| IPM | 46.2 |
| LSS | 49.5 |
| Deformable Attention | 49.7 |
| GKT | 50.3 |

# Caveats

- Experiments focus on nuScenes and three map element classes.
- The method predicts local vectorized elements, not full road graph topology or traffic-rule semantics.
- Calibration errors can hurt performance. Small translation/rotation noise is acceptable, but larger errors degrade mAP strongly.
- The quality still depends on the 2D-to-BEV representation.

# My understanding

MapTR's main idea is simple but important: **do not make the network learn an arbitrary point order that has no geometric meaning.**

The model uses DETR-style instance matching to decide which prediction corresponds to which GT map element. Inside that matching, it already searches over valid point permutations to compute a fair geometric cost. Then, after instances are matched, it again selects the best point permutation for supervision.

This makes training more stable and makes vectorized HD map prediction practical in a real-time, end-to-end framework.

# Related reading

- [[Deformable DETR 2010]]
- [[SparseDrive 2405]]
- [[SparseDriveV2 2603]]
