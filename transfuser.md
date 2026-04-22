---
categories:
  - "[[Papers]]"
title: "TransFuser: Imitation with Transformer-Based Sensor Fusion for Autonomous Driving"
authors:
  - Kashyap Chitta
affiliation: []
venue:
year:
doi:
url: https://arxiv.org/abs/2205.15997
pdf: "[[2205 TransFuser Imitation with transformer based sensor fusion for autonomous driving.pdf]]"
field: []
keywords:
  - tubingen
  - transformer
  - lidar
  - cam
status:
  - read
rating:
dataset: []
method: []
task:
created: 2026-03-13
updated:
tags:
  - paper
---
![[Pasted image 20260313171714.png]]
分别使用两个RegNetY-3.2GF作为backbone编码Lidar和cam特征，这两个backbone参数不一样，cam的regnet用imgnet数据预训练作为初值，lidar没有初值，然后在端到端训练。

对lidar和cam的Regnet4层不同分辨的特征，用transformer进行融合。每层特征融合完后再跟当前特征(cam/lidar)相加输入给下一层regnet。

将融合后的特征输入给GRU输出未来轨迹。

雷达特征处理：建立一个固定的bev网格（256x256），每个网格分成两个高度的bin（以摸个高度分割），bin1地面/低高度，bin2高于地面的物体。然后每个bin数一下对应的点数作为该层网络格子的值。