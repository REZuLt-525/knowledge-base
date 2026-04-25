# [36] Zhai et al. 2023 · Desktop Fabrication of Monolithic Soft Robots

> **引用**: Y. Zhai, A. De Boer, J. Yan, B. Shih, M. Faber, J. Speros, R. Gupta, M. T. Tolley. *Desktop fabrication of monolithic soft robotic devices with embedded fluidic control circuits.* **Science Robotics**, 8(79):eadg3792, 2023.

`#文献-制造` `#3D打印` `#里程碑`

---

## 🎯 这篇论文想做什么

[[Ref-35-Hubbard-2021-3DPrintedFluidic|Hubbard'21]] 的全 3D 打印方法用昂贵的 PolyJet 工艺。UCSD Tolley 团队 (本论文共作者) 的目标：

> 用 **桌面级 DLP 3D 打印机** 一次性制作软机器人 + 嵌入式流体控制 —— 让任何研究室都能做出复杂软机器人。

> 📌 第一作者 **Yichen Zhai** 出现在本论文 (Yi'25) 致谢名单——同实验室直接传承。

---

## 🛠️ 实现路径

### 1. DLP 工艺
桌面 DLP 打印机 + 自定义软树脂：
- 单层固化时间 < 1 秒。
- 分辨率 ~50 μm。
- 软度可调 (Shore A 30-90)。

### 2. 嵌入式流控

打印时 **同时形成**：
- 主体软结构。
- 流体通道。
- 微阀 (利用几何特性)。
- 双稳态切换器。

### 3. 自主行为示范

文中展示节奏爬行机器人：
- 不接电子。
- 内嵌时序振荡器。
- 脚步交替前进。

---

## 📐 关键技术

### 技术 1：单材料分布刚度
通过 **几何 + 局部光剂量** 改变同一树脂的刚度——单材料即可实现刚柔分布。

**使用场景**：
- **本论文 (Yi'25) 同思想**：FDM 打印 NinjaFlex 时通过 wall loops + infill 改变结构等效模量。
- 让 co-design 落地不需多材料打印机。

### 技术 2：嵌入式流体逻辑
**使用场景**：野外、医疗、教育玩具。

---

## 🔑 对本论文的意义

- **直接前传承**：Tolley 团队 2023 年的 DLP 工艺为本论文 2025 年的 co-design 提供 **可制造性保证**。
- **设计-制造闭环**：算法说"这里要硬度 16.05"，制造层立刻能实现——这正是 Zhai'23 提供的能力。

---

## 🧭 延伸阅读

- 全 PolyJet 打印：[[Ref-35-Hubbard-2021-3DPrintedFluidic]]
- 模塑制造：[[Ref-32-Bilodeau-2015-MonolithicSensors]]
- 软体综述 (同 Tolley)：[[Ref-01-Rus-2015-软体机器人综述]]

---

*← [[_参考文献索引]]*
