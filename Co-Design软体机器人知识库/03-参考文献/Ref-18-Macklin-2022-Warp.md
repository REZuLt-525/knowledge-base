# [18] Macklin 2022 · NVIDIA Warp

> **原文**: M. Macklin. *Warp: A high-performance Python framework for GPU simulation and graphics.* https://github.com/nvidia/warp, March 2022. NVIDIA GTC.
> **译名**: 《Warp: 用于 GPU 仿真与图形的高性能 Python 框架》 — NVIDIA GTC, 2022.

`#文献-工具` `#可微分模拟` `#GPU`

---

## 1 · 工具定位

NVIDIA Matthias Macklin (PhysX, Flex, IsaacSim 主力开发者) 推出 **Warp**, 目标:

> 让研究者用 **Python 语法** 编写自定义物理算法 (FEM, MPM, PBD, XPBD), 编译器自动生成高性能 GPU 代码, **支持自动微分 (AD)**。

定位介于 PyTorch (深度学习) 与 PhysX (引擎) 之间——**高性能 + 灵活 + 可微**。

---

## 2 · 编程模型

### 2.1 Kernel 装饰器

```python
import warp as wp

@wp.kernel
def integrate(x: wp.array(dtype=wp.vec3),
              v: wp.array(dtype=wp.vec3),
              k: float,
              dt: float):
    tid = wp.tid()
    a = -k * x[tid]              # 弹簧力
    v[tid] = v[tid] + a * dt
    x[tid] = x[tid] + v[tid] * dt
```

- `@wp.kernel` 让 Warp 在 import 时把函数 **编译为 CUDA 代码**。
- `wp.tid()` 返回当前线程 ID (类似 CUDA `threadIdx`)。
- `wp.array` 是 GPU 内存数组。
- 类型注解给编译器足够信息。

### 2.2 启动 kernel

```python
N = 10000
x = wp.zeros(N, dtype=wp.vec3, device='cuda')
v = wp.zeros(N, dtype=wp.vec3, device='cuda')
wp.launch(kernel=integrate, dim=N, inputs=[x, v, 100.0, 0.01])
```

---

## 3 · 自动微分 (AD)

### 3.1 Tape 记录

```python
tape = wp.Tape()
with tape:
    for t in range(T):
        wp.launch(integrate, dim=N, inputs=[x, v, k, dt])

loss = compute_loss(x)
tape.backward(loss)
print(k.grad)  # ∂loss/∂k
```

Tape 维护操作记录 (类似 PyTorch autograd):
- 正向: 记录每个 kernel 的输入输出。
- 反向: 通过链式法则反推梯度。

### 3.2 反向 kernel 自动生成

Warp 编译器对每个 `@wp.kernel` 自动生成对应 backward kernel:
$$\bar{x} \mathrel{+}= \frac{\partial L}{\partial x}$$
通过对操作序列反向传播。

### 3.3 Checkpointing

长 rollout 时显存爆炸, Warp 支持:
- 每 $\sqrt{T}$ 步存一次状态 (checkpoint)。
- 反向时重算缺失部分。
- 显存 $O(T) \to O(\sqrt{T})$。

---

## 4 · 内置物理库

### 4.1 FEM 软体

构建四面体网格 + Neo-Hookean 本构:
```python
builder = wp.sim.ModelBuilder()
builder.add_soft_mesh(vertices, indices, density=1000.0,
                       k_mu=1e5, k_lambda=1e5)
model = builder.finalize()
state = model.state()
integrator = wp.sim.SemiImplicitIntegrator()
for t in range(T):
    state = integrator.simulate(model, state, dt)
```

### 4.2 Position-Based Dynamics (PBD)

约束求解迭代:
```
for iter = 1..max_iter:
    for each constraint c:
        Δp = compute_correction(c)
        x ← x + Δp · w_i / Σ w_j
```

适合布料、绳子、流体粒子。

### 4.3 接触

Warp 内置 SDF 接触:
```python
@wp.kernel
def contact(x: wp.array(...), 
            sdf: wp.uint64,    # SDF mesh
            f: wp.array(...)):
    tid = wp.tid()
    d = wp.mesh_query_point(sdf, x[tid])
    if d < 0.0:
        n = wp.mesh_query_normal(sdf, x[tid])
        f[tid] = -k_n * d * n
```

---

## 5 · 性能基准

Warp 论文/文档报告:

| 任务 | CPU (单核) | Warp GPU |
|---|---|---|
| FEM 1k 四面体 1k 步 | ~10 s | ~0.1 s |
| MPM 流体 100k 粒子 | ~分钟 | ~秒 |
| 接触 10k 刚体 | ~秒 | ~毫秒 |

100-1000× 加速 (依赖 GPU 型号)。

---

## 6 · 在本论文 (Yi'25) 中的角色

```python
# 本论文 §3.1 仿真器伪代码
import warp as wp
import warp.sim as wps

builder = wps.ModelBuilder()
# 构建软指 (FEM tet 网格)
builder.add_soft_mesh(vertices=indents, indices=tets,
                      density=1100, k_mu=mu_log_E,
                      k_lambda=lambda_log_E)
# 添加腱 waypoint 约束
for waypoint_xyz in tendon_waypoints:
    builder.add_particle(waypoint_xyz)
model = builder.finalize(device='cuda')
integrator = wps.SemiImplicitIntegrator()

state = model.state()
for t in range(850):       # 4000 fps × ~0.21 s
    apply_tendon_force(state, f_T)
    state = integrator.simulate(model, state, dt=1/4000)
```

但本论文 **不直接用 Warp 的 AD 梯度** —— FEM + 接触的原始梯度不稳, 改用 NN 代理。

---

## 7 · Warp 的关键贡献

1. **Python + GPU**: 让物理算法编写门槛大降。
2. **AD 内置**: 直接做可微仿真。
3. **NVIDIA 生态**: 与 IsaacSim、PyTorch 紧密集成。
4. **开源**: GitHub 持续更新。

---

## 8 · 局限

1. **生态较新**: 文档与示例少于成熟工具。
2. **AD 在接触场景仍有数值问题** (本论文实测)。
3. **CPU 后端弱**。

---

## 9 · 历史影响

- 与 DiffTaichi ([[Ref-19-Hu-2020-DiffTaichi]]) 一道, 推动 Python 可微物理普及。
- NVIDIA IsaacGym / IsaacLab 等 RL 工具底层。
- 学术界软体机器人仿真新标准。

---

## 10 · 与本论文 (Yi'25) 的关系

本论文 **数据生成** 完全依赖 Warp。论文 §3.1 写明:
> "We develop our tendon-driven soft finger simulator in Nvidia Warp [18], using the Finite-Element Method for soft bodies."

但 **不用 Warp 的 AD**, 改用 NN 代理 → AD 路径走 PyTorch。

---

*← [[_参考文献索引]]*
