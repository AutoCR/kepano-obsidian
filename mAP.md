# mAP（mean Average Precision）

## 1️⃣ 本质理解

> **mAP = “检测对不对 + 框准不准”的综合指标**

用于评估目标检测模型（2D/3D 都有）

---

## 2️⃣ 计算流程（nuScenes / 3D版本）

和你熟悉的 COCO 不同，**nuScenes 用 center distance 而不是 IoU**

### Step 1：匹配预测与GT

对于每个预测框：

- 找最近的 GT
- 判断是否匹配成功（TP）

匹配条件（nuScenes）：

center distance < 阈值（如 2m / 4m）

---

### Step 2：排序（关键）

对所有预测按 **confidence score 排序**

---

### Step 3：构建 PR 曲线

逐个加入预测：

- TP → precision ↑
- FP → precision ↓

$$ Precision=TPTP+FP$$$$\text{Recall} = \frac{TP}{GT}$$​

---

### Step 4：计算 AP（每个类别）

$$ AP=\int_0^1 Precision(Recall)\ dRecall $$

实际：用离散点近似（插值）

---

### Step 5：计算 mAP

$$mAP=\frac{1}{C} \sum AP_c$$

## PR曲线
​![[output.png]]