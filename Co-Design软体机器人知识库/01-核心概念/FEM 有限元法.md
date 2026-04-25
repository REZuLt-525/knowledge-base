# FEM 有限元法

> **一句话理解**：把连续物体切成许多小块 (元素)，在每一小块上解局部物理方程，再拼成整体——是工程仿真的"通用武器"。

`#核心概念`

---

## 1. FEM 核心思想

**Finite Element Method** 将偏微分方程 (PDE) 的求解从"处处连续"转换为"节点离散":

$$\text{连续体 } \Omega \to \text{有限元网格 } \bigcup_{e} \Omega_e$$

在每个元素 $\Omega_e$ 内使用形函数 (shape functions) 近似解：
$$u(x) \approx \sum_i N_i(x)\, u_i$$

将变分原理代入后得到 **线性系统**:
$$K u = f$$

- $K$：刚度矩阵 (由材料属性 + 几何决定)
- $u$：节点位移向量
- $f$：节点力向量

---

## 2. 在软体机器人中的作用

软体机器人本质是连续介质，经典多刚体动力学不够用，常用 FEM：
- **变形**：大变形、非线性本构 (Neo-Hookean, Ogden)。
- **接触**：与刚体物体相互作用。
- **腱驱动**：腱的力传递到网格节点。

本论文在 **Warp 框架** 中使用四面体网格做 FEM，具体见 [[../03-参考文献/Ref-18-Macklin-2022-Warp]]。SOFA 框架 ([[../03-参考文献/Ref-41-Faure-2012-SOFA]]) 是软体 FEM 的另一代表。

---

## 3. 时间步迭代

软体动力学:
$$M\ddot{u} + D\dot{u} + K(u)u = f_\text{ext}(t)$$

隐式/显式时间积分 + Newton 求解。本论文每秒 **4000 帧** 跑仿真，非常密集。

---

## 4. FEM 在 Co-Design 中的挑战

1. **速度**：高分辨率网格 + 非线性材料 → 每步数秒。
2. **接触梯度**：离散接触事件导致梯度跳变，见 [[可微分模拟 Differentiable Simulation]]。
3. **自碰撞**：软体可能穿过自己，需额外处理。

**解决方案**：
- 粗化网格 (性能 vs 精度)。
- 神经代理 (本论文路线，见 [[神经物理模拟器 Neural Physics]])。
- 低阶近似 (PCC 常曲率、Cosserat 杆)。

---

## 5. 本论文的 FEM 数据管线

```
STL 物体网格
     │
     ▼
软体四面体网格 (腱 waypoint 对应顶点)
     │
     ▼
Warp FEM (4000 fps, 850 frames)
     │
     ▼
采样: (k, p, obj) → 末态 (f, Δq, c_g)
     │
     ▼
生成 ~80k 训练样本 → 训 PointNet + MLP 代理
```

---

## 6. 相关概念

- [[可微分模拟 Differentiable Simulation]]
- [[神经物理模拟器 Neural Physics]]
- [[软体机器人 Soft Robotics]]
- [[../03-参考文献/Ref-18-Macklin-2022-Warp]]
- [[../03-参考文献/Ref-41-Faure-2012-SOFA]]
- [[../03-参考文献/Ref-50-Craig-2020-MaterialsMechanics]]

---

*← 返回 [[../00-总览/主索引 MOC]]*
