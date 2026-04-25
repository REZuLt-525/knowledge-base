# [50] Craig Jr. & Taleff 2020 · 材料力学教材

> **引用**: R. R. Craig Jr. and E. M. Taleff. *Mechanics of Materials.* John Wiley & Sons, 2020.

`#文献-教材` `#材料力学`

---

## 🎯 这本教材想做什么

材料力学 (Mechanics of Materials) 是工科本科必修课——研究 **杆、梁、轴** 在各种载荷下的应力、应变、变形。Craig & Taleff 的版本是工程院校广泛采用的标准教材。

---

## 📖 核心知识 (与本论文相关部分)

### 1. 应力 - 应变关系
线弹性 (Hooke 定律)：
$$\sigma = E \cdot \varepsilon$$
- $\sigma$ (Pa)：应力。
- $\varepsilon$：应变。
- $E$ (Pa)：杨氏模量。

### 2. 梁弯曲基本方程
对均匀梁:
$$\frac{d^2 v}{dx^2} = \frac{M(x)}{EI}$$
- $v$：挠度。
- $M$：弯矩。
- $I$：截面惯性矩。

### 3. 三点弯曲 (本论文用于杨氏模量测量)
对支跨为 $L$ 的梁，中间施加力 $P$:
$$\delta = \frac{PL^3}{48 EI}$$
反求：
$$\boxed{E = \frac{PL^3}{48 I \delta}}$$

### 4. 压痕测试 (Indentation, 本论文 segment 块测量)
对柱状压头压入弹性体:
$$E = \frac{(1-\nu^2)P}{2 a \delta}$$
- $\nu$：泊松比。
- $a$：压头半径。
- $\delta$：下压深度。

### 5. 缩放律
若几何完全相似，**变形量与 $E$ 反比**：
$$\delta \propto \frac{1}{E}$$
故"相对刚度" 决定行为，而非绝对刚度。

---

## 📐 公式应用场景：本论文 §6.3 ⭐

本论文 Appendix §6.3 写道：

> "Since all of the flexure blocks and segment blocks have the same geometry, respectively, the deformation behavior mainly depends on relative stiffness [50]."

也就是说：**缩放律** 让本论文得以做以下事情：
1. 仿真用 $\log E \in [13.5, 17.0]$ (相对值)。
2. 真实打印参数 (wall loops, infill) 通过三点弯曲 / 压痕测试得 **真实 $E$**。
3. 把仿真 $\log E$ 线性映射到真实 $E$ 范围。

整个 Sim-to-Real 转换之所以能做"线性映射"，正是因为 **变形对相对刚度而非绝对值敏感** —— Craig & Taleff 教材给出这条理论基础。

---

## 🔑 对本论文的意义

- **小但关键**：在 50 篇参考文献中此教材篇幅最短，但提供了 **Sim-to-Real 标定的理论合法性**。
- **学习要点**：本论文作者 (UCSD 研究生) 没有简单地"假设" 线性映射，而是引用经典材料力学教材给出依据——这是好科研的范例。

---

## 🧭 延伸阅读

- 柔性机构 (用到梁理论)：[[Ref-39-Howell-2013-CompliantMechanisms]]
- FEM 概念：[[../01-核心概念/FEM 有限元法]]
- Sim-to-Real：[[../01-核心概念/Sim-to-Real]]
- 刚度分布：[[../01-核心概念/刚度分布 Stiffness Distribution]]

---

*← [[_参考文献索引]]*
