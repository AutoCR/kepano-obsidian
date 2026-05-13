---
categories:
  - "[[Papers]]"
title: "SparseDrive: End-to-End Autonomous Driving via Sparse Scene Representation"
authors:
  - Wenchao Sun
  - Xuewu Lin
  - Yining Shi
  - Chuang Zhang
  - Haoran Wu
  - Sifa Zheng
affiliation:
  - "Tsinghua University"
  - "Horizon"
venue: ICRA
year: 2025
doi:
url: https://arxiv.org/abs/2405.19620
pdf: https://arxiv.org/pdf/2405.19620v2
field:
  - autonomous-driving
  - end-to-end-driving
keywords:
  - sparse-scene-representation
  - planning
  - motion-prediction
  - ego-status
  - nuscenes
status:
  - read
rating:
dataset:
  - nuScenes
method:
  - sparse scene representation
  - parallel motion planner
  - ego instance initialization
  - collision-aware rescore
task: end-to-end autonomous driving
created: 2026-04-20
updated: 2026-05-13
tags:
  - paper
related:
  - "[[Sparse4D 2211]]"
  - "[[SparseDriveV2 2603]]"
  - "[[MapTR 2208]]"
  - "[[Hydra-MDP 2406]]"
  - "[[transfuser]]"
  - "[[TransFuser 2205]]"
  - "[[FeaXDrive 2604]]"
  - "[[DriveTransformer 2503]]"
---

一句话：**SparseDrive 是一个 sparse-centric 端到端自动驾驶系统。它用 sparse instances 表示动态车辆、地图元素和 ego vehicle，并用 parallel motion planner 同时做 motion prediction 和 planning。**

# Core takeaways

- SparseDrive 不依赖 dense BEV feature 作为主要中间表示。
- 它把 detection、tracking、online mapping、motion prediction 和 planning 放进一个 sparse instance framework。
- Planning 不是直接回归一条轨迹。它先生成多模态候选轨迹，再选择一条安全轨迹。
- Ego vehicle 也被建模成一个 instance。这个设计很重要。
- 官方代码中，CAN-bus ego status 主要用于训练监督。它不是 inference 时的直接输入。

# Model structure

SparseDrive 有三个主要部分：

1. **Multi-view image encoder**
2. **Symmetric sparse perception**
3. **Parallel motion planner**

![[SparseDrive Figure 2 overview.png]]

Sparse perception 输出两类 sparse instances：

- **Agent instances**：动态物体，例如 car、truck、pedestrian。
- **Map instances**：静态地图元素，例如 divider、boundary、pedestrian crossing。

每个 agent instance 有：

```text
feature: [B, Nd, C]
anchor:  [B, Nd, 11]
```

其中 anchor 的 11 维是：

```text
[x, y, z, log(w), log(l), log(h), sin(yaw), cos(yaw), vx, vy, vz]
```

![[SparseDrive Figure 3 symmetric sparse perception.png]]

# Planning model

SparseDrive 的 planner 叫 **Parallel Motion Planner**。

它的输入是：

```text
agent instances
+ map instances
+ ego instance
```

它同时输出：

```text
surrounding-agent future trajectories
+ ego future planning trajectory
```

This is why it is called “parallel”. Motion prediction and planning are not separated into two sequential modules.

![[SparseDrive Figure 4 parallel motion planner.png]]

# Ego instance initialization

SparseDrive explicitly builds one ego instance:

```text
ego_feature: [B, 1, 256]
ego_anchor:  [B, 1, 11]
```

## Ego feature

The ego feature is initialized from the smallest front-camera feature map.

In the official code:

```python
feature_maps_inv = feature_maps_format(feature_maps, inverse=True)
feature_map = feature_maps_inv[0][-1][:, 0]
ego_feature = self.ego_feature_encoder(feature_map)
ego_feature = ego_feature.unsqueeze(1).squeeze(-1).squeeze(-1)
```

The first camera is `CAM_FRONT` in the nuScenes converter. So this uses the front camera.

Typical dimension change:

```text
front FPN feature: [B, 256, 8, 22]
Conv stride 1:     [B, 256, 8, 22]
Conv stride 2:     [B, 256, 4, 11]
AvgPool:           [B, 256, 1, 1]
Final ego token:   [B, 1, 256]
```

The paper says the ego car itself is in the camera blind area. So the ego feature is not a crop of the ego car. It is a scene-context feature from the front view.

## Ego anchor

The ego anchor is a fixed box prior:

```text
x = 0
y = 0.5
size = ego vehicle size
yaw ≈ π/2
velocity = 0 at the first frame
```

Shape:

```text
ego_anchor: [B, 1, 11]
```

If a previous predicted ego status exists, the code does:

```python
ego_anchor[..., VY] = prev_ego_status[..., 6]
```

This means:

```text
use previous predicted forward velocity to initialize current ego anchor velocity
```

It looks confusing because `prev_ego_status[..., 6]` is the first velocity component in the ego-status vector, but it is written into `VY` of the anchor. This is probably due to coordinate convention: the ego car points along the model’s +Y direction.

# Ego status and CAN bus

The dataset builds a 10-D ego status from nuScenes CAN bus:

```text
acceleration:    3 dims
rotation rate:   3 dims
velocity:        3 dims
steering angle:  1 dim
```

So:

```text
ego_status: [B, 10]
```

SparseDrive predicts this status with an auxiliary head:

```python
planning_status = self.plan_status_branch(ego_feature + ego_anchor_embed)
```

It is supervised by CAN-bus ego status during training:

```python
L1(predicted_status, data["ego_status"])
```

## Does it use real CAN-bus ego status during inference?

In the official code, **no**.

During inference:

1. At the first frame, there is no previous status.
2. The ego anchor uses the fixed zero-velocity prior.
3. The network predicts ego status from the ego feature.
4. The predicted status is cached.
5. At the next frame, the previous predicted velocity is used in the ego anchor.

So the loop is:

```text
frame t:
front feature + ego anchor → predicted ego status

frame t+1:
front feature + previous predicted ego status → new ego status
```

It does not directly read real CAN-bus ego status as an inference input.

# Is this good for closed-loop driving?

For open-loop benchmark evaluation, this design is understandable. Directly using real ego speed and steering can give a strong shortcut. It may make comparison unfair.

For closed-loop or real-world evaluation, real ego status is important.

The planner should know:

```text
current speed
acceleration
yaw rate
steering angle
actual ego pose
tracking error
```

The same camera image can require different plans at different speeds. A car at 2 m/s and a car at 15 m/s should not produce the same plan.

So for real deployment, I would use real CAN/IMU/odometry ego status as model input.

A simple design is:

```text
ego_status_embed = MLP(real_ego_status)
ego_feature = ego_feature + ego_status_embed
```

Better training should include ego-status dropout or noise, so the model does not overfit to status only.

Important rule:

```text
If real ego status is used during inference, it should also be used during training.
```

Using it only at inference creates train-test mismatch.

# Spatial-temporal interaction

After ego initialization, SparseDrive concatenates ego with agent instances:

```text
agent instances + ego instance
```

Then it uses three interaction blocks. Each block has:

1. **Agent-temporal cross-attention**  
   Each instance attends to its own history.

2. **Agent-agent self-attention**  
   Ego and other agents interact with each other.

3. **Agent-map cross-attention**  
   Ego and agents attend to sparse map instances.

4. **FFN and normalization**

The temporal memory queue length is 4 frames in the small config.

# Planning output

SparseDrive predicts multi-modal planning trajectories.

In the small config:

```text
3 driving commands: left, right, straight
6 planning modes per command
6 future steps per trajectory
2D point at each step
```

So the ego planning output is conceptually:

```text
[B, 3, 6, 6, 2]
```

The model also predicts scores for these candidates.

# Hierarchical planning selection

SparseDrive selects the final plan in three steps.

## 1. Command selection

It first selects trajectories for the current high-level command:

```text
left / right / straight
```

In nuScenes evaluation, this command comes from `gt_ego_fut_cmd`. In a real system, it should come from the route planner.

## 2. Collision-aware rescore

It checks ego candidate trajectories against predicted agent trajectories.

If a candidate has high collision risk, its score is reduced. In the code, collided candidates can be set to score 0.

## 3. Max-score selection

The final trajectory is the highest-score remaining candidate:

```text
final_planning: [B, 6, 2]
```

# Relation to SparseDriveV2 and Hydra-MDP

## SparseDrive

- Main focus: sparse scene representation.
- It is a full-stack system.
- It unifies perception, prediction, and planning.
- Planning candidates are relatively small.
- Safety comes partly from collision-aware rescore.

## Hydra-MDP

- Main focus: fixed trajectory vocabulary.
- It uses multi-target hydra distillation.
- It is more planner-centric than SparseDrive.

## SparseDriveV2

- Main focus: dense factorized trajectory vocabulary.
- It separates path anchors and velocity anchors.
- It makes scoring-based planning more scalable.

Simple summary:

```text
SparseDrive:   represent the scene sparsely, then plan.
Hydra-MDP:     score fixed trajectory anchors with teacher distillation.
SparseDriveV2: build a dense action space, then score it efficiently.
```

# My view

SparseDrive is a strong system paper. Its most useful idea is not only sparse perception. The important planning idea is that ego should also be treated as an instance.

The ego instance has:

```text
semantic context from the front camera
+ geometric ego anchor
+ temporal memory
+ auxiliary ego-status prediction
```

This makes planning more structured than direct trajectory regression.

But for closed-loop or real-world use, I would add real ego status as an explicit input and train with it. The official design is better understood as a fair open-loop benchmark design, not the best deployment design.

# Related notes

- [[SparseDriveV2 2603]]
- [[Hydra-MDP 2406]]
- [[transfuser]]
- [[TransFuser 2205]]
