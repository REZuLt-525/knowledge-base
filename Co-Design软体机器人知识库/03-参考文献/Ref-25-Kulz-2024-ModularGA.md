# [25] Külz & Althoff 2024 · 模块化机器人词典序遗传算法

> **原文**: J. Külz and M. Althoff. *Optimizing modular robot composition: A lexicographic genetic algorithm approach.* **ICRA 2024**, pp. 16752-16758.
> **译名**: 《模块化机器人组合优化：词典序遗传算法方法》 — IEEE 国际机器人与自动化大会 (ICRA), 2024.

`#文献-方法` `#遗传算法` `#模块化机器人`

---

## 1 · 研究问题

模块化机器人 (例如 Schunk LWA、Hebi Robotics 模块臂) 由若干 **可互换关节模块** 组合而成, 给定任务工程师手工挑组合很费时。TUM (Munich) Matthias Althoff 团队的目标:

> 自动化模块化机器人组合设计 — 在多目标 (轻量 / 工作空间覆盖 / 成本 / 关节数) 间平衡, 用 **词典序遗传算法 (Lexicographic GA)** 按用户优先级层级求解。

---

## 2 · 形式化

### 2.1 模块字典

每个模块 $m \in \mathcal{M}$ 有属性向量:
$$m = (L, m_\text{kg}, \tau_\text{max}, k_\text{joint}, \text{type}, \text{cost})$$
- $L$: 长度。
- $m_\text{kg}$: 质量。
- $\tau_\text{max}$: 最大关节力矩。
- $k_\text{joint}$: 刚度。
- $\text{type}$: 关节 / 链接 / 末端类型。

### 2.2 机器人编码

机器人 = 模块序列 (染色体):
$$\mathcal{R} = [m_1, m_2, ..., m_n]$$
$n$ 可变 (遗传操作可加 / 减模块)。

### 2.3 多目标评估

任务表现:
$$\mathbf{o}(\mathcal{R}) = (o_1, o_2, ..., o_K)$$
- $o_1$: 任务循环时间 (越小越好)。
- $o_2$: 总质量 (越小越好)。
- $o_3$: 工作空间覆盖率 (越大越好)。
- $o_4$: 成本 (越小越好)。

---

## 3 · 词典序比较 (Lexicographic Order)

### 3.1 用户给优先级

把目标按优先级排序:
$$o_1 \succ o_2 \succ ... \succ o_K$$

### 3.2 词典序比较函数

```python
def lex_better(a, b, tol):
    """如果 a 比 b 好, 返回 True"""
    for i in range(K):
        if a[i] < b[i] - tol[i]:    # a 显著更好
            return True
        elif a[i] > b[i] + tol[i]:  # b 显著更好
            return False
        # 否则 (近似相等), 比较下一个目标
    return False  # 所有目标都近似相等
```

容差 $\text{tol}_i$ 决定"近似相等"的阈值, 让算法在第 $i$ 个目标稍劣时仍能优化第 $i+1$ 个目标。

### 3.3 与 Pareto / 加权和对比

| 方法 | 用户输入 | 结果 |
|---|---|---|
| 加权和 $\sum w_i o_i$ | 系数 $w_i$ | 单一最优 (但 $w_i$ 主观) |
| Pareto | 无 | Pareto 前沿 (用户后选) |
| **词典序** | **优先级 + 容差** | **逐级最优** |

词典序适合 **优先级清晰** 的工程场景 (例如 "首先满足安全, 其次性能, 最后成本")。

---

## 4 · 遗传算法工作流

```
─── Algorithm: Lexicographic GA ────────────
初始化: 种群 P_0 = N 个随机机器人 (序列)
重复 generation = 1..G_max:
    # 评估
    for each R ∈ P:
        如运动学不可行: discard
        否则用仿真评估 o(R) = (o_1, ..., o_K)
    
    # 词典序排序
    P_sorted = sort(P, by=lex_compare)
    
    # 选择 top μ
    parents = P_sorted[:μ]
    
    # 交叉
    children = []
    for pair in parents:
        # 单点交叉 / 模块插入交换
        c1, c2 = crossover(pair)
        children += [c1, c2]
    
    # 变异
    for c in children:
        with prob p_m:
            mutate(c)  # 随机加/删/换模块
    
    P = parents + children
    
返回 best ∈ P
```

---

## 5 · 关键算法细节

### 5.1 运动学可达性检查
对每个候选 $\mathcal{R}$:
1. 计算 forward kinematics 得末端工作空间。
2. 检验是否覆盖任务所需姿态。
3. 不覆盖 → 丢弃。

### 5.2 模块兼容性
确保相邻模块接口 (法兰、电气) 匹配:
$$\text{interface}_\text{out}(m_i) = \text{interface}_\text{in}(m_{i+1})$$

### 5.3 模拟评估
- ROS / Gazebo 跑任务循环。
- 测周期时间、能耗。
- 用 IK + 轨迹规划。

---

## 6 · 实验

### 6.1 任务场景
- 装配线上工件抓取 + 移动。
- 桌面操作 (笔、纸)。
- 焊接 / 喷涂。

### 6.2 模块库
~10 种模块: 不同长度连杆 + 旋转 / 平移关节 + 末端工具。

### 6.3 量化结果

| 方法 | 任务时间 | 总质量 | 工作空间 |
|---|---|---|---|
| 工程师手工 | 8 s | 5 kg | 80% |
| 加权和 GA | 7 s | 4 kg | 85% |
| Pareto NSGA-II | 6.5 s | 4.2 kg | 88% |
| **词典序 GA** | **6 s** | **3.8 kg** | **90%** |

词典序在 **优先级明确** 的场景下找到更好解。

---

## 7 · 关键贡献

1. **词典序 + GA 组合**: 解决多目标主观加权问题。
2. **运动学可达性内嵌**: 避免无效候选。
3. **自动接口匹配**: 保证模块兼容。

---

## 8 · 局限

1. **优先级用户给定**: 仍依赖人工。
2. **离散搜索慢**: $n$ 大时 GA 收敛慢。
3. **未联合学控制**: 假设固定 IK / 轨迹规划。

---

## 9 · 历史影响

- 模块化机器人 (Hebi, Schunk) 商业化推动此类研究。
- 与 [[Ref-08-Zhao-2020-RoboGrammar|RoboGrammar]] 同属离散结构搜索。

---

## 10 · 与本论文 (Yi'25) 的关系

本论文综述列其为 **采样优化** 在离散结构空间的代表。本论文连续刚度优化用梯度更高效, 两者互补处理不同设计空间。

---

*← [[_参考文献索引]]*
