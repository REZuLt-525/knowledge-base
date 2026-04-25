# [27] Xu, Ha & Song 2024 · Dynamics-Guided Diffusion for Manipulator Design

> **引用**: X. Xu, H. Ha, S. Song. *Dynamics-guided diffusion model for robot manipulator design.* arXiv:2402.15038, 2024.

`#文献-方法` `#生成式设计` `#扩散模型`

---

## 🎯 这篇论文想做什么

[[Ref-26-Ha-2021-Fit2Form|Fit2Form]] 用 GAN 生成夹爪，但 GAN 多样性差、训练不稳。Columbia 团队跟进：

> 用 **扩散模型 (Diffusion Model)** 替代 GAN，引入 **动力学引导 (dynamics guidance)**，实现 **多样化 + 高质量** 的机器人操作器生成。

应用：开放式末端执行器设计——不仅是"夹爪"，更广泛的操作工具 (拧螺丝、打蛋、剪线等)。

---

## 🛠️ 实现路径

### 1. 扩散模型基础

扩散模型经过 **正向加噪** + **反向去噪** 训练：

正向 (noising)：
$$\mathbf{x}_t = \sqrt{\bar\alpha_t}\mathbf{x}_0 + \sqrt{1-\bar\alpha_t}\boldsymbol{\epsilon}$$

反向 (denoising)：训练 $\boldsymbol{\epsilon}_\theta(\mathbf{x}_t, t)$ 预测加入的噪声，损失为
$$L = \mathbb{E}\|\boldsymbol{\epsilon} - \boldsymbol{\epsilon}_\theta(\mathbf{x}_t, t)\|^2$$

采样时从纯噪声反向去噪到设计样本。

### 2. 引入动力学引导

在反向去噪过程中加入 **动力学梯度**：
$$\mathbf{x}_{t-1} = \mu_\theta(\mathbf{x}_t, t) + s\,\boldsymbol{\sigma}\,\nabla_\mathbf{x} \log p(d | \mathbf{x}_t)$$

其中 $p(d | \mathbf{x})$ 是"该设计能完成期望动作 $d$" 的概率，由可微仿真或代理评估。

类似 classifier-free guidance：
$$\boldsymbol{\epsilon}_\text{guided} = \boldsymbol{\epsilon}_\theta(\mathbf{x}_t, t, d) + w\,(\boldsymbol{\epsilon}_\theta(\mathbf{x}_t, t, d) - \boldsymbol{\epsilon}_\theta(\mathbf{x}_t, t))$$

### 3. 操作任务条件

$d$ 编码任务需求 (动作轨迹、力学性能)：
- 抓重 100g 的瓶子 → 夹爪要稳；
- 打开纸箱盖 → 末端要细且硬。

不同任务下生成的设计差异巨大。

---

## 📐 关键公式与使用场景

### 公式 1：动力学引导
$$\nabla_\mathbf{x} \log p(\text{task success} | \mathbf{x})$$

**使用场景**：
- 让生成模型不只考虑外观，更考虑功能。
- co-design 中的 **"设计 → 任务"** 反向引导。

### 公式 2：扩散采样
$$\mathbf{x}_{t-1} = \frac{1}{\sqrt{\alpha_t}}\left(\mathbf{x}_t - \frac{1-\alpha_t}{\sqrt{1-\bar\alpha_t}}\boldsymbol{\epsilon}_\theta\right) + \sigma_t \mathbf{z}$$

**使用场景**：每步反向去噪即一次设计精炼。

---

## 🔑 对本论文 (Yi'25) 的意义

- 都是 **"NN + 物理感知"** co-design 的代表，但路径完全不同：
  - 扩散模型：**生成式**，一次性产出多样化设计；
  - 本论文：**优化式**，迭代精炼一个设计。
- 互补 use case：
  - 扩散适合 **从 0 探索新形态**；
  - 本论文适合 **在已知形态上精调局部参数**。
- 未来融合：扩散生成几何骨架 → 本论文优化局部刚度。

---

## 🧭 延伸阅读

- Fit2Form：[[Ref-26-Ha-2021-Fit2Form]]
- 生成式设计：[[../02-方法分类/生成式设计]]
- 反设计 GNN 路径：[[Ref-20-Allen-2022-InverseDesignGNS]]

---

*← [[_参考文献索引]]*
