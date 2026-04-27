# [8] Zhao et al. 2020 · RoboGrammar

> **原文**: A. Zhao, J. Xu, M. Konaković-Luković, J. Hughes, A. Spielberg, D. Rus, W. Matusik. *RoboGrammar: Graph grammar for terrain-optimized robot design.* **ACM TOG (SIGGRAPH ASIA)**, 39(6):1-16, 2020.
> **译名**: 《RoboGrammar: 面向地形优化机器人设计的图语法》 — ACM 图形学汇刊 (SIGGRAPH ASIA), 2020.

`#文献-方法` `#规划式设计` `#图语法` `#里程碑`

---

## 1 · 研究问题

机器人 **形态空间** 是 **离散 + 高维 + 拓扑可变** 的: 加一段腿、改关节类型、变足端等都是离散选择。梯度方法 / BO 都难处理。MIT CSAIL Daniela Rus + Wojciech Matusik 团队的目标：

> 用 **形式语言 (formal grammar)** 严格定义机器人结构空间，把 "**机器人设计**" 转为 "**在语法树中找最优树**" 问题，再用 MCTS 高效搜索 + MPC 评估。

应用：地形适应——给定地形 (平地/楼梯/障碍)，自动生成最适合该地形的机器人结构。

---

## 2 · 形式化

### 2.1 图语法 (Graph Grammar)

字母表:
- **非终结符 $N$**: 抽象部件，例 `Robot`, `Body`, `Leg`, `Joint`。
- **终结符 $T$**: 具体部件，例 `bar_5cm`, `revolute_joint`, `wheel`。

产生式规则集 $R$，每条形如:
$$A \rightarrow w_1 w_2 \cdots w_k$$
其中 $A \in N$, $w_i \in N \cup T$。

例:
```
Robot     ::= Body + Legs(4)
Body      ::= rigid_box(W, H, L)
Legs(n)   ::= Leg | Leg + Legs(n-1)
Leg       ::= Hip + Knee + Ankle + Foot
Hip       ::= revolute_joint + bar_5cm
Knee      ::= revolute_joint + bar_3cm
...
```

每个完全推导的句子 = 一个机器人结构。

### 2.2 设计空间 = 推导树空间

把"应用规则" 视为搜索动作:
- 状态 $s$: 当前已展开的推导树。
- 动作 $a$: 选一个非终结符并应用一条产生式。
- 终止: 所有非终结符都展开。

设计空间 $\mathcal{D} = \{$所有合法终结句子$\}$，规模 $\sim 10^{10}$。

### 2.3 任务: 地形上的运动

每个候选 $D \in \mathcal{D}$:
- 仿真模型转 URDF。
- 用 MPC 训控制器在指定地形跑 $T$ 秒。
- 适应度 = 前进距离。

$$\text{fitness}(D) = \max_\pi \mathbb{E}_\tau\left[\sum_t r_t(D, \pi)\right]$$

---

## 3 · MCTS 搜索算法

```
─── Algorithm: MCTS for Robot Design ─────────
节点 = 部分推导树
重复 simulation = 1..N_sim:
    1) Selection (从 root 沿树下行):
       child = argmax UCB1(child)
       UCB1(c) = avg_Q(c) + C·sqrt(ln N_p / N_c)
    2) Expansion: 对未展开非终结符, 尝试一条产生式 → 新节点
    3) Rollout (随机展开到完整设计):
       while 还有非终结符:
           随机选一条产生式
       得完整设计 D, 评估 fitness(D)
    4) Backup: 把 reward 沿路径反传
最终选择: 路径中 fitness 最高的设计
```

每次评估需 MPC 训练 + 仿真 (~分钟级)，总搜索 ~$10^4$ 评估。

---

## 4 · 实验

### 4.1 测试地形
1. 平地 (baseline)
2. 楼梯
3. 山地
4. 不规则障碍
5. 平台

### 4.2 量化结果

| 地形 | 搜得最优 | 速度 |
|---|---|---|
| 平地 | 4 腿对称 | 1.2 m/s |
| 楼梯 | 6 腿不对称 | 0.8 m/s |
| 山地 | 长腿 + 弯关节 | 0.6 m/s |

### 4.3 多样性
同一任务下 MCTS 找到 3-5 种合理形态 → 设计空间存在多个局部极大值。

### 4.4 对照基线
- 随机搜索: 平均 50% 性能。
- NEAT/HyperNEAT: 平均 70% 性能。
- **RoboGrammar (MCTS): 90% 性能**。

---

## 5 · 关键贡献

1. **图语法描述机器人**: 第一次系统性地把 formal grammar 应用于机器人设计。
2. **保证可制造**: 规则集本身限制只产出物理可行结构。
3. **MCTS 高效搜索**: 比纯进化算法效率高一个数量级。
4. **多样性涌现**: 不同地形产出根本不同的机器人形态。

---

## 6 · 局限

1. **语法需手工设计**: 规则集本身限制了结构搜索范围。
2. **MPC 评估慢**: 每个候选要训控制器。
3. **不自动学控制**: MPC 是预设的, 不与设计联合学习。
4. **离散结构难微分**: 梯度 / 代理思路无法直接套用。

---

## 7 · 历史影响

- 启发 LLM 时代的 "代码生成机器人结构" 思路。
- 与 [[Ref-04-Xu-2021-可微设计框架|DiffHand]] 形成离散 vs 连续的对照。
- 至今仍是机器人离散设计的代表方法。

---

## 8 · 与本论文 (Yi'25) 的关系

互补: RoboGrammar 处理 **离散结构空间** (拓扑), 本论文处理 **连续刚度空间** (材料)。未来融合 = 用语法选拓扑 + 神经代理优化局部刚度。

---

*← [[_参考文献索引]]*
