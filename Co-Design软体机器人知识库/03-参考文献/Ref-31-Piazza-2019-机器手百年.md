# [31] Piazza et al. 2019 · A Century of Robotic Hands

> **引用**: C. Piazza, G. Grioli, M. G. Catalano, A. Bicchi. *A century of robotic hands.* **Annual Review of Control, Robotics, and Autonomous Systems**, 2(1):1-32, 2019.

`#文献-综述` `#机器手历史`

---

## 🎯 这篇综述想做什么

机器人手 (robotic hand) 从 19 世纪假肢发展至今 100 多年，技术路线变化巨大。Antonio Bicchi (Pisa / IIT，Pisa-IIT SoftHand 发明者) 团队梳理：

> 把 100 年来 **机器手设计的核心思想演变** 总结成一篇综述，回答 "未来手在哪里" 这个问题。

---

## 📖 五大时代

### 时代 1：1900-1950 — 假肢与机械
- 一战后大量需求 → 机械假肢。
- 单自由度抓握，靠身体动作驱动。

### 时代 2：1950-1990 — 工业刚性
- Stanford / MIT 早期机器手 (Salisbury Hand, Utah/MIT Hand)。
- 多自由度、马达驱动、刚性结构。

### 时代 3：1990-2010 — 拟人灵巧
- Shadow Hand (24 DoF)、DLR Hand。
- 高复杂度、高成本、高维护。
- 控制难、可靠性差。

### 时代 4：2010+ — 软 / 欠驱动
- SDM Hand ([[Ref-10-Odhner-2014-SDMhand]])、Pisa-IIT SoftHand、iHY。
- 简单驱动 + 智能机构 = 鲁棒。
- "Synergies" 思想：少量驱动控制多自由度。

### 时代 5：2020+ — 学习 / Foundation
- RL 控制 Shadow Hand (OpenAI 立方体)。
- 数据驱动 co-design (本论文)。
- 大模型生成动作。

---

## 📐 关键概念：Synergy

**Synergy (协同)** 来自 Santello (2002) 神经科学发现——人类手部 23 自由度 **被神经系统压缩为 ~3 个主成分**。

应用：少量电机也能产生类人抓握模式。
$$q_\text{joint} = U \cdot c,\quad c \in \mathbb{R}^k,\; k \ll n$$

---

## 🔑 对本论文 (Yi'25) 的意义

- **历史定位**：本论文属第 5 时代——**学习驱动的 co-design**。
- **承接思想**：从 SDM Hand (第 4 时代) 借结构，从 Shadow Hand (第 3 时代) 借多物体抓取基准，从 OpenAI 风格 (第 5 时代) 借数据驱动。
- **指引未来**：Piazza 等预言"软 + 学习" 是下一代手——本论文是其代表实现。

---

## 🧭 延伸阅读

- 软体综述：[[Ref-01-Rus-2015-软体机器人综述]]
- 软夹爪综述：[[Ref-02-Shintake-2018-软体夹爪综述]]
- SDM Hand：[[Ref-10-Odhner-2014-SDMhand]]
- 时间线：[[../04-研究脉络/Co-Design 研究时间线]]

---

*← [[_参考文献索引]]*
