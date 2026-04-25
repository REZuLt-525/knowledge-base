# [33] Rothemund et al. 2018 · 软体双稳态气动阀

> **引用**: P. Rothemund, A. Ainla, L. Belding, D. J. Preston, S. Kurihara, Z. Suo, G. M. Whitesides. *A soft, bistable valve for autonomous control of soft actuators.* **Science Robotics**, 3(16):eaar7986, 2018.

`#文献-制造` `#气动控制`

---

## 🎯 这篇论文想做什么

软机器人通常依赖 **外部电磁阀 + 控制板**——这违背"全软" 理念。Whitesides 团队 (Harvard) 想要：

> 完全用 **软材料 + 几何结构** 实现 **双稳态阀 (bistable valve)**——无需电子，自动周期切换软体执行器的状态。

---

## 🛠️ 实现路径

### 1. 双稳态原理 (snap-through)
把硅胶板做成 **特殊曲面** (类似快门)，使其在两个稳定形态间切换：
- 状态 A：弯向左。
- 状态 B：弯向右。
- 中间 = 不稳定，气压超阈值即跳转。

数学上有 **负刚度** 区域：
$$\frac{dF}{dx} < 0$$ (在跳转区域)

### 2. 阀门集成
气流通过双稳态板：
- 状态 A：通道关闭 → 气压上升 → 推动 snap-through。
- 状态 B：通道打开 → 排气 → 气压下降 → 反向 snap-through。

### 3. 应用：自动振荡机器人
把多个阀串联，软体机器人 **自激振荡** 完成步态——无需电子控制器。

---

## 📐 关键概念

### 概念：Bistability + Snap-through
**含义**：两个稳定平衡 + 一个不稳定中间态。
$$E_\text{potential}(\theta)$$ 有两个极小、一个极大。

**使用场景**：
- 软机器人内嵌"逻辑门"。
- 行进机器人无需电控。
- **本论文 (Yi'25) 启示**：硬件可以编码控制逻辑——co-design 框架下未来可优化"双稳态结构 + 抓取动作"。

---

## 🔑 对本论文的意义

- "**物理智能**" 思想：硬件本身完成部分计算/控制。
- 本论文虽用电机驱动，但其腱走线均匀压力公式 ([[Ref-42-Hirose-1978-UniformPressure]]) 同样属于"硬件设计承担智能" 的传统。

---

## 🧭 延伸阅读

- 颗粒阻塞变刚度：[[Ref-34-Wei-2016-ParticleJamming]]
- 桌面制造：[[Ref-36-Zhai-2023-DesktopFabrication]]
- 软体机器人综述：[[Ref-01-Rus-2015-软体机器人综述]]

---

*← [[_参考文献索引]]*
