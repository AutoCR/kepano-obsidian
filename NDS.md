# NDS（NuScenes Detection Score）

## 1️⃣ 本质理解

> **NDS = 检测 + 几何 + 运动质量**

用于衡量自动驾驶“是否能用”

---

## 2️⃣ 公式

$$\text{NDS} = \frac{1}{10} \left( 5 \cdot mAP + \sum_{i=1}^{5} (1 - error_i) \right)$$

---

## 3️⃣ 五个 TP 指标（重点）

### ① ATE（位置误差）

$$ATE = \|p_{pred} - p_{gt}\|$$

---

### ② ASE（尺寸误差）

基于 scale IoU：

$$ASE = 1 - IoU_{scale}$$​

---

### ③ AOE（方向误差）

$$AOE = |\theta_{pred} - \theta_{gt}|$$

---

### ④ AVE（速度误差）

$$ AVE = \|v_{pred} - v_{gt}\| $$

---

### ⑤ AAE（属性误差）

如：

- parked / moving
- standing / sitting

---

## 4️⃣ normalization（关键）

每个 error 会映射到 [0,1]：

score = max(0, 1 - error / threshold)

👉 防止尺度不同影响