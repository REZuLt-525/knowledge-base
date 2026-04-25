# [12] Chen et al. 2018 · 拓扑优化软索驱动夹爪

> **引用**: F. Chen, W. Xu, H. Zhang, Y. Wang, J. Cao, M. Y. Wang, H. Ren, J. Zhu, Y. Zhang. *Topology optimized design, fabrication, and characterization of a soft cable-driven gripper.* **IEEE RAL**, 3(3):2463-2470, 2018.

`#文献-方法` `#拓扑优化` `#软体夹爪`

---

## 🎯 这篇论文想做什么

软体夹爪的几何形状经验上是"指状空腔" 或 "片状弯折"，**为什么是这个形状？** 没人能给数学最优答案。Chen 等想让 **算法** 来自动决定材料分布 —— 给定材料预算和性能目标，输出一张"哪里有材料、哪里是空" 的灰度图。具体目标：

- 输入：腱拉力大小、夹爪外接矩形包络、目标变形量。
- 输出：一张每个像素 (元素) 是否填材料的 0/1 分布图。
- 约束：使用材料体积不超过预算。

---

## 🛠️ 实现路径

### 步骤 1：把夹爪变形场离散化

把待设计区域 $\Omega$ 切分为 $N$ 个有限元 (例如平面四边形单元)，每个单元有 **密度变量** $\rho_e \in [0, 1]$。
- $\rho_e = 1$：实心。
- $\rho_e = 0$：空。
- 中间值代表"灰度"，最终经过惩罚收敛到 0/1。

### 步骤 2：SIMP 插值

按 **Solid Isotropic Material with Penalization (SIMP)**，单元有效弹性模量为：
$$E_e = E_{\min} + \rho_e^p \,(E_0 - E_{\min})$$
- $E_0$：实材料模量。
- $E_{\min}$：极小值 (避免奇异)。
- $p$：惩罚指数，通常 $p=3$。$\rho_e^p$ 让 0.5 灰度的"代价" 远大于真二值，从而驱使解收敛到 0/1。

### 步骤 3：求解力学位移

组装全局刚度矩阵：
$$\mathbf{K}(\boldsymbol{\rho}) = \sum_e \rho_e^p \,\mathbf{K}_e$$
求解线性系统 $\mathbf{K U} = \mathbf{F}$ 得节点位移 $\mathbf{U}$ (其中 $\mathbf{F}$ 是腱拉力等效节点力)。

### 步骤 4：定义目标函数

软夹爪要的是 **大柔度 / 大输出位移**。设输出点 (指尖) 位移为 $u_{\text{out}}$：
$$\max_{\boldsymbol{\rho}}\; u_{\text{out}}(\boldsymbol{\rho}) \quad \text{s.t.}\; \sum_e v_e \rho_e \le V^\star$$
也可以写成最小化 **柔度 (compliance)** $c = \mathbf{U}^\top \mathbf{K} \mathbf{U}$。

### 步骤 5：伴随法求灵敏度

由于 $\mathbf{K U} = \mathbf{F}$ 隐式定义 $\mathbf{U}(\boldsymbol{\rho})$，不能直接求 $\partial u_{\text{out}}/\partial\rho_e$。引入 **伴随变量 (adjoint)** $\boldsymbol{\lambda}$，满足：
$$\mathbf{K}\boldsymbol{\lambda} = \mathbf{e}_{\text{out}}$$
($\mathbf{e}_{\text{out}}$ 是输出点位置的单位向量)。则灵敏度为：
$$\boxed{\frac{\partial u_{\text{out}}}{\partial \rho_e} = -p\,\rho_e^{p-1}\,(E_0 - E_{\min})\,\mathbf{u}_e^\top \mathbf{k}_e^0\, \boldsymbol{\lambda}_e}$$
其中 $\mathbf{k}_e^0$ 是单元在 $E_0$ 下的刚度矩阵。

### 步骤 6：迭代更新

用 **OC (Optimality Criteria)** 或 **MMA (Method of Moving Asymptotes)** 更新 $\rho_e$，加 **密度过滤** 防止棋盘格。

### 步骤 7：制造与表征

把灰度图阈值化 → 转换成 STL → 多材料 3D 打印 (硬骨架 + 软体)。在万能试验机上测刚度、抓取测试 100+ 物体。

---

## 📐 关键公式与使用场景

| 公式 | 使用场景 |
|---|---|
| $E_e = E_{\min} + \rho_e^p (E_0 - E_{\min})$ | 把"密度"翻译成"刚度"，让连续优化可行 |
| $\mathbf{K U} = \mathbf{F}$ | FEM 求解夹爪在腱拉力下的形变 |
| $\mathbf{K}\boldsymbol{\lambda} = \mathbf{e}_{\text{out}}$ | 伴随法把"指尖位移"对全部 $\rho_e$ 的灵敏度 **一次** 算出 |
| OC 更新规则 $\rho_e \leftarrow \min(1, \max(0, \rho_e \cdot \eta))$ | 反复迭代逼近最优分布 |

**伴随法** 是工程优化中的核心技巧——如果不用伴随法，每个 $\rho_e$ 都要重解一次 $\mathbf{K U}=\mathbf{F}$，对于 $10^4$ 个单元意味着 $10^4$ 次重解；伴随法只解 **2 次** (一次正向、一次伴随) 就能算出全部梯度。

---

## 🔑 对本论文的意义

- **同一思想，不同自由度**：Chen'18 优化 **几何拓扑** (材料在/不在)；本论文优化 **刚度分布** (在固定几何下材料多硬)。可以理解为本论文是 **"结构级拓扑优化的子问题"**。
- **同一难点，不同应对**：Chen'18 直接用 SIMP + 伴随法；本论文用 **神经代理 + 反向传播** 来绕过 FEM + 接触的梯度噪声 (因为加上动态接触场景后，伴随法变得极其复杂)。

简言之：**Chen'18 是"静态、可导" 的拓扑优化经典；本论文是"动态接触、神经代理" 的现代变体。**

---

## 🧭 延伸阅读

- 软体设计综述：[[Ref-13-Chen-2020-软体设计综述]]
- 拓扑优化方法：[[../02-方法分类/拓扑优化]]
- 刚度分布：[[../01-核心概念/刚度分布 Stiffness Distribution]]
- 翼型 / DAFoam 同样使用伴随法：[[Ref-14-Buckley-2010-Airfoil]]、[[Ref-15-He-2019-DAFoam]]

---

*← [[_参考文献索引]]*
