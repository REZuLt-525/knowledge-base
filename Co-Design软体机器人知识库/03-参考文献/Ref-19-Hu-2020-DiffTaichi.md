# [19] Hu et al. 2020 · DiffTaichi

> **原文**: Y. Hu, L. Anderson, T.-M. Li, Q. Sun, N. Carr, J. Ragan-Kelley, F. Durand. *DiffTaichi: Differentiable programming for physical simulation.* **ICLR 2020**.
> **译名**: 《DiffTaichi: 面向物理仿真的可微分编程》 — 国际表征学习大会 (ICLR), 2020.

`#文献-工具` `#可微分模拟` `#里程碑`

---

## 1 · 研究问题

**Taichi** (胡渊鸣 MIT 博士) 是面向并行计算的 DSL (Domain Specific Language)。**DiffTaichi** 给它加上 **自动微分 (AD)**, 成为 **最早开源 + 完整文档** 的"**面向物理仿真的可微分编程框架**"。论文目标:

> 让程序员 **像写 NumPy 一样写物理模拟**, 编译器自动产生 GPU 代码 + 自动产生反向传播。让任何研究者都能在自己课题上做"可微分物理"。

---

## 2 · 编程模型

### 2.1 Taichi 字段 (field)

类似 NumPy 数组, 但运行在 GPU/CPU:
```python
import taichi as ti

x = ti.field(dtype=ti.f32, shape=N)            # 1D 数组
F = ti.Matrix.field(3, 3, dtype=ti.f32, shape=N)  # 3x3 矩阵字段
ti.root.dense(ti.i, N).place(x, F)             # 内存布局
```

### 2.2 Kernel + 自动并行

```python
@ti.kernel
def step(dt: ti.f32):
    for i in range(N):                         # 自动并行
        a = -k * x[i]
        v[i] += a * dt
        x[i] += v[i] * dt
```

`@ti.kernel` 函数被编译为 CUDA / x86 / Metal 代码。

### 2.3 自动微分

```python
@ti.ad.grad_replaced
def loss():
    return ti.sum(x ** 2)

loss.grad()
print(k.grad[None])      # ∂loss/∂k
```

---

## 3 · AD 实现技术

### 3.1 Source-to-Source 转换

DiffTaichi 解析 Python AST, 生成对应反向 kernel。例如对操作 $z = y \cdot a, y = \sin(x)$:

正向计算:
$$y = \sin(x);\;\; z = y \cdot a$$

反向计算 (对 $z$ 求 $\partial z / \partial \cdot$):
$$\frac{\partial z}{\partial a} = y;\;\; \frac{\partial z}{\partial x} = a \cdot \cos(x)$$

**变量含义**：
- $x, y, z, a$ — 中间变量。
- 对每个基本操作 (sin, mul) 都有对应反向规则。

**公式解读**：反向模式 AD 利用链式法则反向传播梯度, 与神经网络 backprop 同理。

### 3.2 Megakernel 融合

把多个连续操作融合为单个大 kernel, 减少全局内存读写:
```
原: kernel1; kernel2; kernel3
融合: combined_kernel { ... 1 ...; ... 2 ...; ... 3 ...; }
```

### 3.3 Checkpointing 反向

长时间步 ($T$ 大) 显存爆。DiffTaichi 实现:
- 每 $\sqrt{T}$ 步存快照 (checkpoint)。
- 反向时重算缺失。
- 显存复杂度从 $O(T)$ 降到 $O(\sqrt{T})$。

**变量含义**：
- $T$ — 时间步总数。
- 显存 $O(T)$ → $O(\sqrt{T})$ — 用计算量换显存。

---

## 4 · 物理算法库

DiffTaichi 内置:
- **MPM (Material Point Method)**: 弹塑性, 流体。
- **Mass-Spring**: 布料、绳子、软体。
- **FEM**: 网格变形。
- **SPH**: 流体粒子。
- **Rigid body** + 接触。

每种都已实现可微分版本, 用户可直接使用或自定义。

---

## 5 · 案例: 可微 MPM 软体优化

### 5.1 MPM 关键方程

变形梯度演化:
$$\dot{\mathbf{F}} = \nabla\mathbf{v}\,\mathbf{F}$$

**变量含义**：
- $\mathbf{F} \in \mathbb{R}^{3 \times 3}$ — 变形梯度 (描述局部如何变形)。
- $\mathbf{v}$ — 速度场。
- $\nabla\mathbf{v}$ — 速度梯度。
- $\dot{\mathbf{F}}$ — 变形梯度对时间导数。

**公式解读**：变形梯度随速度场演化, 体现连续介质物质变形。

应力计算 (从自由能 $\psi$):
$$\boldsymbol{\sigma} = \frac{\partial \psi}{\partial \mathbf{F}}\mathbf{F}^\top$$

**变量含义**：
- $\boldsymbol{\sigma}$ — Cauchy 应力。
- $\psi$ — 弹性自由能 (例如 Neo-Hookean)。

**公式解读**：应力是自由能对变形梯度的偏导。

DiffTaichi 自动把 $\mathbf{F}$ 的更新和 $\boldsymbol{\sigma}$ 的计算反向求导。

### 5.2 优化 demo: 弹性模量场反向

设杨氏模量场 $E(\mathbf{x})$ 离散为粒子上的 $E_p$。任务损失 = 末态质心位置:
$$L = \|\bar{\mathbf{x}}_T - \mathbf{x}^*\|^2$$

**变量含义**：
- $\bar{\mathbf{x}}_T$ — $T$ 时刻所有粒子质心位置。
- $\mathbf{x}^*$ — 目标位置。

通过 AD 反向: 求 $\partial L / \partial E_p$ → 优化 $E_p$ 让机器人到达目标。

---

## 6 · 论文实验亮点

### 6.1 可微弹簧机器人
从一根弹簧网络出发, 优化弹簧刚度 + 静息长度让机器人自己学会跑。
- 设计变量: ~100 个弹簧参数。
- 优化迭代: 100 步, 速度从 0.1 → 1.5 m/s。

### 6.2 可微流体光学
用流体表面波形 (透镜) 把光聚焦到指定位置:
$$\min_\text{波形参数} \|\text{光强中心} - \text{目标}\|^2$$

通过 AD 反向优化波形参数。

### 6.3 可微软体抓取
优化夹爪硬度参数让其能抓物体——**这正是本论文 Yi'25 的同类问题**, 但 Yi'25 用 NN 代理替代 DiffTaichi 直接梯度。

---

## 7 · 关键贡献

1. **Python DSL + 编译器**: 可微物理普及化。
2. **多物理算法**: MPM/FEM/SPH 都有可微版本。
3. **优化技术**: Megakernel 融合 + Checkpointing。
4. **开源 + 良好文档**: 学术界 / 工业界广泛采用。

---

## 8 · 局限

1. **接触场景梯度坏**: FEM + 接触 + 大变形会产生不稳定梯度 (本论文 Fig 4d 验证)。
2. **初始化敏感** ([[Ref-04-Xu-2021-可微设计框架|Xu'21]] 量化)。
3. **长时间步累积误差**: 随机/混沌系统反向梯度爆炸。
4. **Python 启动开销**: 小问题反而比 C++ 慢。

---

## 9 · 历史影响

- 推动 "**Python 可微物理**" 范式。
- 衍生 PlasticineLab (软体操控基准)、ChainQueen (弹塑性物理引擎)。
- 影响 NVIDIA Warp ([[Ref-18-Macklin-2022-Warp]]) 设计。
- 胡渊鸣后创办 Taichi Inc., 工业化此技术。

---

## 10 · 与本论文 (Yi'25) 的关系

本论文综述把 DiffTaichi 列为 **"可微分模拟"** 代表, 与 Warp 一道作为对照基线。本论文用 Warp (而非 DiffTaichi) 做正向数据生成, 但 **不直接用其 AD 梯度** — 改用 NN 代理因为:
- 接触梯度不稳 (Xu'21 验证)。
- 大规模并行数据生成 Warp 更优。

---

*← [[_参考文献索引]]*
