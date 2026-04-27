# 可微分模拟 Differentiable Simulation

> **一句话理解**：物理引擎以 $y = \Phi(x; \xi)$ 形式实现 (其中 $x$ 输入、$\xi$ 参数)，且支持 **自动微分 (AD)** 计算 $\partial y / \partial x, \partial y / \partial \xi$。这样就能像训神经网络一样对系统参数做 **梯度下降优化**。

`#核心概念`

---

## 1. 形式化设定

### 1.1 离散时间动力学

$$\mathbf{s}_{t+1} = f(\mathbf{s}_t, \mathbf{u}_t; \boldsymbol{\theta}), \quad t = 0, 1, ..., T-1$$

**变量含义**：
- $\mathbf{s}_t \in \mathbb{R}^{n_s}$ — $t$ 时刻系统 **状态** (位置 + 速度 + 应变, 典型 $n_s \sim 10^3-10^6$)。
- $\mathbf{u}_t \in \mathbb{R}^{n_u}$ — **控制输入** (扭矩、气压、腱拉力)。
- $\boldsymbol{\theta} \in \mathbb{R}^{n_\theta}$ — 物理 / 设计 **参数** (材料模量、几何尺寸、密度)。
- $f$ — 物理引擎的 **一步转移函数**, 由 PDE 离散得到。
- $T$ — 总时间步数。

**公式解读**：把整套物理仿真分解为重复调用 "**一步函数**" $f$。每步根据当前状态 + 控制 + 参数算下一状态。

### 1.2 任务损失

$$L(\boldsymbol{\theta}, \mathbf{u}_{0:T}) = \sum_{t=0}^{T} \ell(\mathbf{s}_t, \mathbf{u}_t)$$

**变量含义**：
- $\ell$ — **每步损失**: 例如目标距离、能耗、违规惩罚。
- $L$ — 总累计损失。
- $\mathbf{u}_{0:T}$ — 整个动作序列 $(\mathbf{u}_0, ..., \mathbf{u}_{T-1})$。

**公式解读**：把"轨迹是否好"量化为损失函数。

### 1.3 解析梯度链式法则

$$\nabla_{\boldsymbol{\theta}} L = \sum_{t=0}^{T} \frac{\partial \ell}{\partial \mathbf{s}_t} \cdot \frac{\partial \mathbf{s}_t}{\partial \boldsymbol{\theta}}$$

**变量含义**：
- $\frac{\partial \ell}{\partial \mathbf{s}_t}$ — 损失对状态的偏导 (在每个时间步算)。
- $\frac{\partial \mathbf{s}_t}{\partial \boldsymbol{\theta}}$ — 状态对参数的偏导 (沿动力学链反推)。

**公式解读**：损失对参数的总梯度 = 各时间步贡献之和。每步贡献 = "**该步损失对状态敏感度**" × "**状态对参数敏感度**"。

进一步展开 $\frac{\partial \mathbf{s}_t}{\partial \boldsymbol{\theta}}$:

$$\frac{\partial \mathbf{s}_t}{\partial \boldsymbol{\theta}} = \sum_{k=0}^{t-1} \prod_{j=k+1}^{t-1} \frac{\partial f_j}{\partial \mathbf{s}_j} \cdot \frac{\partial f_k}{\partial \boldsymbol{\theta}}$$

**变量含义**：
- $\frac{\partial f_j}{\partial \mathbf{s}_j}$ — 每步动力学对当前状态的雅可比矩阵 (描述该步对小扰动的放大)。
- $\frac{\partial f_k}{\partial \boldsymbol{\theta}}$ — 第 $k$ 步动力学对参数的偏导。
- $\prod$ 项 — 从第 $k$ 步影响传播到第 $t$ 步, 中间所有雅可比矩阵的连乘。

**公式解读**：参数 $\theta$ 在 $k$ 步影响动力学 → 通过 $t-k-1$ 步雅可比传播到 $t$ 步状态。**所有这种贡献加和** 得到总敏感度。

**这就是为何长 rollout 梯度容易爆炸/消失**: 雅可比矩阵连乘, 谱半径 > 1 爆炸, < 1 消失 (类似 RNN)。

---

## 2. 实现路径：自动微分 (AD)

### 2.1 前向 / 反向模式

**前向模式 (Forward-mode AD)**：
$$\dot{y} = J \cdot \dot{x}$$

**变量含义**：
- $\dot{x}$ — 输入扰动方向 (用户给定)。
- $J = \partial y / \partial x$ — 雅可比矩阵。
- $\dot{y}$ — 输出对该方向的导数。

**单次前向计算所有 $\partial y / \partial x_i$**, 复杂度 $O(\dim x)$。适合 $\dim x \ll \dim y$。

**反向模式 (Reverse-mode / Backprop)**：
$$\bar{x} = J^\top \cdot \bar{y}$$

**变量含义**：
- $\bar{y} = \partial L / \partial y$ — 输出梯度 (一个标量损失对每个输出的偏导)。
- $\bar{x} = \partial L / \partial x$ — 输入梯度 (反传得到)。
- $J^\top$ — 雅可比矩阵转置。

**单次反向计算所有 $\partial L / \partial x_i$** ($L$ 标量)，复杂度 $O(\dim y)$。适合 $\dim x \gg \dim y$。

**机器人 co-design**: $\dim \boldsymbol{\theta} \gg 1, \; \dim L = 1$ → **必用反向模式**。

### 2.2 Tape 计算图

可微分引擎 (DiffTaichi, Warp, JAX) 通过 **tape (磁带)** 记录正向运算:
```
正向: y = sin(x); z = y * a;     → tape = [sin(x)→y, y*a→z]
反向: dz/da = y;  dz/dx = a·cos(x)
```

**Checkpointing 技术** — 长 rollout 显存爆炸时:
$$\text{显存}: O(T) \to O(\sqrt{T})$$
每 $\sqrt{T}$ 步存一次状态 (checkpoint), 反向时重算缺失部分。

---

## 3. 软体 + 接触的梯度坑

### 3.1 接触不连续

硬接触条件 (hard contact):

$$F_n(d) = \begin{cases} k_n |d|, & d < 0\ (\text{穿透}) \\ 0, & d \ge 0 \end{cases}$$

**变量含义**：
- $d$ — **间隙距离**: 物体表面间距离, 负值表示穿透。
- $k_n$ — 接触刚度 (惩罚系数, 通常很大如 $10^6$ N/m)。
- $F_n$ — 法向接触力。

**公式解读**：穿透时按胡克定律产生反力, 不穿透时无力。**问题**: $\partial F_n / \partial d$ 在 $d=0$ 处 **跳变** ($-k_n \to 0$)，梯度不连续。

```
F_n
 │
 │\
 │ \
 │  \
─┴───\──────→ d
      0
```

### 3.2 平滑接触解决方案 (Xu et al. 2021)

$$F_n^\text{smooth}(d) = k_n \cdot \log(1 + e^{-d/\epsilon})$$

**变量含义**：
- $\epsilon > 0$ — **平滑系数**: 越小越接近硬接触, 越大越平滑。
- 其他符号同上。

**导数**:
$$\frac{\partial F_n^\text{smooth}}{\partial d} = -\frac{k_n}{1 + e^{d/\epsilon}}$$

**公式解读**: $\log(1+e^{-d/\epsilon})$ 是 **softplus 函数**, 处处可微。$\epsilon$ 控制过渡区宽度。训练时 $\epsilon$ 大, 后期 anneal 到小值逼近硬接触。

### 3.3 大变形 + 自碰撞

变形梯度行列式 $J = \det(\mathbf{F})$ 表示局部体积比:
- $J = 1$: 无体积变化。
- $J > 1$: 膨胀。
- $J < 1$: 压缩。
- $J < 0$: **网格反转, 物理不可行**, 模拟发散。

**对策**: 加 **障碍项**:
$$U_\text{barrier} = -\log(\det \mathbf{F})$$

**变量含义**：
- $U_\text{barrier}$ — 添加到自由能的惩罚势, 当 $J \to 0^+$ 时 $\to +\infty$。

**公式解读**: 阻止网格塌缩 (类似内点法的 log 障碍)。

### 3.4 长 rollout 梯度爆炸 / 消失

雅可比连乘:
$$\prod_{t=k+1}^{T-1} \frac{\partial f_t}{\partial \mathbf{s}_t}$$

**变量含义**：每步雅可比谱半径 $\rho_t$。

**公式解读**: 总梯度规模 $\sim \prod \rho_t$。$\rho > 1$ 爆炸; $\rho < 1$ 消失。**对策**:
- 截断反向传播 (BPTT): 只反传 $K \ll T$ 步。
- 梯度裁剪: $\|g\|_2 \le c$。

---

## 4. 主流可微分物理引擎对照

| 工具 | 后端 | 物理 | 特色 |
|---|---|---|---|
| **DiffTaichi** | LLVM/CUDA | MPM, FEM, mass-spring | DSL 编译, 早期开源 |
| **NVIDIA Warp** | CUDA | FEM, PBD, XPBD | Python kernel + AD |
| **Brax** | JAX/XLA | Rigid body | 大规模 RL 友好 |
| **Tiny Differentiable Simulator** | C++ | Rigid + Articulated | MIT |
| **SOFA-Diff** | C++ | FEM 软体 | 实验性 |
| **PlasticineLab** | DiffTaichi 上 | 弹塑性 MPM | 软体操控基准 |

### Warp 示例

```python
import warp as wp

# 1. 定义可微 kernel
@wp.kernel
def step(x: wp.array(dtype=wp.vec3), 
         v: wp.array(dtype=wp.vec3),
         k: float, dt: float):
    tid = wp.tid()
    a = -k * x[tid]            # 弹簧力
    v[tid] = v[tid] + a * dt
    x[tid] = x[tid] + v[tid] * dt

# 2. 用 tape 记录梯度
tape = wp.Tape()
with tape:
    for t in range(T):
        wp.launch(step, dim=N, inputs=[x, v, k, dt])

# 3. 计算损失并反传
loss = compute_loss(x)
tape.backward(loss)

# 4. 取出梯度
print(k.grad)  # ∂loss/∂k
```

---

## 5. 端到端梯度优化的标准工作流

### 5.1 控制策略优化

```
初始化策略 π_φ (神经网络)
重复 epoch = 1..E:
    s_0 ← reset()
    for t = 0..T-1:
        u_t = π_φ(s_t)              # 策略给动作
        s_{t+1} = sim(s_t, u_t; θ)  # 可微模拟一步
    L = Σ_t ℓ(s_t, u_t)             # 累计损失
    ∇φ L = autograd.grad(L, φ)
    φ ← φ - η ∇φ L                  # 梯度下降
```

### 5.2 设计参数优化

```
初始化设计 θ_0
重复:
    跑模拟得轨迹 {s_t}
    L = task_loss({s_t})
    ∇θ L = autograd.grad(L, θ)
    θ ← θ - η ∇θ L
```

### 5.3 联合 (设计 + 控制)

```
重复:
    跑模拟
    L = task_loss
    ∇θ L, ∇φ L = autograd.grad(L, [θ, φ])
    θ ← θ - η_θ ∇θ L
    φ ← φ - η_φ ∇φ L
```

---

## 6. 与代理模型的关系 (本论文哲学)

| 路线 | 物理表达 | 梯度来源 | 数值稳定 | 速度 |
|---|---|---|---|---|
| **可微分模拟** | 真实 PDE 离散 | AD 通过物理 | 接触场景差 | 慢 |
| **神经代理** ⭐ 本论文 | NN 拟合 | AD 通过 NN | **平滑** | **快** |
| **混合 (residual)** | 可微 PDE + NN 残差 | AD 两路 | 中 | 中 |

本论文 (Yi'25) 实测对比 (Fig 4d):
- 直接 Warp 可微分梯度：$\sim 10$ s/iter，梯度抖动严重，常陷局部极小。
- 神经代理梯度：$\sim 10$ ms/iter (1000× 加速)，梯度平滑，多物体联合优化更稳定。

---

## 7. 何时选可微分模拟、何时选代理？

| 决策因子 | 选可微 | 选代理 |
|---|---|---|
| 物理是否光滑 | ✅ | (任何) |
| 接触/塑性 | ❌ | ✅ |
| 数据丰富 | (任何) | ✅ |
| 设计变化大 | ✅ | (重训) |
| 实时性要求 | 中 | 高 |
| 工程成熟 | 工业 (CFD) | 新兴 (机器人) |

---

## 8. 关联概念

- [[神经物理模拟器 Neural Physics]]：代理路线。
- [[代理模型 Surrogate Model]]：通用代理理论。
- [[FEM 有限元法]]：物理基础。
- [[../02-方法分类/梯度优化方法]]：方法学分类。

## 9. 关联文献

- [[../03-参考文献/Ref-18-Macklin-2022-Warp]]：NVIDIA Warp。
- [[../03-参考文献/Ref-19-Hu-2020-DiffTaichi]]：DiffTaichi。
- [[../03-参考文献/Ref-04-Xu-2021-可微设计框架]]：DiffHand + 平滑接触。
- [[../03-参考文献/Ref-21-Georgiev-PWM]]：可微 world model。
- [[../03-参考文献/Ref-41-Faure-2012-SOFA]]：SOFA 软体框架。

---

*← 返回 [[../00-总览/主索引 MOC]]*
