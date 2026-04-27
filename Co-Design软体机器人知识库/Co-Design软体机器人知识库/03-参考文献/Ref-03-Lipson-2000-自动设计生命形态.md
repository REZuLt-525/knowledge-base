# [3] Lipson & Pollack 2000 · Automatic Design and Manufacture of Robotic Lifeforms

> **原文**: H. Lipson and J. B. Pollack. *Automatic design and manufacture of robotic lifeforms.* **Nature**, 406(6799):974-978, 2000.
> **译名**: 《机器人生命形态的自动设计与制造》 — *自然*, 2000.

`#文献-方法` `#里程碑` `#Co-Design开山`

---

## 1 · 研究问题

2000 年前的机器人都遵循 "**人类设计 → 人类制造 → 人类编程**" 的串行流程。Brandeis 大学 Hod Lipson 与 Jordan Pollack 提出根本性挑战：

> 在不使用任何人类设计输入的情况下，**能否让算法 (a) 演化出机器人的形态结构, (b) 同时演化对应的神经控制器, (c) 自动 3D 打印出实物, (d) 在物理世界完成预定任务?**

定义任务为"**在平面上向前移动**"，论文要演示 **形态-控制协同进化** 的全闭环可行性。

---

## 2 · 系统形式化

### 2.1 机器人编码 — 加权图

机器人 $\mathcal{R} = (V, E, N, S)$:
- $V = \{v_1, ..., v_n\}$：球关节 (节点)，$v_i = (x_i, y_i, z_i)$。
- $E \subseteq V \times V$：刚性杆 / 线性执行器 (边)，每条边带类型标签 $\tau_{ij} \in \{\text{bar}, \text{actuator}\}$。
- $N$：神经元集合，每个神经元 $n$ 接收若干边的状态作为输入。
- $S$：神经元间突触 + 权重 $w \in \mathbb{R}$。

执行器长度变化方程:
$$l_{ij}(t) = l_{ij}^0 + A_{ij} \sin\big(\omega t + \phi_{ij} + \sum_k w_{k} \cdot n_k(t)\big)$$

### 2.2 适应度函数

让机器人在仿真物理世界 (重力 + 地面摩擦) 运行 $T$ 秒，记录质心前进距离：
$$\text{fitness}(\mathcal{R}) = \big[ x_\text{center}(T) - x_\text{center}(0) \big] \cdot \mathbb{1}[\text{not collapsed}]$$

### 2.3 进化算法

```
─── Algorithm: Co-Evolution ───────────────────
Population P₀ ← random_init(N=200)
for generation g = 1..300:
    1) 评估: fitness(R) for each R ∈ P_{g-1}
    2) 选择: top 50% (锦标赛选择)
    3) 交叉:
        for each pair (R_a, R_b):
            子图交换 (随机切边)
            神经网络权重交叉
    4) 变异:
        以概率 p_node 增删节点
        以概率 p_edge 增删边
        以概率 p_w 改变神经权重
    5) P_g ← 新种群
return argmax fitness over all generations
```

### 2.4 自动制造

- **3D 打印** (1999 年成熟的 Stereolithography):
  - 节点 = 球关节 (打印整体)。
  - 杆 = 直线段。
- **后期插装** 直流电机到 actuator 边。
- 通电运行验证。

---

## 3 · 实验设置与结果

### 3.1 仿真平台
2D / 简化 3D 物理引擎，模拟刚体动力学 + 接触摩擦。
时间步 $dt = 0.01$ s，运行 $T = 100$ s。

### 3.2 演化结果
- 种群规模 200，世代 300，约 60,000 次仿真评估。
- 适应度从 ~0.5 m 提升到 ~5 m。

### 3.3 涌现的运动策略
论文报告几种 **算法自发发现** 的运动方式：
1. **对称三足摆动** — 类似爬行虫；
2. **不对称推动** — 类似单脚跳；
3. **波浪推进** — 多关节序贯运动。

这些策略 **没有人类规则** 灌输，全是适应度驱动的进化结果。

### 3.4 真机验证
打印 + 装配后置于平地：
- 装配成功率 ~90%。
- 真机运动模式与仿真一致 (有摩擦微调)。

---

## 4 · 关键贡献

1. **首次端到端 co-design 闭环**: 演化 → 制造 → 真实物理验证。
2. **形态-控制耦合演示**: 单纯优化形态或控制都不够，必须一起。
3. **方法无关性**: 进化算法不需可微，适用任何任务。
4. **公开种群多样性**: 同一任务出现多种合理形态——证明 co-design 空间多极小。

---

## 5 · 局限

- 评估极慢: 60k 仿真。
- 离散 + 连续混合空间，进化效率低。
- 真机制造受限于当时 3D 打印技术 (材料、连接强度)。

---

## 6 · 历史影响

- **奠定 co-design 学科**: 后 25 年内此文被几乎所有 co-design 论文引用为开山。
- 启发后续 RoboGrammar ([[Ref-08-Zhao-2020-RoboGrammar]])、HaaP ([[Ref-05-Chen-2020-HardwareAsPolicy]]) 等系列工作。
- Hod Lipson 后转 Cornell / Columbia，持续推动 "Self-aware robots"、"3D 打印生物" 等方向。

---

## 7 · 与本论文 (Yi'25) 的关系

本论文综述把它列为 co-design 起点。两者关键差异:
- Lipson'00: 进化算法 (无梯度)，适合形态拓扑。
- Yi'25: 神经代理 + 梯度，适合连续刚度参数。

两者哲学共通: **算法找出人类直觉之外的设计**。

---

*← [[_参考文献索引]]*
