# [43] Glick et al. 2018 · 壁虎粘附软夹爪

> **引用**: P. Glick, S. A. Suresh, D. Ruffatto, M. Cutkosky, M. T. Tolley, A. Parness. *A soft robotic gripper with gecko-inspired adhesive.* **IEEE RAL**, 3(2):903-910, 2018.

`#文献-方法` `#软体夹爪` `#仿生粘附`

---

## 🎯 这篇论文想做什么

光滑表面 (玻璃、太阳能板、金属面板) 难以用机械夹钳抓取。Stanford Cutkosky + JPL Parness + UCSD Tolley 合作的目标：

> 把 **壁虎刚毛 (gecko adhesive)** 与 **气动软体** 结合，造出 **同时具备适形能力和粘附力** 的夹爪——专门应对光滑面物体。

---

## 🛠️ 实现路径

### 1. 壁虎粘附原理回顾
壁虎脚上有 **数十亿微米级刚毛 (setae)**。每根毛末端分叉成更细的纳米结构。
粘附力来源：**范德华力** (van der Waals)。
关键：**剪切方向加载** 时刚毛贴紧 → 高粘附；**法向方向卸载** 时刚毛释放 → 易脱离。

### 2. 仿生材料制作
- PDMS / 聚氨酯材料。
- 微刻刚毛阵列 (~50 μm)。
- 每平方厘米 ~$10^4$ 根。

### 3. 与气动软体集成

```
    ┌── 气动腔 (软体) ──┐
    │   充气 → 弯曲贴合│
    └─── 壁虎贴片 ────┘
```

抓取流程：
1. 软指接近物体，气压驱动贴合。
2. 切向滑动 → 壁虎刚毛激活 → 粘附。
3. 提起。
4. 释放：法向脱离 → 自动卸载。

### 4. 均匀压力模型应用 ⭐

为了让 **整片粘附面均匀贴合** 物体表面 (而非只贴一角)，作者采用 **均匀压力 (uniform pressure) 模型**——直接来自 [[Ref-42-Hirose-1978-UniformPressure|Hirose 1978]]。

> 📌 **这是连接 Hirose 1978 公式与本论文 (Yi'25) 的桥梁文献**。Hirose 是滑轮 + 链条的均匀压力；Glick'18 是 **气动软体** 的均匀压力；本论文 (Yi'25) 是 **腱驱动 + 柔性铰链** 的均匀压力。

---

## 📐 关键公式 (沿用 Hirose'78)

腱 / 弯曲压力分布：
$$h_i = H\left(1 - \frac{l_i}{L}\right)^2$$

**使用场景** (Glick'18 适配)：
- 决定气动腔的几何，使弯曲后整面均匀贴合物体。
- 防止"指尖卷起" 导致只有局部接触。

---

## 🔑 对本论文 (Yi'25) 的意义

论文 §3.2 第二句明确写道：

> "[the uniform pressure model] was later first used in a soft robot with pneumatic actuation [43]. We adopt this formulation on our tendon-driven soft finger."

也就是说，**Glick'18 是 Hirose 1978 在软体气动场景的首次应用**，本论文则把它推广到 **软体腱驱动场景**。这条公式由滑轮 → 气动 → 腱驱动的传承非常清晰。

---

## 🧭 延伸阅读

- 公式源头：[[Ref-42-Hirose-1978-UniformPressure]]
- 软体夹爪综述：[[Ref-02-Shintake-2018-软体夹爪综述]]
- 概念：[[../01-核心概念/腱驱动 Tendon-driven]]

---

*← [[_参考文献索引]]*
