# [23] Snoek, Larochelle & Adams 2012 · Practical Bayesian Optimization of ML Algorithms

> **引用**: J. Snoek, H. Larochelle, R. P. Adams. *Practical Bayesian optimization of machine learning algorithms.* **NIPS 2012**. arXiv:1206.2944

`#文献-方法` `#贝叶斯优化` `#里程碑`

---

## 🎯 这篇论文想做什么

机器学习模型有很多超参 (学习率、正则化、网络宽度...)，调优一直靠 grid search 或随机搜索，**慢且不智能**。Snoek 等的目标：

> 把 **贝叶斯优化** 真正做"实用" —— 解决核函数选择、并行评估、采集函数选择三大瓶颈。

具体场景：让 BO 把 CNN 在 CIFAR-10 上的最佳超参搜出来，比手调还快。

---

## 🛠️ 实现路径

### 核心循环 (BO 通用)

```
重复 t = 1, 2, ...
  1) 用观测数据 D = {(x_i, y_i)} 拟合 GP 代理 f̂
  2) 用采集函数 a(x; f̂) 求下一个最有希望的 x_t
  3) 真实评估 y_t = f(x_t)
  4) D ← D ∪ {(x_t, y_t)}
```

### 1. 高斯过程代理

GP 把函数值看成多元正态：
$$f(\mathbf{x}) \sim \mathcal{GP}(\mu(\mathbf{x}), k(\mathbf{x}, \mathbf{x}'))$$

后验均值与方差：
$$\mu_n(\mathbf{x}) = \mathbf{k}_n^\top (\mathbf{K}_n + \sigma_n^2\mathbf{I})^{-1}\mathbf{y}_n$$
$$\sigma_n^2(\mathbf{x}) = k(\mathbf{x},\mathbf{x}) - \mathbf{k}_n^\top (\mathbf{K}_n + \sigma_n^2\mathbf{I})^{-1}\mathbf{k}_n$$

### 2. Matérn 5/2 核 ⭐ 论文重点

Snoek 选 **Matérn 5/2** 而非常用的 RBF：
$$k_{\nu=5/2}(r) = \left(1 + \sqrt{5}r + \frac{5r^2}{3}\right)\exp(-\sqrt{5}r)$$
$$r = \sqrt{\sum_i (x_i - x_i')^2 / \ell_i^2}$$

**理由**：RBF 假设函数无限光滑——超参搜索空间往往不那么平滑；Matérn 5/2 (二阶可微) 更贴近现实。

### 3. Expected Improvement (EI) 采集函数

$$\text{EI}(\mathbf{x}) = \mathbb{E}\left[\max(f^\star - f(\mathbf{x}), 0)\right]$$
对 GP 后验解析积分：
$$\boxed{\text{EI}(\mathbf{x}) = (f^\star - \mu(\mathbf{x}))\Phi(z) + \sigma(\mathbf{x})\phi(z),\quad z=\frac{f^\star-\mu(\mathbf{x})}{\sigma(\mathbf{x})}}$$
其中 $\Phi$ 是标准正态 CDF, $\phi$ 是 PDF。

### 4. 并行 BO (论文创新点之一)

把 EI 推广到 batch：用 **Monte Carlo Fantasy** —— 假设当前已采样但未评估的点 $\mathbf{x}_p$ 的值是 GP 后验抽样 $\tilde{y}_p$，把它当作真值再选下一个 batch 成员。

---

## 📐 关键公式与使用场景

### 公式 1：EI 采集函数
$$\text{EI}(\mathbf{x}) = (f^\star - \mu(\mathbf{x}))\Phi(z) + \sigma(\mathbf{x})\phi(z)$$

**直观解释**：
- 第 1 项：均值改进部分 (利用)。
- 第 2 项：方差贡献部分 (探索)。
- 当 $\sigma=0$ 时退化为贪心。

**使用场景**：
- **机器人步态调优** (Calandra 2016, Ref [22])：参数维度 ~10。
- **超参搜索**：CNN/RNN/XGBoost 等。
- **co-design 低维子问题**：例如固定几何下调 PID 增益、单根腱张力。

### 公式 2：Matérn 5/2 核
$$k_{5/2}(r) = (1+\sqrt 5 r + 5r^2/3)\exp(-\sqrt 5 r)$$

**使用场景**：当目标函数 **不是无限光滑** 但可微 (例如带噪音的真实实验数据)，Matérn 5/2 比 RBF 更稳。这条建议沿用至今 ([[Ref-22-Calandra-2016-BO|Calandra 2016]] 也用)。

---

## 🎯 实践要点

- **维度上限**：BO 通常在 $n \le 20$ 维表现最好，超过 50 维需要 trust-region BO 或 high-dim 变体。
- **样本数**：理想运行在 $50$-$1000$ 次评估范围。
- **$f^\star$ 选取**：当前最佳值；初期可加 jitter 鼓励探索。

---

## 🔑 对本论文的意义

- **维度限制是关键**：本论文设计空间 22 维刚度 + 6D 位姿 = 28 维，**正好处于 BO 甜区的边缘**。
- **评估代价**：每次 FEM 模拟 ~分钟级，BO 在数百评估内确实可行；但本论文需要 **同时优化 45 个物体的联合损失**，数据需求 $\times 45$，BO 不再划算。
- **结论**：BO 适合"低维 + 单目标 + 评估贵"，本论文是"中维 + 多物体联合 + 评估贵但可批量并行"——后者更适合代理梯度。

---

## 🧭 延伸阅读

- BO 在机器人步态：[[Ref-22-Calandra-2016-BO]]
- 进化策略对比：[[Ref-24-Hansen-2003-CMA-ES]]
- 采样优化分类：[[../02-方法分类/采样优化方法]]

---

*← [[_参考文献索引]]*
