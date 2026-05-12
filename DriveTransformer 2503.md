---
categories:
  - "[[Papers]]"
title: "DriveTransformer: Unified Transformer for Scalable End-to-End Autonomous Driving"
authors:
  - Xiaosong Jia
  - Junqi You
  - Zhiyuan Zhang
  - Junchi Yan
affiliation:
  - "Shanghai Jiao Tong University"
venue: ICLR
year: 2025
doi:
url: https://arxiv.org/abs/2503.07656
pdf: https://arxiv.org/pdf/2503.07656v2
field:
  - autonomous-driving
  - end-to-end-driving
keywords:
  - drive-transformer
  - task-parallelism
  - sparse-representation
  - streaming-processing
  - sensor-cross-attention
  - bench2drive
  - nuscenes
status:
  - read
rating:
dataset:
  - Bench2Drive
  - nuScenes
method:
  - unified transformer
  - task self-attention
  - sensor cross-attention
  - temporal cross-attention
  - sparse task queries
task: end-to-end autonomous driving
created: 2026-05-11
updated:
tags:
  - paper
related:
  - "[[SparseDrive 2405]]"
  - "[[SparseDriveV2 2603]]"
  - "[[Sparse4D 2211]]"
  - "[[Sparse4Dv2 2305]]"
  - "[[Sparse4Dv3 2311]]"
  - "[[Hydra-MDP 2406]]"
---

One-line takeaway: **DriveTransformer replaces the manually ordered perception → prediction → planning pipeline with a unified Transformer where agent, map, and ego queries interact in parallel, attend directly to multi-view sensor feature tokens, and reuse history through a streaming query memory.**

# Core takeaways

- The paper argues that many E2E-AD systems are hard to scale because they still keep a **sequential task dependency**: perception first, then prediction, then planning.
- DriveTransformer removes this hand-designed order. All task queries can exchange information through attention at every block.
- The model avoids constructing dense BEV features. Instead, sparse task queries directly retrieve information from camera feature-map tokens.
- Temporal fusion is also sparse: the model stores selected historical task queries in a FIFO memory queue instead of storing dense BEV history.
- The key nuance: **DriveTransformer is sparse in query/output space, but its Sensor Cross Attention is dense over sensor feature tokens in the paper formulation.** This is different from [[Sparse4D 2211]] / [[SparseDrive 2405]] style deformable aggregation, which samples a small set of projected image points per anchor.

# Paradigm comparison

Figure 1 positions DriveTransformer against three common E2E autonomous-driving paradigms: direct planning from BEV, sequential BEV pipelines, and parallel BEV pipelines. The authors' claim is that the pure Transformer paradigm keeps auxiliary tasks and task relations, while improving training stability and avoiding dense BEV cost.

![[Attachments/drivetransformer-2503/figure1-paradigm-comparison.png]]

My reading: this figure is mainly a **design-positioning figure**. It says the paper is not just another planner head; it is trying to change the system-level interface between perception, mapping, prediction, and planning.

# Model structure

## Tokenization

DriveTransformer converts inputs into several token types:

- **Sensor tokens** from multi-view camera backbone feature maps.
- **Agent queries** for dynamic objects and motion prediction.
- **Map queries** for static map elements and online mapping.
- **Ego query** for planning.
- **Temporal tokens** from historical task queries stored in memory.

Each token has semantic features plus positional encoding. Sensor tokens use camera geometry to build 3D positional encoding for image patches.

## Overall architecture

Figure 2 is the most important architecture diagram. Each layer contains three attention operations:

1. **Sensor Cross Attention**: task queries attend to current sensor feature tokens.
2. **Temporal Cross Attention**: task queries attend to historical task tokens from the memory queue.
3. **Task Self-Attention**: ego, agent, and map queries interact with each other.

![[Attachments/drivetransformer-2503/figure2-overall-framework.png]]

The architecture can be summarized as:

```text
multi-view image features + task queries + historical query memory
        ↓
Sensor Cross Attention
        ↓
Temporal Cross Attention
        ↓
Task Self-Attention
        ↓
detection / prediction / online mapping / planning heads
```

# Attention details

## Sensor Cross Attention

The paper writes Sensor Cross Attention as:

$$
H'_{ego}, H'_{agent}, H'_{map} = \operatorname{SCA}(Q=[H_{ego}+PE_{ego}, H_{agent}+PE_{agent}, H_{map}+PE_{map}], K=H_{sensor}+PE_{sensor}, V=H_{sensor})
$$

So all ego / agent / map queries use the sensor feature tokens as keys and values.

Important interpretation:

- This is **vanilla cross-attention** in the paper's formulation.
- Each task query can attend to all sensor feature patches from all cameras.
- It is therefore denser than DeformableAttention / DeformableAggregation in Sparse4D-style systems.
- The paper's word “sparse” mainly means **sparse task tokens instead of dense BEV grids**, not sparse image-point sampling.

## Task Self-Attention

Task Self-Attention lets all task queries interact directly:

- agent ↔ agent
- agent ↔ map
- ego ↔ agent
- ego ↔ map
- map ↔ map

This is the mechanism for planning-aware perception and interaction-aware planning.

## Temporal Cross Attention

Historical task queries are stored in a FIFO queue. Current task queries attend to these history tokens after motion compensation / ego transformation.

This is cheaper than storing dense BEV history and also keeps object/map/ego information in query form.

# Task heads

- **Object detection and motion prediction** share agent features.
- Motion prediction is expressed in each agent's local coordinate frame to decouple prediction loss from detection errors.
- **Online mapping** replicates each map query into point-level queries during Sensor Cross Attention, so points in the same polyline can retrieve different local image features.
- **Planning** uses the ego query with multiple mode embeddings for multi-mode trajectory prediction.

## Planning head

The planning head itself is relatively simple. The important part is that its input, the **ego query**, has already interacted with sensor tokens, historical task queries, agent queries, and map queries inside the unified Transformer.

The paper models ego planning as a **multi-modal trajectory prediction** problem to avoid mode averaging. It divides ego trajectories into six behavior modes:

1. Go straight
2. Stop
3. Left turn
4. Sharp left turn
5. Right turn
6. Sharp right turn

For each mode, the model builds a mode embedding and adds it to the ego feature:

```text
ego feature + go-straight embedding      → trajectory 1
ego feature + stop embedding             → trajectory 2
ego feature + left-turn embedding        → trajectory 3
ego feature + sharp-left-turn embedding  → trajectory 4
ego feature + right-turn embedding       → trajectory 5
ego feature + sharp-right-turn embedding → trajectory 6
```

So the planning head outputs:

```text
6 candidate ego trajectories + 6 mode confidence scores
```

During training, it uses a **winner-take-all** style loss: only the ground-truth mode's trajectory is used for regression loss, and a classification head learns to predict the current mode. During inference, the trajectory with the highest mode confidence is selected for evaluation or execution.

My reading: the planning head is **not the main novelty** of DriveTransformer. It is a small 6-mode regression/classification head on top of the ego query. This is much simpler than [[Hydra-MDP 2406]] or [[SparseDriveV2 2603]], which formulate planning as large-scale trajectory scoring over a much larger candidate set.

A compact comparison:

| Method | Planning style |
|---|---|
| DriveTransformer | 6-mode trajectory regression + mode classification |
| [[Hydra-MDP 2406]] | fixed trajectory vocabulary scoring with multi-target distillation |
| [[SparseDriveV2 2603]] | factorized path × velocity vocabulary scoring |
| [[SparseDrive 2405]] | sparse scene representation + parallel motion planner |

# Results

## Bench2Drive closed-loop

DriveTransformer-Large reports strong closed-loop performance:

| Method | Driving Score ↑ | Success Rate ↑ | Latency |
|---|---:|---:|---:|
| UniAD-Base | 45.81 | 16.36% | 663.4 ms |
| VAD | 42.35 | 15.00% | 278.3 ms |
| DriveAdapter* | 64.22 | 33.08% | 931 ms |
| **DriveTransformer-Large** | **63.46** | **35.01%** | **211.7 ms** |

The paper's key empirical point is that it gets high closed-loop performance while being much faster than heavy sequential systems like UniAD / DriveAdapter.

## nuScenes open-loop planning

With ego-status input, DriveTransformer-Large reaches:

- Avg. L2: **0.33 m**
- Avg. collision: **0.07%**

This is better than ParaDrive in the reported table and competitive with concurrent sparse methods.

## Scaling

The model scales cleanly with Transformer size:

| Size | Layers / hidden | Parameters | Latency | Driving Score on Dev10 |
|---|---:|---:|---:|---:|
| Small | 3 / 256 | 47.41M | 93.8 ms | 45.04 |
| Base | 6 / 512 | 178.05M | 139.6 ms | 60.45 |
| Large | 12 / 768 | 646.33M | 221.6 ms | 68.22 |

The authors argue this supports the “unified Transformer is easy to scale” thesis.

# Relation to Sparse4D and SparseDrive

## Compared with Sparse4D-style DeformableAggregation

Sparse4D-style systems usually keep sparse 3D anchors and sample a small number of projected image features around anchor keypoints. Complexity is closer to:

$$
O(N_{anchor} \cdot N_{points} \cdot N_{cams} \cdot N_{levels})
$$

DriveTransformer Sensor Cross Attention is closer to:

$$
O(N_{query} \cdot N_cHW)
$$

So DriveTransformer has simpler global access, but its sensor attention is denser over image features.

## Compared with SparseDrive

[[SparseDrive 2405]] is sparse in a more geometry-driven sense: it builds sparse scene representation and uses deformable feature aggregation. DriveTransformer is sparse mainly because it removes dense BEV and keeps task information as queries.

A compact distinction:

- **SparseDrive**: sparse scene representation + deformable image sampling + parallel planner.
- **DriveTransformer**: unified query Transformer + dense sensor cross-attention + streaming query memory.

# Caveats

- The architecture still entangles all subtasks inside one model, which makes debugging and maintenance harder.
- Sensor Cross Attention may become expensive if high-resolution multi-camera feature maps are used directly.
- The paper reports strong efficiency relative to BEV-heavy E2E systems, but it is not as image-sampling-sparse as Sparse4D / SparseDrive-style deformable aggregation.

# My understanding

DriveTransformer is best understood as a **system-level simplification paper**. Its main contribution is not a new planner loss or a new trajectory representation, but a cleaner way to organize an E2E autonomous-driving stack:

> keep everything as tokens, let all tasks interact in parallel, retrieve sensor/history information through attention, and avoid dense BEV state.

The most important design tradeoff is:

- It removes BEV complexity and sequential task dependency.
- But it pays for this with dense cross-attention over sensor feature tokens and a more coupled all-in-one model.

# Related reading

- [[SparseDrive 2405]]
- [[SparseDriveV2 2603]]
- [[Sparse4D 2211]]
- [[Sparse4Dv2 2305]]
- [[Sparse4Dv3 2311]]
- [[Hydra-MDP 2406]]
