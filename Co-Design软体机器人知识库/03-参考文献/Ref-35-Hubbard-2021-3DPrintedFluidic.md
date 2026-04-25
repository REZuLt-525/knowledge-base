# [35] Hubbard et al. 2021 · 全 3D 打印软机器人 + 流体电路

> **引用**: J. D. Hubbard, R. Acevedo, K. M. Edwards, A. T. Alsharhan, Z. Wen, J. Landry, K. Wang, S. Schaffer, R. D. Sochol. *Fully 3D-printed soft robots with integrated fluidic circuitry.* **Science Advances**, 7(29):eabe5257, 2021.

`#文献-制造` `#3D打印` `#流体控制`

---

## 🎯 这篇论文想做什么

软机器人要做完全自主运动，需要 **集成流体逻辑**——但传统流控板需 PDMS 铸造，难和软机器人一体化。马里兰大学 Sochol 团队的目标：

> 用 **PolyJet 多材料 3D 打印** 一次性制作 "**软体 + 流体逻辑 + 微阀**"，实现真正全 3D 打印的可控软机器人。

---

## 🛠️ 实现路径

### 1. PolyJet 工艺
PolyJet (类似 SLA) 喷射光固化树脂，可同时打印 **多种材料** (硬塑料 + 软橡胶)。

### 2. 流体电路设计
模仿电路逻辑门用流体实现：
- **流体 OR / AND / NOT 门**：用通道几何让气流路径表现逻辑。
- 阀门：双稳态压差阀 (类似 [[Ref-33-Rothemund-2018-BistableValve|Rothemund'18]])。

### 3. 软体执行单元
气压驱动的弯曲指 → 单元化打印 → 连接到流控板。

### 4. 应用 demo
- 自主运动的小爬虫机器人。
- 可控顺序操作的多指夹爪。

---

## 📐 关键技术

### 技术 1：多材料一次打印
**使用场景**：
- 减少装配步骤。
- 柔性接缝处直接打印 → 减少应力集中。

### 技术 2：流体逻辑
**使用场景**：
- 在不能用电子的环境 (强磁场、爆炸性气体、潜水)。
- 软体机器人极简化。

---

## 🔑 对本论文 (Yi'25) 的意义

- **制造工艺借鉴**：本论文用 FDM TPU 打印，比 PolyJet 更廉价但功能更简单。
- **未来融合**：把流体逻辑嵌入软夹爪 + co-design 优化 → 完全自主、无电子的智能夹爪。

---

## 🧭 延伸阅读

- 一体化模塑：[[Ref-32-Bilodeau-2015-MonolithicSensors]]
- 桌面制造：[[Ref-36-Zhai-2023-DesktopFabrication]]
- 双稳态阀：[[Ref-33-Rothemund-2018-BistableValve]]

---

*← [[_参考文献索引]]*
