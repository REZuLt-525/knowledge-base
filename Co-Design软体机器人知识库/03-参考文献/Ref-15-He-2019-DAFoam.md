# [15] He et al. 2019 · DAFoam: Open-source Adjoint Framework

> **引用**: P. He, C. Mader, J. Martins, K. Maki. *DAFoam: An open-source adjoint framework for multidisciplinary design optimization with OpenFOAM.* **AIAA Journal**, 2019. doi:10.2514/1.J058853

`#文献-工具` `#CFD` `#伴随法`

---

## 🎯 这个工具想做什么

OpenFOAM 是工业级开源 CFD，但本身 **不带伴随法**。Joaquim Martins (UMich) 团队的目标：

> 在 OpenFOAM 之上构建 **可微分 + 伴随法** 的工具集 DAFoam，让大型工程问题 (汽车、航空) 能用伴随法做大规模 (~10⁶ 设计变量) 优化。

---

## 🛠️ 实现路径

### 1. 伴随法在 OpenFOAM 中实现

DAFoam 把 OpenFOAM 的 PDE 残差和雅可比 **自动展开**，构建伴随方程：
$$\left(\frac{\partial \mathbf{R}}{\partial \mathbf{u}}\right)^\top \boldsymbol{\lambda} = \frac{\partial J}{\partial \mathbf{u}}$$

支持：
- 离散伴随 (DA, Discrete Adjoint)：先离散后求伴随，与 CFD 解保持一致。
- 多物理场：流体 + 结构 + 热 + 声学耦合。

### 2. 多学科 (Multidisciplinary)

把空气动力 + 结构 + 热载荷 **联合优化**：
$$\min_\mathbf{x} \; w_1\,J_\text{drag} + w_2\,J_\text{stress} + w_3\,J_\text{noise}$$
不同学科的伴随方程独立求解，但共享设计变量。

### 3. 与 OpenMDAO 集成

DAFoam 与 OpenMDAO (NASA 多学科优化工具) 集成，支持复杂工业流水线。

---

## 📐 关键技术与使用场景

### 技术 1：可微 OpenFOAM
**含义**：让开源 CFD 求得高效梯度。
**使用场景**：汽车外形、风机叶片、航空翼型。

### 技术 2：多学科耦合伴随
**使用场景**：现代飞机设计 (空气-结构-热-声学)；类比到机器人：**机械-控制-热-声** 多学科 co-design。

---

## 🔑 对本论文 (Yi'25) 的意义

- 与 [[Ref-14-Buckley-2010-Airfoil|Buckley'10]] 一道，作为"工程界可微设计已成熟" 的论据。
- 提醒：机器人 co-design 只是更广 **multidisciplinary design optimization (MDO)** 的子集，可借鉴航空 / 汽车几十年积累。

---

## 🧭 延伸阅读

- 翼型优化：[[Ref-14-Buckley-2010-Airfoil]]
- 拓扑优化：[[Ref-12-Chen-2018-拓扑优化软夹爪]]
- 方法分类：[[../02-方法分类/梯度优化方法]]

---

*← [[_参考文献索引]]*
