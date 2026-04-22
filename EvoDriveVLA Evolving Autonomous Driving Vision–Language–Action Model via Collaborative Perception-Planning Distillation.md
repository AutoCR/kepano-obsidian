---
categories:
  - "[[Papers]]"
title: "EvoDriveVLA: Evolving Autonomous Driving Vision–Language–Action Model via Collaborative Perception-Planning Distillation"
authors:
  - Jiajun Cao
  - xiao peng
  - peking university
affiliation:
  - "State Key Laboratory of Multimedia Information Processing, School of Computer Science, Peking University"
  - "XPeng Motors"
venue:
year: 2026
doi:
url: https://arxiv.org/abs/2603.09465
pdf: https://arxiv.org/pdf/2603.09465
field: []
keywords:
  - e2e
  - VLA
  - Qwen 2.5 VL 3B
status:
  - read
rating:
dataset: []
method: []
task:
created: 2026-03-17
updated:
tags:
  - paper
---
这篇论文只是提出了一个训练方法，把Qwen2.5 VL 3B训练成一个planner，没有修改模型。最终的部署模型就是Qwen2.5 VL 3B. 寻来呢Qwen的所有参数，fully fine tune.
perception-planning distillation framework.
### self-anchored visual distillation
vlm的一大争议就是是否训练vision encoder。训练的话，可能丧失原本的能力，不训练，不能加强对特有任务能力。作者采用视觉蒸馏解决该问题。
复制一个visual encoder作为teacher, 训练的时候只更新student的参数，并且新增一个代价就是teacher和student的差异。同时，为了只针对自动驾驶任务，使用anchor distillation, 设计了AnchoredLayer和AnchoredScore， AnchoredLayer就是transformer decoder，用一组learnable queries和image, ego_state, future_trajectory做cross attn，然后把queries送个AnchoredScore(linear layer)打分，该分数考虑到teacher/student输出差异里面，最终目的是直实现visual encoder对自动驾驶场景能力的提升

### oracle-guided trajectory distillation
oracle teacher输入是当前和未来的自车状态和相机图片，oracle teacher生成粗轨迹，粗轨迹再由trajectory refiner生成最终的精细轨迹。同时使用粗轨迹和精细轨迹的hidden state和logits、以及真值对student进行监督。
oracle teacher和student用的是一个模型，Qwen 2.5 VL 3B，在训练时冻结。粗、细轨迹的生产其实就是让teacher做两次forward，第一次forward吃的是当前和未来的状态和照片，第二次forward吃的是第一次的输入和第一次的输出，并提示模型给他一个初始解。
teacher输出的是多条轨迹组成的集合，最后选一条距离GT最近的作为student的监督。
值得注意的是，轨迹集合并不是通过多次前向推理得到的。前向推理之得到一条轨迹，通过对这一次推理的hidden state做mc-dropout，再把dropout之后的hidden state送个lm_head计算logits得到不同的轨迹！
