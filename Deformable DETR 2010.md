---
categories:
  - "[[Papers]]"
title: "Deformable DETR: Deformable Transformers for End-to-End Object Detection"
authors:
  - Xizhou Zhu
  - Weijie Su
  - Lewei Lu
  - Bin Li
  - Xiaogang Wang
  - Jifeng Dai
affiliation:
  - SenseTime Research
  - University of Science and Technology of China
  - The Chinese University of Hong Kong
venue: ICLR
year: 2021
doi:
url: https://arxiv.org/abs/2010.04159
pdf: Attachments/deformable-detr-2010/deformable-detr-2010.pdf
field:
  - computer vision
  - object detection
keywords:
  - DETR
  - deformable attention
  - multi-scale attention
  - transformer
status: read
rating:
dataset:
  - COCO 2017
method:
  - Deformable DETR
  - multi-scale deformable attention
task: object detection
created: 2026-05-06
updated: 2026-05-13
tags:
  - paper
  - object-detection
  - transformer
  - attention
related:
  - "[[MapTR 2208]]"
---

# Deformable DETR 2010

Related: [[DriveNeXt Camera Encoder 2407]], [[TransFuser 2205]]

## One-line summary

Deformable DETR makes DETR faster and better for small objects by replacing dense image attention with sparse, learned, multi-scale sampling points.

## Problem

DETR is simple and end-to-end, but it has two problems:

- It trains slowly. Original DETR needs about 500 COCO epochs.
- It is weak on small objects because high-resolution feature maps make normal attention expensive.

The reason is simple: normal attention over image features looks at too many spatial locations.

## Model structure

![[Attachments/deformable-detr-2010/figure1-model-structure.png]]

The model keeps the DETR idea:

1. A CNN backbone extracts image feature maps.
2. A Transformer encoder processes image features.
3. A Transformer decoder uses object queries.
4. Prediction heads output classes and boxes.
5. Hungarian matching trains the set of predictions.

The main change is the attention module:

- Encoder self-attention becomes multi-scale deformable self-attention.
- Decoder cross-attention becomes multi-scale deformable cross-attention.
- Decoder self-attention between object queries stays normal multi-head attention.

## Deformable attention structure

![[Attachments/deformable-detr-2010/figure2-deformable-attention.png]]

Deformable attention uses three things:

- A reference point: where the query starts looking.
- Sampling offsets: where to sample around the reference point.
- Attention weights: how much to use each sampled feature.

The simple idea is:

```text
sampling location = reference point + learned offset
output = weighted sum of sampled values
```

## Normal attention equation

For one query $q$, normal multi-head attention aggregates all keys:

$$
\mathrm{MultiHeadAttn}(z_q, x)
= \sum_{m=1}^{M} W_m \left[ \sum_{k \in \Omega_k} A_{mqk} \cdot W'_m x_k \right]
$$

where:

- $m$ is the attention head.
- $k$ indexes all key locations.
- $A_{mqk}$ is the attention weight between query $q$ and key $k$.
- $x_k$ is the value at location $k$.

In image tasks, $\Omega_k$ can be very large because it contains all feature pixels.

## Deformable attention equation

Single-scale deformable attention samples only $K$ points around one reference point $p_q$:

$$
\mathrm{DeformAttn}(z_q, p_q, x)
= \sum_{m=1}^{M} W_m
\left[
\sum_{k=1}^{K} A_{mqk} \cdot W'_m x(p_q + \Delta p_{mqk})
\right]
$$

where:

- $p_q$ is one reference point for query $q$.
- $\Delta p_{mqk}$ is a learned 2D offset.
- $p_q + \Delta p_{mqk}$ is the sampled location.
- $A_{mqk}$ is the attention weight for that sampled point.
- $x(\cdot)$ is sampled by bilinear interpolation.

So one query has one reference point, but many sampled points around it.

## Multi-scale deformable attention equation

Multi-scale deformable attention samples from several feature levels:

$$
\mathrm{MSDeformAttn}(z_q, \hat{p}_q, \{x^l\}_{l=1}^{L})
= \sum_{m=1}^{M} W_m
\left[
\sum_{l=1}^{L} \sum_{k=1}^{K}
A_{mlqk} \cdot W'_m x^l(\phi_l(\hat{p}_q) + \Delta p_{mlqk})
\right]
$$

where:

- $L$ is the number of feature levels.
- $\hat{p}_q$ is a normalized reference point in $[0,1]^2$.
- $\phi_l(\hat{p}_q)$ maps the reference point to feature level $l$.
- Each head samples $K$ points from each level.

Default settings in the paper:

```text
M = 8 heads
L = 4 feature levels
K = 4 sampling points per head per level
C = 256 channels
```

So each query samples:

```text
8 × 4 × 4 = 128 points
```

## Normal attention vs deformable attention

| Item | Normal attention | Deformable attention |
|---|---|---|
| Where it looks | All feature locations | A few learned sampling locations |
| Cost | High for large feature maps | Much lower |
| Sampling | Dense | Sparse |
| Reference point | Not used | Used as the starting point |
| Offset | Not used | Predicted from query |
| Best use | General token relation modeling | Efficient image feature attention |

Normal attention asks:

```text
Which of all locations should I attend to?
```

Deformable attention asks:

```text
Starting from this reference point, where should I sample?
```

## Reference points

Reference points are created before calling DeformAttn.

### Encoder

In the encoder, every feature location is one query.

Each query uses its own location as the reference point.

Example for a 3 × 3 feature map:

```text
(0.167, 0.167)  (0.500, 0.167)  (0.833, 0.167)
(0.167, 0.500)  (0.500, 0.500)  (0.833, 0.500)
(0.167, 0.833)  (0.500, 0.833)  (0.833, 0.833)
```

So:

```text
encoder reference point = each feature token's own normalized location
```

### Decoder

In the one-stage decoder, each object query predicts a reference point:

```python
reference_points = Linear(query_embed).sigmoid()
```

So:

```text
object query → linear layer → sigmoid → (x, y)
```

In the two-stage version, reference points come from top encoder proposals.

## Offsets

Offsets are predicted from the query feature.

Official code idea:

```python
sampling_offsets = Linear(query)
sampling_offsets = sampling_offsets.view(
    B, num_queries, n_heads, n_levels, n_points, 2
)
```

The last dimension is:

```text
(dx, dy)
```

So each query has different offsets for:

- each head,
- each feature level,
- each sampling point.

Then the sampling location is computed as:

```python
sampling_location = reference_point + offset / [W, H]
```

The offset is needed even in the encoder.

Why? Because the reference point only says where the query starts. The offset says where the query should actually look.

Without offsets:

```text
each feature token mostly reads itself
```

With offsets:

```text
each feature token can read useful nearby or cross-scale locations
```

Offset points from different reference points can overlap. That is fine. Normal attention also lets many queries attend to the same key.

## Heads and levels

The head is the same high-level idea as in multi-head attention.

Each head is one parallel attention branch.

But the inside is different:

```text
normal attention head: attends to all keys
DeformAttn head: samples a few points
```

`n_levels` means the number of feature-map scales.

Example:

```text
level 0: stride 8
level 1: stride 16
level 2: stride 32
level 3: stride 64
```

If `n_levels = 4`, each query can sample from all four scales.

Each head has different offsets.

Shape:

```text
[B, num_queries, n_heads, n_levels, n_points, 2]
```

## Simple pseudocode

```python
def ms_deform_attn(query, reference_points, feature_maps):
    # query: [B, num_queries, C]
    # reference_points: [B, num_queries, n_levels, 2]

    value = value_proj(feature_maps)

    offsets = Linear(query)
    offsets = offsets.view(B, num_queries, n_heads, n_levels, n_points, 2)

    weights = Linear(query)
    weights = weights.view(B, num_queries, n_heads, n_levels * n_points)
    weights = softmax(weights, dim=-1)
    weights = weights.view(B, num_queries, n_heads, n_levels, n_points)

    output = 0

    for head in heads:
        for level in feature_levels:
            H, W = feature_maps[level].shape[-2:]

            for point in sampling_points:
                location = reference_points[..., level, :] + offsets[..., head, level, point, :] / [W, H]
                sampled_value = bilinear_sample(value[level], location)
                output += weights[..., head, level, point] * sampled_value

    return output_proj(output)
```

## Results

On COCO val with ResNet-50:

| Method                     | Epochs |   AP | Small AP |
| -------------------------- | -----: | ---: | -------: |
| DETR                       |    500 | 42.0 |     20.5 |
| DETR-DC5, 50 epochs        |     50 | 35.3 |     15.2 |
| Deformable DETR            |     50 | 43.8 |     26.4 |
| + iterative box refinement |     50 | 45.4 |     26.8 |
| ++ two-stage               |     50 | 46.2 |     28.8 |

Main result:

```text
Deformable DETR trains about 10× faster than DETR and improves small-object detection.
```

## Takeaway

Deformable DETR keeps DETR's end-to-end detection design, but changes how image features are attended.

The core idea is simple:

```text
Do not attend to every pixel.
Start from a reference point.
Learn a few offsets.
Sample only useful locations across scales.
```

This makes attention cheaper, easier to train, and better for small objects.
