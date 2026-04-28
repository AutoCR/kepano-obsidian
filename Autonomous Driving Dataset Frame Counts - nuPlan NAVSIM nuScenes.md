---
categories:
  - "[[Evergreen]]"
title: Autonomous driving dataset frame counts - nuPlan, NAVSIM, nuScenes
created: 2026-04-28
updated: 2026-04-28
tags:
  - 0🌲
  - autonomous-driving
  - datasets
  - nuplan
  - navsim
  - nuscenes
sources:
  - https://github.com/motional/nuplan-devkit
  - https://github.com/autonomousvision/navsim
  - https://github.com/nutonomy/nuscenes-devkit
  - https://arxiv.org/pdf/2106.11810
  - https://arxiv.org/pdf/2406.15349
  - https://arxiv.org/pdf/2506.04218
  - https://arxiv.org/pdf/1903.11027
related:
  - "[[Hydra-MDP 2406]]"
  - "[[ZTRS 2510]]"
  - "[[CdDrive 2602]]"
  - "[[DiffusionDriveV2 2512]]"
  - "[[ComDrive 2410]]"
---

# Autonomous driving dataset frame counts - nuPlan, NAVSIM, nuScenes

## Executive summary

This note compares **nuPlan**, **NAVSIM/OpenScene**, and **nuScenes** with emphasis on **sample frequency** and **total frames**, since those are the most important planning-resource numbers.

| Dataset | Main sampling frequency | Clips / scenes | Duration per clip | Frames per clip | Total frames / samples |
|---|---:|---:|---:|---:|---:|
| **nuPlan** | **20 Hz** database timesteps / lidar_pc samples | **15,000+ logs**; scenes are snippets up to **20s** | scene ≤20s; logs are longer | up to **400 frames/scene** at 20Hz | **~93.6M timesteps** for v1.0 1,300h × 20Hz; paper planned 1,500h implies **108M** |
| **NAVSIM / OpenScene** | **2 Hz** data frames; evaluation interpolates/scores at **10Hz** | OpenScene: 120h from nuPlan. NAVSIM v1: navtrain **103,288** samples, navtest **12,146** samples. NAVSIM v2 navhard: **450** stage-1 scenes + **5,462** synthetic stage-2 scenes | v1 sample: history 1.5s + future 5s; v2 navhard: history 1.5s + future 4s | v1: **14 frames** = 4 history/current + 10 future; v2: **12 frames** = 4 + 8 | OpenScene base: **864,000 frames** = 120h × 3600 × 2Hz; navtrain sample-frame refs **1,446,032**, navtest **170,044**, navhard all **70,944** |
| **nuScenes** | Annotated keyframes **2Hz**; lidar **20Hz**; cameras **12Hz**; radar ~13Hz | **1,000 scenes** | **20s** each | **40 annotated frames/scene**; ~400 lidar sweeps/scene; ~1,440 camera images/scene across 6 cameras | **40,000 annotated frames**, ~**400k lidar sweeps**, ~**1.44M camera images**, ~**1.3M radar sweeps** |

## Source PDFs and links

### nuPlan
- Paper PDF: [nuPlan: A closed-loop ML-based planning benchmark for autonomous vehicles](https://arxiv.org/pdf/2106.11810)
- Local PDF: [[Attachments/2106 nuPlan closed-loop ML-based planning benchmark.pdf]]
- Official repo/docs: [motional/nuplan-devkit](https://github.com/motional/nuplan-devkit)
- Official site/download: https://www.nuscenes.org/nuplan

### NAVSIM / OpenScene
- NAVSIM v1 paper PDF: [NAVSIM: Data-Driven Non-Reactive Autonomous Vehicle Simulation and Benchmarking](https://arxiv.org/pdf/2406.15349)
- Local PDF: [[Attachments/2406 NAVSIM data-driven non-reactive autonomous vehicle simulation.pdf]]
- NAVSIM v1 supplementary PDF: [supplementary](https://mh0797.github.io/assets/pdf/navsim_supplementary.pdf)
- Local PDF: [[Attachments/2406 NAVSIM supplementary.pdf]]
- NAVSIM v2 paper PDF: [Pseudo-Simulation for Autonomous Driving](https://arxiv.org/pdf/2506.04218)
- Local PDF: [[Attachments/2506 NAVSIM v2 pseudo-simulation for autonomous driving.pdf]]
- NAVSIM v2 supplementary PDF: [supplementary](https://vveicao.github.io/projects/NavsimV2/Cao2025_supp.pdf)
- Local PDF: [[Attachments/2506 NAVSIM v2 supplementary.pdf]]
- Official repo/docs: [autonomousvision/navsim](https://github.com/autonomousvision/navsim)

### nuScenes
- Paper PDF: [nuScenes: A multimodal dataset for autonomous driving](https://arxiv.org/pdf/1903.11027)
- Local PDF: [[Attachments/1903 nuScenes multimodal dataset for autonomous driving.pdf]]
- Official repo/devkit: [nutonomy/nuscenes-devkit](https://github.com/nutonomy/nuscenes-devkit)
- Official site/download: https://www.nuscenes.org/nuscenes

## Dataset details

### nuPlan

- **Official release scale:** nuPlan devkit release note says v1.0 has **over 1,300 hours** of driving data, **15,000+ logs**, across **Las Vegas, Pittsburgh, Boston, Singapore**.
- **Paper scale:** the paper describes a planned release of **1,500 hours** and notes raw/sensor scale of **200+ TB**.
- **Sampling frequency:** the official FAQ says default `NuPlanScenario` creates a sample per timestep in the database at **20Hz**; subsampling by `0.1` reduces to **2Hz**.
- **Clip definition:** the schema says a `scene` is a snippet of **up to 20s** from a log.
- **Frames per scene:** up to **20s × 20Hz = 400 frames**.
- **Total frame estimate:**
  - v1.0 release: **1,300h × 3600 × 20Hz = ~93.6M frames/timesteps**.
  - paper planned scale: **1,500h × 3600 × 20Hz = 108M frames/timesteps**.
- **Caveat:** nuPlan uses “logs”, “scenes”, and “scenarios”; the docs do not publish one canonical total-frame count, so the most grounded estimate is official hours × official 20Hz.

### NAVSIM / OpenScene

- **Base data:** NAVSIM uses **OpenScene**, a compact redistribution of nuPlan.
- **OpenScene scale:** paper and docs say **120 hours** of annotated urban driving, downsampled to **2Hz**.
- **Total base frames:** **120h × 3600 × 2Hz = 864,000 frames**.
- **Sensors:** 8 cameras at **1920×1080** + merged LiDAR from 5 LiDAR sensors.
- **History input:** current frame + up to 3 past frames = **4 frames**, or **1.5s history at 2Hz**.
- **NAVSIM v1 filtered splits:**
  - `navtrain`: paper says **103k samples**; repo config has **103,288 tokens**.
  - `navtest`: repo config has **12,146 tokens**.
  - Each sample config: `num_history_frames=4`, `num_future_frames=10`, `frame_interval=1`.
  - Each sample has **14 frames** = 4 history/current + 10 future, i.e. 7s worth of 2Hz frame slots.
  - Sample-frame references: `navtrain` **103,288 × 14 = 1,446,032**; `navtest` **12,146 × 14 = 170,044**.
- **NAVSIM v2 navhard:**
  - Config has **450** original/stage-1 tokens and **5,462** reactive synthetic stage-2 scenes.
  - `num_history_frames=4`, `num_future_frames=8`; each scene has **12 frames**.
  - Stage-1 frame refs: **5,400**; all original + synthetic: **70,944**.
- **Evaluation frequency:** docs say PDM/EPDMS scoring evaluates a **4s horizon at 10Hz**. OpenScene source data remains **2Hz**, and agent/object states can be interpolated to 10Hz.

### nuScenes

- **Scale:** **1,000 scenes**, each **20s**.
- **Annotated keyframes:** **2Hz**, so **40 annotated frames per scene**.
- **Total annotated frames:** **1,000 × 20s × 2Hz = 40,000**.
- **Sensors:**
  - 6 cameras at **12Hz** → about **240 images per camera per scene**, **~1,440 images/scene**, **~1.44M images total**.
  - 1 LiDAR at **20Hz** → about **400 sweeps/scene**, **~400k sweeps total**.
  - 5 radars at about **13Hz**; the paper table reports **~1.3M radar sweeps/pointclouds**.
- **Total dataset time:** **1,000 × 20s = 20,000s = 5.56h**.
- **Caveat:** if the question is “labeled training frames,” use **40k annotated keyframes**. If it is “raw sensor frames,” use camera/lidar/radar totals separately.

## Related notes

- [[Hydra-MDP 2406]] — uses nuPlan-like / OpenScene / NAVSIM concepts.
- [[ZTRS 2510]] — NAVSIM/Navhard-related planning note.
- [[CdDrive 2602]] — NAVSIM v1/v2 benchmark note.
- [[DiffusionDriveV2 2512]] — NAVSIM v1/v2 benchmark note.
- [[ComDrive 2410]] — nuScenes planning benchmark note.
