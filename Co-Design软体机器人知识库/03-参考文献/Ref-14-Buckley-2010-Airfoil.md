# [14] Buckley, Zhou & Zingg 2010 · 翼型气动优化

> **引用**: H. Buckley, B. Zhou, D. Zingg. *Airfoil optimization using practical aerodynamic design requirements.* **Journal of Aircraft**, 47, 2010. doi:10.2514/1.C000256

`#文献-方法` `#拓扑优化` `#伴随法`

---

## 🎯 这篇论文想做什么

翼型 (airfoil) 设计长期靠经验。CFD 让数值优化成为可能，但要在 **多工况 + 实际工程约束** 下找最优外形仍很难。Buckley 等的目标：

> 把 **伴随法 (adjoint method) 求梯度** 与 **多工况约束** 结合，设计出在多种飞行条件下都鲁棒、且满足结构和制造约束的实用翼型。

---

## 🛠️ 实现路径

### 1. 翼型参数化
用 **B-spline 控制点** 描述翼面：
$$y(x) = \sum_{i=0}^n c_i B_i(x)$$
- $c_i$：控制点高度。
- $B_i$：基函数。
- 设计变量 = 控制点向量。

### 2. CFD 求解
对 Navier-Stokes 方程做有限体积法求解，得到：
- 升力 $C_L$
- 阻力 $C_D$
- 力矩 $C_M$

### 3. 伴随法求梯度
经典推导。设目标 $J = J(\mathbf{u}, \mathbf{x})$，约束 $\mathbf{R}(\mathbf{u}, \mathbf{x}) = 0$ (NS 残差)。
拉格朗日乘子法：
$$\mathcal{L} = J + \boldsymbol{\lambda}^\top \mathbf{R}$$
$$\frac{dJ}{d\mathbf{x}} = \frac{\partial J}{\partial\mathbf{x}} - \boldsymbol{\lambda}^\top \frac{\partial \mathbf{R}}{\partial\mathbf{x}}$$
其中 $\boldsymbol{\lambda}$ 解伴随方程：
$$\left(\frac{\partial\mathbf{R}}{\partial\mathbf{u}}\right)^\top \boldsymbol{\lambda} = \frac{\partial J}{\partial\mathbf{u}}$$

**好处**：梯度计算成本与设计变量数 **无关**，只多解一个伴随方程。

### 4. 多工况优化
$$\min_{\mathbf{x}} \sum_{k=1}^{K} w_k\,J_k(\mathbf{x})$$
不同 $J_k$ 代表不同 Mach 数 / 攻角。

### 5. 实用约束
- 厚度最小值 (结构强度)；
- 前缘半径 (制造)；
- 弯度连续性。

---

## 📐 关键公式与使用场景

### 公式 1：伴随梯度
$$\frac{dJ}{dc_i} = \frac{\partial J}{\partial c_i} - \boldsymbol{\lambda}^\top \frac{\partial \mathbf{R}}{\partial c_i}$$

**使用场景**：
- **任何高维设计变量 + 物理约束** 的优化问题。
- 翼型、汽车外形、燃气轮机叶片、潜艇壳。
- 软体机器人 ([[Ref-12-Chen-2018-拓扑优化软夹爪]]) 同样使用伴随法。

---

## 🔑 对本论文 (Yi'25) 的意义

- **梯度优化经典先例**：本论文综述把它列入"梯度优化方法" 类。
- **对比启示**：
  - 翼型: CFD 可微 + 伴随法 → 高效。
  - 软体接触: FEM 接触不可微 → 伴随法失效 → 需 NN 代理。

简言之：Buckley'10 展示伴随法在 **可导问题** 上的威力，本论文在 **不可导问题** 上用神经代理替代。

---

## 🧭 延伸阅读

- DAFoam 工具：[[Ref-15-He-2019-DAFoam]]
- 拓扑优化：[[Ref-12-Chen-2018-拓扑优化软夹爪]]
- 方法分类：[[../02-方法分类/梯度优化方法]]

---

*← [[_参考文献索引]]*
