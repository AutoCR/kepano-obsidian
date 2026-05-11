---
categories:
  - "[[Papers]]"
title: "Sparse4D v3: Advancing End-to-End 3D Detection and Tracking"
authors:
  - Xuewu Lin
  - Zixiang Pei
  - Tianwei Lin
  - Lichao Huang
  - Zhizhong Su
affiliation:
  - "Horizon Robotics"
venue: arXiv
year: 2023
doi:
url: https://arxiv.org/abs/2311.11722
pdf: https://arxiv.org/pdf/2311.11722v1
field:
  - autonomous-driving
  - 3d-object-detection
  - 3d-multi-object-tracking
keywords:
  - sparse-3d-detection
  - multi-view-camera
  - temporal-instance-denoising
  - quality-estimation
  - decoupled-attention
  - end-to-end-tracking
  - nuscenes
status:
  - read
rating:
dataset:
  - nuScenes
method:
  - temporal instance denoising
  - quality estimation
  - decoupled attention
  - inference-time ID assignment
task: camera-only 3D object detection and multi-object tracking
created: 2026-05-09
updated: 2026-05-09
tags:
  - paper
related:
  - "[[Sparse4Dv2 2305]]"
  - "[[Sparse4D 2211]]"
  - "[[Deformable DETR 2010]]"
  - "[[SparseDrive 2405]]"
  - "[[SparseDriveV2 2603]]"
  - "[[VAD 2303]]"
---

One sentence: **Sparse4D v3 improves Sparse4D v2 for camera-only 3D detection, then turns it into a simple end-to-end tracker by carrying IDs on temporal instances.**

# Core takeaways

- Sparse4D v3 is built on [[Sparse4Dv2 2305]].
- It keeps the same sparse instance idea.
- It adds three main detection improvements:
  - temporal instance denoising
  - quality estimation
  - decoupled attention
- It also extends detection to tracking.
- The tracking design is simple.
- It does not add a separate association module.
- It does not need tracking labels or tracking-specific fine-tuning.

# What problem does it solve?

Sparse4D v2 already uses temporal instances.

A temporal instance is an object hypothesis from a previous frame.
It has a 3D box state and a feature vector.
The model projects it into the current frame and refines it with current images.

Sparse4D v3 asks:

> If temporal instances already move through time, can we also use them as tracking queries?

The paper's answer is yes.

# Main improvements

## 1. Temporal instance denoising

Denoising means the model receives noisy training examples and learns to recover the correct object.

Sparse4D v3 extends this idea to temporal instances.
It simulates imperfect temporal queries during training.
This matches the real inference problem.
At inference, the model must refine imperfect instances carried from previous frames.

The basic training idea is:

```text
clean GT box
-> add noise
-> noisy instance
-> model predicts a clean box again
```

For a temporal denoising instance, the noisy instance is propagated through time.
So from the next frame's view, it looks like a noisy historical instance.

Simple example:

```text
frame t GT center:      x = 10.0
add noise:             x = 10.5
model learns:          10.5 -> 10.0

propagate to frame t+1: x = 11.5
frame t+1 GT center:    x = 11.0
model learns:           11.5 -> 11.0
```

Important understanding from our discussion:

- It is not just adding noise independently to historical GT boxes.
- It is closer to: noisy instance at frame `t` -> propagate to frame `t+1` -> refine as a temporal instance.
- This better mimics inference, where imperfect temporal instances are carried forward.

The paper uses multiple noisy groups.
Each group gives a different noisy version of the same GT object.

Example:

```text
GT car center: x = 10.0, y = 4.0

group 1: x = 10.4, y = 3.8 -> recover GT
group 2: x =  9.6, y = 4.3 -> recover GT
group 3: x = 10.9, y = 4.5 -> recover GT
```

The groups are independent.
The model uses attention masks so noisy groups do not leak information to each other.

Implementation detail from the paper:

```text
M = 5 denoising groups
3 groups are randomly selected as temporal denoising groups
```

The paper also distinguishes noisy instances from normal learnable instances.

- Noisy instances use **pre-matching**.
- Normal learnable instances use **post-matching**.

Simple meaning:

```text
pre-matching:
noisy input anchors <-> GT
used to assign denoising targets

post-matching:
decoder predictions <-> GT
used for normal detection loss
```

This helps the denoising task know which GT each noisy anchor should recover.

## 2. Quality estimation

The paper argues that classification confidence is not enough.
A high classification score does not always mean the 3D box is accurate.

So Sparse4D v3 predicts two extra quality scores:

- centerness
- yawness

These scores help the model rank predictions by box quality, not only by class confidence.

### Centerness

Centerness measures how close the predicted box center is to the ground-truth box center.

```text
C = exp(-||[x, y, z]_pred - [x, y, z]_gt||_2)
```

If the predicted center is close, centerness is high.
If it is far away, centerness is low.

At inference, centerness can be multiplied with classification confidence.
This makes the result ranking better.
It especially helps reduce translation error, or mATE.

Our simple understanding:

```text
classification score: "Is this a car?"
centerness:           "Is this car box centered well?"
```

### Yawness

Yawness measures whether the predicted yaw direction matches the ground truth.

```text
Y = [sin(yaw), cos(yaw)]_pred · [sin(yaw), cos(yaw)]_gt
```

If the yaw directions align, yawness is high.
If the yaw directions disagree, yawness is low.

Our simple understanding:

```text
yawness: "Does this box face the right direction?"
```

Centerness mainly helps position quality.
Yawness helps orientation quality.

The paper trains these quality scores with:

```text
L = λ1 CE(Y_pred, Y) + λ2 Focal(C_pred, C)
```

Ablation understanding:

- Centerness reduces distance error.
- But centerness alone can hurt orientation error.
- Yawness helps compensate for that.
- Together, centerness and yawness improve detection ranking and also help tracking metrics.

## 3. Decoupled attention

Sparse4D uses both instance features and anchor embeddings.

Earlier versions mix them by addition.
Sparse4D v3 changes this to a more decoupled design.
It concatenates the instance feature and anchor embedding before attention.

Simple view:

```text
old: feature + anchor_embedding
new: concatenate(feature, anchor_embedding)
```

This gives attention more flexibility.
It helps the model avoid mixing semantic content and geometry too early.

# Extension to tracking

The tracking part is the most interesting idea.

Sparse4D v3 redefines an instance as a trajectory state.

```text
instance = (confidence, anchor_box, track_id)
```

Here:

- `confidence` says how likely this instance is a real object.
- `anchor_box` is the current 3D box state.
- `track_id` is the object identity.
- `track_id` can be empty at first.

The neural network predicts the confidence and box.
The ID is assigned by inference code.

## What is an instance?

An instance is one object hypothesis inside the detector.

It is like a small package that says:

> I may be an object. I have a feature, a 3D box, a confidence score, and maybe an ID.

A simplified instance contains:

```text
instance = {
  feature/query embedding,
  anchor/3D box,
  confidence score,
  optional track_id
}
```

The model has many instances.
Only some of them become real detections.
Most are background or redundant hypotheses.

## What is a temporal instance?

A temporal instance is an instance from the previous frame.
It is reused in the current frame.

Example:

```text
frame t-1: instance 17 -> car at old position
frame t:   instance 17 -> same car at new position
frame t+1: instance 17 -> same car again
```

Sparse4D carries the instance feature and anchor state forward.
Then the model updates them with current image features.

This recurrent design makes the instance behave like a tracking query.

## How does an instance carry an ID?

The ID is not learned by the network.
It is stored together with the temporal instance during inference.

Example at frame 1:

```text
instance 17 predicts a car
confidence = 0.86
track_id = empty
```

The confidence is above the threshold.
So the system assigns a new ID:

```text
instance 17 -> track_id = 5
```

The output is:

```text
car box, ID 5
```

Then instance 17 is kept in temporal memory:

```text
temporal instance 17 = {
  box: car box,
  confidence: 0.86,
  track_id: 5
}
```

At frame 2, the same temporal instance is passed back into the model.
The model updates its box.
The tracking code keeps the same ID:

```text
instance 17 predicts the car again
confidence = 0.82
track_id = 5
```

So the output is again:

```text
car box, ID 5
```

This is how the ID is carried through time.

# Tracking pipeline

The paper's tracking algorithm is simple.

1. Keep temporal instances from the previous frame.
2. Run the Sparse4D model on current sensor data.
3. If an instance has confidence above threshold `T`, output it.
4. If it has no ID, assign a new ID.
5. If it already has an ID, keep that ID.
6. Apply confidence decay to old temporal instances.
7. Keep the top `Nt` instances as memory for the next frame.

In the paper:

```text
T = 0.25
S = 0.6
Nt = 600
```

`S` is the confidence decay scale.
It lets an existing track survive short confidence drops.

Simplified pseudocode:

```python
for each frame:
    detections = model(sensor_data, temporal_instances, current_instances)

    results = []

    for instance in detections:
        if instance.confidence >= T:
            if instance.track_id is empty:
                instance.track_id = new_track_id()
            results.append(instance)

        if instance came from temporal memory:
            instance.confidence = max(
                instance.confidence,
                old_confidence * S,
            )

    temporal_instances = top_k(detections, k=Nt)
```

# Difference from tracking-by-detection

Traditional tracking-by-detection does this:

```text
detect boxes in each frame
-> match boxes across frames
-> use association, filtering, or Hungarian matching
-> output tracks
```

Sparse4D v3 does this:

```text
carry temporal instances through frames
-> attach IDs to confident instances
-> keep the IDs while the instances survive
-> output tracks
```

So the tracker is almost built into the detector.
There is no separate matching stage at inference.

# Results

On nuScenes validation with ResNet50 and image size `256 × 704`:

| Model | mAP | NDS | AMOTA |
|---|---:|---:|---:|
| Sparse4D v2 | 0.439 | 0.539 | 0.414 |
| Sparse4D v3 | 0.469 | 0.561 | 0.490 |
| Improvement | +0.030 | +0.022 | +0.076 |

On nuScenes test with VoVNet-99:

| Method | E2E | AMOTA | IDS | Recall | MOTA |
|---|---|---:|---:|---:|---:|
| DQTrack | yes | 0.523 | 1204 | 0.622 | 0.444 |
| DORT | no | 0.576 | 774 | 0.634 | 0.484 |
| Sparse4D v3 | yes | 0.574 | 669 | 0.669 | 0.521 |

Sparse4D v3 is strong in tracking, especially because it uses a very simple tracking extension.

# My understanding

Sparse4D v3 is not a fully new tracker architecture.
Its tracking ability comes from Sparse4D v2's temporal instance memory.

The model already carries object-like queries from frame to frame.
Sparse4D v3 adds a simple rule:

> If a temporal instance is confident, give it an ID. If it survives, keep the ID.

This makes tracking simple and elegant.

The main limitation is that the paper calls this tracking attempt preliminary.
There is still room to improve lifecycle management, ID stability, and tracking-specific training.

# Related reading

- [[Sparse4D 2211]] explains the original sparse 3D anchors and deformable 4D aggregation.
- [[Sparse4Dv2 2305]] explains recurrent temporal fusion and temporal instances.
- [[Deformable DETR 2010]] is useful background for sparse query-based detection.
