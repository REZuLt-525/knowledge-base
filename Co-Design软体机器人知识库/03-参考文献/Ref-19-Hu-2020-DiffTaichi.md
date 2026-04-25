# [19] Hu et al. 2020 · DiffTaichi

> **引用**: Y. Hu, L. Anderson, T.-M. Li, Q. Sun, N. Carr, J. Ragan-Kelley, F. Durand. *DiffTaichi: Differentiable programming for physical simulation.* **ICLR 2020**.

`#文献-工具` `#可微分模拟` `#里程碑`

---

## 🎯 这篇论文想做什么

物理模拟用 GPU 写很快、用 Python 写好调试，但 **要让物理模拟"可微分"** (即输出对参数求导)，意味着要么手写反向传播 (痛苦)，要么用 PyTorch 写但巨慢。Hu 等的目标：

> 让程序员 **像写 NumPy 一样写物理模拟，编译器自动产生 GPU 代码 + 自动产生反向传播**。

应用场景：
- 软体机器人 co-design (优化几何/材料属性)；
- 光学逆向设计；
- 流体形状优化；
- 神经网络物理模拟器训练。

---

## 🛠️ 实现路径

### 核心 1：Taichi DSL

Taichi 是一种 Python 嵌入式 DSL，用 `@ti.kernel` 装饰函数，编译器会把它编译为 GPU/CPU 代码。

```python
@ti.kernel
def advance(dt: ti.f32):
    for i in range(N):
        x[i] += v[i] * dt
        v[i] += -k * x[i] * dt
```

### 核心 2：反向传播自动生成

DiffTaichi 把每条计算保存到 **磁带 (tape)**，反向时按依赖关系反推梯度。关键技术：

1. **Source-code transformation**：解析 Python AST，生成对应反向 kernel。
2. **Megakernel fusion**：把多步动力学融合成一个大 kernel，减少全局内存读写。
3. **Checkpointing**：长时间步序列下，保存稀疏快照而不是全状态，节省显存。

### 核心 3：物理算法库

DiffTaichi 自带：
- **MPM** (Material Point Method)：弹塑性、流体；
- **Mass-spring**：布料、软体；
- **FEM**：网格变形；
- **SPH**：流体粒子；
- **Rigid body** + 接触。

每种都已实现可微分版本。

---

## 📐 关键技术与使用场景

### 技术 1：可微分 MPM
$$\dot{\mathbf{F}} = \nabla\mathbf{v}\,\mathbf{F},\quad \boldsymbol{\sigma} = \frac{\partial \psi}{\partial \mathbf{F}}\mathbf{F}^\top$$
DiffTaichi 自动把变形梯度 $\mathbf{F}$ 的更新和应力 $\boldsymbol{\sigma}$ 的计算反向求导。

**使用场景**：
- 软体机器人形态优化 (Hu 等的"可微 MPM 软体")；
- 弹塑性变形逆向设计 (例如设计夹爪让物体弹回特定形状)。

### 技术 2：Checkpointing 反向

时间步数 $T$ 大时，存全部正向状态需要 $O(T)$ 显存。Checkpointing 只存 $\sqrt{T}$ 个快照，反向时重算缺失部分，把显存降到 $O(\sqrt{T})$。

**使用场景**：长水平 (long-horizon) 的 co-design 任务，例如优化几百时间步的轨迹。

---

## 🎯 实战示例 (论文给出)

1. **可微弹簧机器人**：从一根弹簧网络出发，优化弹簧刚度让机器人自己学会跑。
2. **可微流体光学**：优化液面让光聚到指定位置。
3. **可微软体抓取**：优化夹爪硬度参数让其抓住物体——**这正是本论文 Yi'25 的同类问题，但用 NN 代理替代 DiffTaichi 直接梯度**。

---

## 🔑 对本论文的关键意义

论文 §Related Work 把 DiffTaichi 列为可微分模拟的代表，并通过对比展示：

> "Differentiable simulations [18, 19] offer promising co-design directions ... but require smooth, tractable gradients, and are often sensitive to initialization [4, 20, 21]."

也就是说：**DiffTaichi 在原理上能解 co-design**，但实际有两大坑：
1. **梯度噪声**：接触不连续 → 梯度跳变。
2. **初值敏感**：随机初始化导致优化结果差异巨大 ([[Ref-04-Xu-2021-可微设计框架|Xu'21]] 直接验证)。

本论文 (Yi'25) **不直接用 DiffTaichi 梯度**，而是：
1. 用 Warp ([[Ref-18-Macklin-2022-Warp]]) 做正向模拟生成数据。
2. 用 NN 代理拟合，**代理本身光滑可微**，绕过原始物理梯度的坑。

---

## 🧭 延伸阅读

- NVIDIA Warp (本论文所用工具)：[[Ref-18-Macklin-2022-Warp]]
- 软体物理框架：[[Ref-41-Faure-2012-SOFA]]
- 可微分 co-design 痛点：[[Ref-04-Xu-2021-可微设计框架]]
- 概念详解：[[../01-核心概念/可微分模拟 Differentiable Simulation]]

---

*← [[_参考文献索引]]*
