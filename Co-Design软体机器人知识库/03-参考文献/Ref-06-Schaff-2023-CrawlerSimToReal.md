# [6] Schaff et al. 2023 · Sim-to-Real Co-Optimized Soft Crawler

> **引用**: C. Schaff, A. Sedal, S. Ni, M. R. Walter. *Sim-to-real transfer of co-optimized soft robot crawlers.* **Autonomous Robots**, 47(8):1195-1211, 2023.

`#文献-方法` `#软体机器人` `#Sim-to-Real` `#Co-Design`

---

## 🎯 这篇论文想做什么

软体机器人 co-design 在仿真里玩得很 fancy，但 **真机迁移成功率长期低下**。Schaff (TTIC) 等的目标：

> 设计一个 **完整的 Sim-to-Real co-design 管线**：让仿真共优化的软体爬行机器人在真世界里也能爬。

具体场景：自制软体管状爬行虫，4 个充气段，目标在地毯上前进。

---

## 🛠️ 实现路径

### 1. 软体仿真器

- 基于 SOFA ([[Ref-41-Faure-2012-SOFA]]) 软体 FEM。
- 加入摩擦地面、气压驱动、阻尼。
- 支持 **设计参数** (各段长度、半径、壁厚) 与 **控制参数** (气压时序) 的联合优化。

### 2. Co-Optimization (双层结构)

```
外层: 进化算法采样设计参数
内层: PPO 训练每个候选设计的步态控制
评估: 仿真前进距离 + 鲁棒性
```

每个候选设计内层训 ~10⁵ 步 RL，外层 50-100 代。

### 3. Domain Randomization (DR)

为缓解 sim-real gap：
- 摩擦系数随机化 ($\mu \sim [0.2, 0.8]$)；
- 气压响应延迟随机化；
- 软体材料模量 ±20%；
- 重力扰动。

经过 DR 训练的策略对真实参数变化更鲁棒。

### 4. 真机制造 + 实验

- 把仿真共优化结果 3D 打印 (软硅胶模具)。
- 接入空气泵 + 电磁阀。
- 真机爬行实验：成功在地毯上以预测速度前进。

---

## 📐 关键技术与使用场景

### 技术 1：双层 co-optimization
$$\text{outer}: \arg\max_d \; \mathbb{E}_{\pi^*(d)}[R]$$
$$\text{inner}: \pi^*(d) = \arg\max_\pi \; \mathbb{E}[R | d]$$

**使用场景**：当内层是 RL 时这是 co-design 的标准范式。本论文 (Yi'25) 是 **简化版**：内层不训 RL，只做位姿采样。

### 技术 2：DR 用于 sim-real 鲁棒性
$$L_\text{train} = \mathbb{E}_{\eta \sim p(\eta)} \left[L(\pi, \eta)\right]$$

**使用场景**：让策略在多种环境扰动下保持有效。本论文用 **DR 较少**——因为优化对象是 **静态刚度** + **位姿采样**，不是长期策略，对动力学扰动天然不敏感。

---

## 🔑 对本论文 (Yi'25) 的意义

- **软体 co-design 真机化的早期里程碑**：Schaff 证明仿真训练的软体能爬到真机上。
- **本论文的差异**：
  - Schaff: locomotion (爬行)；本论文: manipulation (抓取)。
  - Schaff: PPO + 双层 + DR；本论文: NN 代理 + 位姿采样。
  - Sim-to-Real 简化: 本论文通过 **打印参数 ↔ 杨氏模量标定** 直接迁移，不依赖大规模 DR。

简言之：Schaff 是"用 RL + DR 强行硬冲" 的 Sim-to-Real，本论文是"通过精确标定 + 简化设计空间" 的 Sim-to-Real——后者更可控。

---

## 🧭 延伸阅读

- 残差物理 (另一种 Sim-to-Real)：[[Ref-11-Gao-2024-ResidualPhysics]]
- MORPH (RL + 硬件代理)：[[Ref-07-He-2024-MORPH]]
- SOFA 仿真器：[[Ref-41-Faure-2012-SOFA]]
- 概念：[[../01-核心概念/Sim-to-Real]]

---

*← [[_参考文献索引]]*
