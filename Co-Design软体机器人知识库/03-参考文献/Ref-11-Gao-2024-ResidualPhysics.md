# [11] Gao et al. 2024 · Sim-to-Real of Soft Robots with Learned Residual Physics

> **引用**: J. Gao, M. Y. Michelis, A. Spielberg, R. K. Katzschmann. *Sim-to-real of soft robots with learned residual physics.* **IEEE RAL**, 2024.

`#文献-方法` `#Sim-to-Real` `#软体机器人`

---

## 🎯 这篇论文想做什么

软体机器人在仿真里学到的策略迁移到真机时，**性能往往大幅下降**——材料属性、阻尼、空气泄漏、温度漂移共同作用。ETH Katzschmann 团队的目标：

> 不修复仿真器，而是学一个 **残差网络** 补偿仿真与真实的差距，**简单有效地** 弥合 Sim-to-Real 鸿沟。

---

## 🛠️ 实现路径

### 1. 仿真基线 (粗模型)

用低保真模型快速得到状态:
$$s_{t+1}^\text{sim} = f_\text{sim}(s_t, a_t)$$

例：常曲率 (PCC) 模型对软臂的预测。

### 2. 残差学习

收集少量真机数据：
$$\Delta_t = s_{t+1}^\text{real} - s_{t+1}^\text{sim}$$

训神经网络 $g_\theta$ 学习残差：
$$\hat\Delta_t = g_\theta(s_t, a_t)$$

最终预测：
$$\hat s_{t+1} = f_\text{sim}(s_t, a_t) + g_\theta(s_t, a_t)$$

### 3. 部署策略

策略在 **修正后的混合模型** 上训练，再迁移到真机：
$$\pi^* = \arg\max_\pi \mathbb{E}\big[\sum_t r_t \mid \hat s\big]$$

### 4. 实验

- ETH 自制软臂 (PneuNet 风格)。
- 真机数据采集 ~20 分钟。
- 残差补偿后 sim-real gap 从 ~30% 降至 ~5%。

---

## 📐 关键公式与使用场景

### 公式 1：残差物理
$$\hat s_{t+1} = f_\text{sim}(s_t, a_t) + g_\theta(s_t, a_t)$$

**使用场景**：
- **任何 sim 与 real 有偏差** 的机器人系统。
- 不需要重新建模仿真器，**最小侵入** 的 Sim-to-Real 方案。
- **本论文 (Yi'25) 的对比**：本论文不学残差，而是 **直接对打印参数 ↔ 杨氏模量做静态标定**——更简单，但需更多手工工作。

### 公式 2：少样本残差
**思想**：$g_\theta$ 只学差值，比直接学动力学需要的数据少 1-2 个数量级。

**使用场景**：真机数据稀缺时的最佳方案。

---

## 🔑 对本论文 (Yi'25) 的意义

- **Sim-to-Real 路径对比**：
  - Gao'24: 学残差 (动态补偿)；
  - 本论文: 测真材料 (静态标定)；
  - 各有适用场景。
- **互补性**：未来若把残差物理叠加到本论文神经代理上，可以让代理更精确反映真机制造误差。

---

## 🧭 延伸阅读

- Schaff'23 (DR Sim-to-Real)：[[Ref-06-Schaff-2023-CrawlerSimToReal]]
- MORPH (硬件代理)：[[Ref-07-He-2024-MORPH]]
- 神经物理：[[../01-核心概念/神经物理模拟器 Neural Physics]]
- 概念：[[../01-核心概念/Sim-to-Real]]

---

*← [[_参考文献索引]]*
