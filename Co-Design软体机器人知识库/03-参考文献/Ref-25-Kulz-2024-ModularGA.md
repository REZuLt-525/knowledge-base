# [25] Külz & Althoff 2024 · Modular Robot Composition via Lexicographic GA

> **引用**: J. Külz and M. Althoff. *Optimizing modular robot composition: A lexicographic genetic algorithm approach.* **ICRA 2024**, pp. 16752-16758.

`#文献-方法` `#遗传算法` `#模块化机器人`

---

## 🎯 这篇论文想做什么

模块化机器人 (例如 Schunk LWA、华为模块臂) 由若干 **可互换关节模块** 组合而成。给定任务，工程师手工挑模块组合很费时。TUM 团队问：

> 如何 **自动选最佳模块序列**，在多目标 (轻量 / 工作空间 / 成本) 之间平衡？

挑战：
- 离散组合空间 (10 模块库 × 6 关节 = $10^6$ 种组合)；
- 多目标无主观加权偏好。

---

## 🛠️ 实现路径

### 1. 模块字典

每个模块 $m$ 有属性向量 $(L, m_\text{kg}, \tau_\text{max}, \text{type})$ —— 长度、质量、最大力矩、类型。

机器人 = 一串模块序列 $[m_1, m_2, ..., m_n]$。

### 2. 词典序遗传算法 (Lexicographic GA)

经典多目标 GA 用 Pareto 前沿；Külz 选 **词典序 (lexicographic)**：
- 用户给出 **目标优先级列表**：$o_1 \succ o_2 \succ \cdots \succ o_k$。
- 算法 **先优化 $o_1$**；在 $o_1$ 等值的解集中再优化 $o_2$，依次类推。

```
GA 主循环:
   按优先级 o_1, o_2, ... 评估种群
   按 lexicographic 比较排序
   选择 top → 交叉 + 变异 → 新代
```

### 3. 任务适应度

每个候选机器人：
- 验证可达 (运动学逆解)；
- 仿真任务表现 (循环时间、负载、能耗)；
- 评估 ($o_1, o_2, ..., o_k$)。

---

## 📐 关键算法与使用场景

### 算法 1：Lexicographic 比较
```
def lex_better(a, b, tol):
    for i in range(k):
        if a[i] < b[i] - tol[i]:
            return True
        elif a[i] > b[i] + tol[i]:
            return False
    return False  # 相等
```

**使用场景**：
- **目标层次明确** 的工程问题 (安全 > 性能 > 成本)。
- 避免多目标加权的主观系数。

### 算法 2：模块编码
```
chromosome = [module_id_1, module_id_2, ..., module_id_n]
```

**使用场景**：
- **离散结构 co-design**：本质是字符串/序列优化。
- 类似 [[Ref-08-Zhao-2020-RoboGrammar|RoboGrammar]] 的图语法但更简单。

---

## 🔑 对本论文 (Yi'25) 的意义

- **离散 vs 连续** 的对照：本论文设计空间是 22 维连续刚度，不需 GA；但当扩展到 **"模块化软指 + 刚度"** 时，词典序 GA 可结合本论文神经代理使用。
- **多目标处理思路**：本论文用单加权损失 ($w_1 + w_2$ 加权)，词典序 GA 提供另一种处理方式 —— 当目标优先级明确时更直观。

---

## 🧭 延伸阅读

- CMA-ES：[[Ref-24-Hansen-2003-CMA-ES]]
- 图语法离散设计：[[Ref-08-Zhao-2020-RoboGrammar]]
- 进化机器人开山：[[Ref-03-Lipson-2000-自动设计生命形态]]
- 采样方法：[[../02-方法分类/采样优化方法]]

---

*← [[_参考文献索引]]*
