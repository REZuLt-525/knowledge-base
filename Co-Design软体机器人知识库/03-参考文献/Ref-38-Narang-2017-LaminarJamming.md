# [38] Narang et al. 2017 · Laminar Jamming 层阻塞

> **引用**: Y. S. Narang, A. Degirmenci, J. J. Vlassak, R. D. Howe. *Transforming the dynamic response of robotic structures and systems through laminar jamming.* **IEEE RAL**, 3(2):688-695, 2017.

`#文献-方法` `#变刚度` `#层阻塞`

---

## 🎯 这篇论文想做什么

颗粒阻塞 ([[Ref-34-Wei-2016-ParticleJamming]]) 用粉末，但粉末重、易泄漏。Harvard Howe 团队提出 **层阻塞 (Laminar Jamming)**：用薄片代替粉末，更轻、响应快。

> 系统研究层阻塞的 **动力学响应** —— 不仅是静态刚度变化，还包括阻尼和振动行为。

---

## 🛠️ 实现路径

### 1. 层阻塞原理
多层 **薄片** (聚丙烯、纸、塑料) 叠放，外裹气密薄膜：
- **常压**：薄片之间能相对滑动 → 整体软、阻尼低。
- **抽真空**：膜紧压薄片 → 摩擦锁紧 → 整体硬、阻尼高。

### 2. 关键参数
- 薄片层数 $N$。
- 薄片摩擦系数 $\mu$。
- 真空度 $\Delta P$。
- 几何 (长方形 / 弯曲)。

### 3. 数学模型

锁紧后等效弯曲刚度：
$$EI_\text{jammed} \approx N^3 \cdot EI_\text{single}$$
- 类似 **复合梁** 公式：$N$ 层叠合时刚度按 $N^3$ 增长 (假设界面无滑动)。

未阻塞时各层独立：
$$EI_\text{unjammed} \approx N \cdot EI_\text{single}$$

刚度比 $\sim N^2$，故 **层数翻倍 → 刚度比 4×**。

### 4. 动态响应
论文还测了振动衰减：
$$\tau_\text{jammed} \ll \tau_\text{unjammed}$$
锁紧后阻尼比上升、共振峰下降。

### 5. 应用
- 触觉穿戴：用户按下时硬，回弹时软。
- 软外骨骼：训练辅助时硬，自由活动时软。

---

## 📐 关键公式与使用场景

### 公式 1：层数 - 刚度关系
$$\frac{EI_\text{jammed}}{EI_\text{unjammed}} \sim N^2$$

**使用场景**：设计多层结构时估算可调刚度比。

### 公式 2：动态阻尼变化
$$\zeta_\text{jammed} > \zeta_\text{unjammed}$$
**使用场景**：振动控制场景。

---

## 🔑 对本论文 (Yi'25) 的意义

- 与 [[Ref-34-Wei-2016-ParticleJamming|颗粒阻塞]]、[[Ref-37-Kim-2019-VariableStiffness|连续变刚度]] 并列为 **主动变刚度** 三大思路。
- 本论文 **不使用主动变刚度**——但层阻塞为未来"co-design 静态分布 + 运行时主动调节" 的双层 co-design 提供基础工具。

---

## 🧭 延伸阅读

- 颗粒阻塞：[[Ref-34-Wei-2016-ParticleJamming]]
- 连续变刚度：[[Ref-37-Kim-2019-VariableStiffness]]
- 软夹爪综述：[[Ref-02-Shintake-2018-软体夹爪综述]]

---

*← [[_参考文献索引]]*
