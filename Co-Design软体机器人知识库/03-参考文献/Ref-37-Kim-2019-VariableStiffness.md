# [37] Kim et al. 2019 · 连续变刚度同轴管机构

> **引用**: J. Kim, W.-Y. Choi, S. Kang, C. Kim, K.-J. Cho. *Continuously variable stiffness mechanism using nonuniform patterns on coaxial tubes for continuum microsurgical robot.* **IEEE TRO**, 35(6):1475-1487, 2019.

`#文献-方法` `#变刚度` `#微创手术`

---

## 🎯 这篇论文想做什么

微创手术内窥镜需要"先柔顺穿入，再刚硬定位"。Seoul Cho 教授团队问：

> 能否用 **同轴双管 + 非均匀图案** 实现 **机器内** 的连续刚度调节？无需充气、加热等外部辅助。

---

## 🛠️ 实现路径

### 1. 双管结构
两个不同图案的薄壁管 **同心嵌套**:
- 内管：刻有切口图案 A。
- 外管：刻有切口图案 B。

### 2. 相对旋转改变刚度

旋转外管相对内管：
- **图案对齐**：两组切口重合 → 整体最柔。
- **图案错开**：切口被覆盖 → 整体最硬。

刚度可在 **几个数量级** 范围内连续调节。

### 3. 数学模型

每段弯曲刚度由切口面积和位置决定：
$$EI_\text{eff}(\theta) = EI_0 \cdot f(\text{overlap}(\theta))$$
- $\theta$：相对旋转角。
- $f$：覆盖函数 (与切口几何有关)。

### 4. 微创手术原型
- 直径 ~5 mm。
- 弯曲半径 ~30 mm。
- 现场快速变刚度满足导航 vs 操作切换。

---

## 📐 关键概念

### 连续变刚度 (Continuous Variable Stiffness)
**含义**：刚度随旋转角连续可调，**无离散阶跃**。
**使用场景**：
- 手术内窥镜。
- 软外骨骼。
- **本论文 (Yi'25) 启示**：未来可在 co-design 中加入 **运行时变刚度** 维度——让算法决定何时变。

---

## 🔑 对本论文的意义

- 与颗粒阻塞 ([[Ref-34-Wei-2016-ParticleJamming]])、层阻塞 ([[Ref-38-Narang-2017-LaminarJamming]]) 并列为 **主动变刚度** 三大思路。
- 本论文目前是 **被动静态分布**；连续变刚度是自然扩展方向。

---

## 🧭 延伸阅读

- 颗粒阻塞：[[Ref-34-Wei-2016-ParticleJamming]]
- 层阻塞：[[Ref-38-Narang-2017-LaminarJamming]]
- 软夹爪综述：[[Ref-02-Shintake-2018-软体夹爪综述]]

---

*← [[_参考文献索引]]*
