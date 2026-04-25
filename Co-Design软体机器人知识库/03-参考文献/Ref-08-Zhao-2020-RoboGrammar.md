# [8] Zhao et al. 2020 · RoboGrammar: 图语法 + MCTS 机器人设计

> **引用**: A. Zhao, J. Xu, M. Konaković-Luković, J. Hughes, A. Spielberg, D. Rus, W. Matusik. *RoboGrammar: Graph grammar for terrain-optimized robot design.* **ACM TOG (SIGGRAPH ASIA)**, 39(6):1-16, 2020.

`#文献-方法` `#规划式设计` `#图语法` `#里程碑`

---

## 🎯 这篇论文想做什么

机器人形态空间 **离散 + 高维 + 拓扑可变**——添加一段腿、改变关节类型、增加足端等都是离散操作。梯度方法、BO 都难处理。Zhao 等的目标：

> 用 **正式语言 (formal grammar)** 描述机器人结构空间，把设计问题变成 **"在语法树中找最优树"**，然后用 MCTS 高效搜索。

应用场景：地形适应——给定一种地形 (平地/楼梯/障碍)，自动生成最适合该地形的机器人结构。

---

## 🛠️ 实现路径

### 1. 图语法 (Graph Grammar) 描述机器人

定义：
- **非终结符 (Non-terminal)**：抽象部件 (例如 "Body", "Leg", "Joint")。
- **终结符 (Terminal)**：具体部件 (例如 "rigid bar 5cm", "revolute joint")。
- **产生式规则**：把非终结符替换成终结符 + 非终结符的组合。

例：
```
Robot ::= Body + 4Legs
Body ::= rigid_box(W, H, L)
Leg ::= Hip + Knee + Ankle + Foot
Hip ::= revolute_joint + bar_5cm
Knee ::= revolute_joint + bar_3cm
...
```

每条规则保证 **结构可制造、可仿真、可控制**。

### 2. 蒙特卡洛树搜索 (MCTS)

把"应用产生式规则" 视为搜索动作：
- 状态 = 当前已展开的设计树。
- 动作 = 选一个非终结符并应用一条产生式。
- 终止 = 所有非终结符都展开完。

每个完整树 = 一个机器人设计。

**MCTS 标准 4 步**：
1. **Selection**：用 UCB1 选择子节点
$$\text{UCB}(c) = \bar{Q}(c) + C\sqrt{\frac{\ln N(p)}{N(c)}}$$
2. **Expansion**：扩展一个新节点 (新规则)。
3. **Rollout**：随机展开到叶子，得到一个完整设计；评估。
4. **Backup**：把奖励反传到路径上。

### 3. 设计评估 = MPC 训练 + 仿真

每个候选设计：
- 在地形上构建机器人；
- 训练 MPC 控制器走 N 秒；
- 评估前进距离作为奖励。

---

## 📐 关键公式与使用场景

### 公式 1：UCB1 树搜索
$$\text{UCB}(c) = \bar{Q}(c) + C\sqrt{\frac{\ln N(p)}{N(c)}}$$

**含义**：第 1 项为利用 (exploitation)，第 2 项为探索 (exploration)；$C$ 调节平衡。

**使用场景**：当设计空间是离散组合 (产生式规则的应用)，UCB 高效定向搜索。

### 公式 2：设计奖励
$$R(\text{design}) = \max_{\text{controller}} d_\text{forward}(\text{design}, \text{controller})$$

**含义**：每个设计的得分由 **它在最优控制下的最佳表现** 决定。

**使用场景**：co-design 中"硬件 + 控制" 的标准目标——硬件外层、控制内层。

### 实验亮点

- 在 5 种地形上各搜索 200 个设计 ($\sim 10^4$ 仿真)。
- **结构多样性**：同一地形涌现多种合法形态 (四足、六足、混合)。
- 性能超过 NEAT、HyperNEAT 等基线。

---

## 🔑 对本论文 (Yi'25) 的意义

- **方法对比**：RoboGrammar 在 **离散结构空间** 用规划方法；本论文在 **连续刚度空间** 用梯度方法。两者解决不同 co-design 子问题。
- **共同哲学**：都强调"探索合法、可制造的设计空间" 而非自由组合。
- **未来融合**：RoboGrammar 生成结构骨架 + 本论文优化局部刚度，是自然的两阶段 co-design 流程。

---

## 💡 延伸思考

图语法是 **形式语言学** 与 **机器人学** 的交汇——这与 LLM 时代的"代码生成机器人结构" 一脉相承。RoboGrammar 可视为 GPT 写机器人代码的"经典前身"。

---

## 🧭 延伸阅读

- 规划式设计分类：[[../02-方法分类/规划式设计]]
- 同 MIT 团队可微方法：[[Ref-04-Xu-2021-可微设计框架]]
- 进化机器人开山：[[Ref-03-Lipson-2000-自动设计生命形态]]
- 模块化机器人 GA：[[Ref-25-Kulz-2024-ModularGA]]
- 生成式设计：[[Ref-26-Ha-2021-Fit2Form]]、[[Ref-27-Xu-2024-DynamicsDiffusion]]

---

*← [[_参考文献索引]]*
